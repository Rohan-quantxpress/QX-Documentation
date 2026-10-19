# Market Data WebSocket

Stream real-time market data ticks as binary Protocol Buffers, automatically decoded by the SDK.

=== "Python"

    ```python
    from blitzsdk.marketdata import MarketDataWebSocketClient

    def on_message(decoded):
        # decoded is a protobuf MarketDataMessageBase object
        # Use decoded.WhichOneof("subtype") to check which field is set
        print(decoded)

    client = MarketDataWebSocketClient(app_key="YOUR_KEY", user_id="YOUR_USER")
    client.set_on_message(on_message)
    client.set_on_connect(lambda: print("Connected"))
    client.set_on_close(lambda code, reason: print("Disconnected:", code))
    client.set_on_error(lambda e: print("Error:", e))
    client.start()
    client.subscribe([110010002000001, 110010000002885])

    # To unsubscribe:
    # client.unsubscribe([110010000002885])
    ```

=== "C#"

    ```csharp
    using BlitzConnect.MarketData;

    var mdWs = new MarketDataWebSocket("wss://{baseurl}/md-streaming/ws?key=YOUR_TOKEN");
    mdWs.OnMessage += (msg) => Console.WriteLine(msg);
    mdWs.OnMarketDepth += (d) => Console.WriteLine($"Depth: {d.InstrumentName} LTP={d.LTP}");
    mdWs.OnTick += (t) => Console.WriteLine($"Tick: {t.InstrumentName} LTP={t.LTP}");
    mdWs.OnConnected += () => Console.WriteLine("Connected");
    mdWs.OnDisconnected += (code, reason) => Console.WriteLine($"Disconnected: {code}");
    await mdWs.ConnectAsync();
    await mdWs.SubscribeAsync(new List<long> { 110010002000001, 110010000002885 });

    // To unsubscribe:
    // await mdWs.UnsubscribeAsync(new List<long> { 110010000002885 });
    ```

## Typed callbacks

Besides the generic `on_message`, the SDKs dispatch each decoded frame to a per-message-type callback:

=== "Python"

    ```python
    def on_tick(t):
        print("TICK", t.InstrumentName, t.LTP)

    def on_depth(d):
        print("DEPTH", d.InstrumentName, len(d.BestBidLevel), "bid levels")

    def on_touchline(t):
        print("TOUCHLINE", t.InstrumentName, t.LTP)

    def on_index(i):
        print("INDEX", i.InstrumentName, i.LTP)

    def on_incremental(inc):
        print("INCREMENTAL", inc)

    client.set_on_tick(on_tick)
    client.set_on_depth(on_depth)
    client.set_on_touchline(on_touchline)
    client.set_on_index(on_index)          # receives both IndexDataMessage and IndexDataListMessage
    client.set_on_incremental(on_incremental)
    ```

=== "C#"

    ```csharp
    mdWs.OnMarketDepth += (d) => ...;      // MarketDepthMessage
    mdWs.OnTick += (t) => ...;             // TickDataMessage
    mdWs.OnTouchline += (t) => ...;        // TouchLineDataMessage
    mdWs.OnIndex += (i) => ...;            // IndexDataMessage
    mdWs.OnIndexList += (il) => ...;       // IndexDataListMessage
    mdWs.OnIncremental += (inc) => ...;    // IncrementalUpdateMessage

    // Convenience event: fires for any frame carrying an LTP > 0
    // (TickDataMessage, TickData, TouchLineDataMessage, MarketDepthMessage)
    mdWs.OnLtp += (instrumentId, ltp) => ...;

    mdWs.OnError += (err) => ...;          // string error (parse failures etc.)
    ```

> Python notes: the raw decoded protobuf envelope of the last received frame is always available as `ws.last_envelope`; `TickData` (1506) frames are folded into `set_on_tick` alongside `TickDataMessage`. The C# client exposes the generic `OnMessage` event for every decoded envelope plus the typed events above.

---

## Connection URL

```
wss://{baseurl}/md-streaming/ws?key=YOUR_TOKEN
```

## Subscribe / unsubscribe

After connecting, send a JSON subscribe message:

```json
{
    "action": "subscribe",
    "instrumentIds": [110010002000001, 110010000002885]
}
```

To unsubscribe:

```json
{
    "action": "unsubscribe",
    "instrumentIds": [110010000002885]
}
```

> `instrumentIds` must be numeric instrument IDs (not symbol strings).

## Heartbeat

The server sends `"ping"` as a text message periodically. Both SDKs filter ping messages out automatically, so callbacks never see them. (The Python interactive WebSocket client additionally sends its own `"ping"` keep-alive every 30 seconds.)

## Message types

Binary frames are base64-wrapped Protocol Buffers (`MarketDataMessageBase`); exactly one subtype is set per frame.

| Subtype | MessageCode | Description |
|---------|-------------|-------------|
| `TickDataMessage` | 1501 | Trade tick |
| `MarketDepthMessage` | 1502 | Top-of-book snapshot with depth levels |
| `IndexDataMessage` | 1503 | Single index snapshot |
| `IncrementalUpdateMessage` | 1504 | Incremental market data entries |
| `IndexDataListMessage` | 1505 | Batch of index snapshots (`IndexDataList` array) |
| `TickData` | 1506 | Compact trade tick |
| `TouchLineDataMessage` | 1507 | Touchline (OHLC + best prices) |

!!! note "**Depth is top-5 only**"
    The feed disseminates **5 bid + 5 ask levels maximum** per `MarketDepthMessage` — this is an exchange-side limit of the public tick feed, not an SDK or gateway restriction. `BestBidLevel` / `BestAskLevel` always contain at most 5 entries.

## Response

Below are the JSON representations of each message type, as produced by `google.protobuf.json_format.MessageToJson()` on the protobuf envelope received by `on_message`:

### MarketDepthMessage (MessageCode = 1502)

```json
{
  "MessageCode": 1502,
  "ID": "110010000014366",
  "MarketDepthMessage": {
    "InstrumentID": "110010000014366",
    "ExchangeSegment": "NSECM",
    "ExchangeInstrumentID": 14366,
    "InstrumentName": "IDEA",
    "TimeStamp": "1770728960",
    "LTP": 11.51,
    "LTQ": 6,
    "LTT": "1770728958",
    "ATP": 11.55,
    "VTT": "268805264",
    "TBQ": "28554459",
    "TSQ": "80855892",
    "Open": 11.58,
    "High": 11.67,
    "Low": 11.48,
    "Close": 11.58,
    "BestBidLevel": [
      { "Qty": 833392, "Price": 11.5, "Orders": 32 },
      { "Qty": 1544228, "Price": 11.49, "Orders": 77 },
      { "Qty": 2661470, "Price": 11.48, "Orders": 376 },
      { "Qty": 979035, "Price": 11.47, "Orders": 126 },
      { "Qty": 835550, "Price": 11.46, "Orders": 126 }
    ],
    "BestAskLevel": [
      { "Qty": 1022084, "Price": 11.51, "Orders": 66 },
      { "Qty": 812256, "Price": 11.52, "Orders": 69 },
      { "Qty": 1156649, "Price": 11.53, "Orders": 70 },
      { "Qty": 1867230, "Price": 11.54, "Orders": 120 },
      { "Qty": 1503815, "Price": 11.55, "Orders": 219 }
    ]
  }
}
```

> In the live JSON, `TimeStamp`, `LTT`, `VTT`, `TBQ` and `TSQ` (int64 values) are serialized as strings. `ExchangeSegment` is returned as a string (`"NSECM"`) rather than a numeric code. Depth levels use the `Qty` / `Price` / `Orders` fields.

| Field | Type | Description |
|-------|------|-------------|
| `MessageCode` | int | Message type code (`1502`) |
| `ID` | string | Instrument ID |
| `MarketDepthMessage` | object | Market depth payload |
| `InstrumentID` | string | Instrument ID |
| `ExchangeSegment` | string | Exchange segment (`"NSECM"`) |
| `ExchangeInstrumentID` | uint64 | Exchange instrument ID |
| `InstrumentName` | string | Instrument name |
| `TimeStamp` | string | Tick timestamp (epoch seconds) |
| `LTP` | double | Last traded price |
| `LTQ` | int | Last traded quantity |
| `LTT` | string | Last traded time (epoch seconds) |
| `ATP` | double | Average traded price |
| `VTT` | string | Volume today |
| `TBQ` | string | Total buy quantity |
| `TSQ` | string | Total sell quantity |
| `Open` | double | Opening price |
| `High` | double | High price |
| `Low` | double | Low price |
| `Close` | double | Previous close |
| `BestBidLevel` | repeated | Best bid depth levels |
| `BestAskLevel` | repeated | Best ask depth levels |

#### Depth level fields (BestBidLevel / BestAskLevel)

| Field | Type | Description |
|-------|------|-------------|
| `Qty` | int | Quantity at level |
| `Price` | double | Bid/Ask price |
| `Orders` | int | Number of orders |

### IndexDataMessage (MessageCode = 1503) / IndexDataListMessage (MessageCode = 1505)

Single index snapshots arrive as `IndexDataMessage`; batches arrive as `IndexDataListMessage`, where the payloads are wrapped in an `IndexDataList` **array**:

```json
{
  "MessageCode": 1505,
  "IndexDataListMessage": {
    "IndexDataList": [
      {
        "InstrumentID": "110010002000001",
        "ExchangeSegment": "NSECM",
        "ExchangeInstrumentID": 2000001,
        "IndexName": "NIFTY 50",
        "TimeStamp": "1770719042",
        "Last": 25963.3,
        "Open": 25922.65,
        "High": 25971.85,
        "Low": 25870.45,
        "Close": 24511.2,
        "YearlyHigh": 26373.2,
        "YearlyLow": 21743.65
      }
    ]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `InstrumentID` | string | Index instrument ID |
| `ExchangeSegment` | string | Exchange segment (`"NSECM"`) |
| `ExchangeInstrumentID` | uint64 | Exchange instrument ID |
| `IndexName` | string | Index name |
| `TimeStamp` | string | Tick timestamp (epoch seconds) |
| `Last` | double | Last index value |
| `Open` | double | Open value |
| `High` | double | High value |
| `Low` | double | Low value |
| `Close` | double | Previous close value |
| `YearlyHigh` | double | 52-week high |
| `YearlyLow` | double | 52-week low |

> When consuming via the Python typed callback, `set_on_index` receives both subtypes: use `payload.IndexDataList` for batch frames and `payload.IndexData` for single frames.

### TickDataMessage (MessageCode = 1501) / TickData (MessageCode = 1506)

Trade-tick updates for actively traded instruments. `1501` is the full tick message; `1506` is a compact variant carrying an extra `Flag`.

```json
{
  "MessageCode": 1501,
  "ID": "110010000002885",
  "TickDataMessage": {
    "InstrumentID": "110010000002885",
    "ExchangeSegment": "NSECM",
    "ExchangeInstrumentID": 2885,
    "InstrumentName": "RELIANCE",
    "TimeStamp": "1455185634",
    "LTP": 1463.9,
    "LTQ": 3,
    "LTT": "1455185633"
  }
}
```

```json
{
  "MessageCode": 1506,
  "ID": "110010000002885",
  "TickData": {
    "InstrumentID": "110010000002885",
    "ExchangeSegment": "NSECM",
    "ExchangeInstrumentID": 2885,
    "InstrumentName": "RELIANCE",
    "LTP": 1463.9,
    "LTQ": 3,
    "LTT": "1455185633",
    "Flag": 0
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `InstrumentID` | string | Instrument ID |
| `ExchangeSegment` | string | Exchange segment (`"NSECM"`) |
| `ExchangeInstrumentID` | uint64 | Exchange instrument ID |
| `InstrumentName` | string | Instrument name |
| `TimeStamp` | string | Tick timestamp (epoch seconds); absent in the `1506` variant |
| `LTP` | double | Last traded price |
| `LTQ` | int | Last traded quantity |
| `LTT` | string | Last traded time (epoch seconds) |
| `Flag` | int | Tick flag (`1506` variant only) |

### TouchLineDataMessage (MessageCode = 1507)

OHLC/statistics snapshot without depth levels (touchline data):

```json
{
  "MessageCode": 1507,
  "ID": "110010000002885",
  "TouchLineDataMessage": {
    "InstrumentID": "110010000002885",
    "ExchangeSegment": "NSECM",
    "ExchangeInstrumentID": 2885,
    "InstrumentName": "RELIANCE",
    "TimeStamp": "1455185634",
    "LTP": 1463.9,
    "LTQ": 3,
    "LTT": "1455185633",
    "ATP": 1467.26,
    "VTT": "916059",
    "TBQ": "326915",
    "TSQ": "707507",
    "Open": 1471.0,
    "High": 1471.0,
    "Low": 1462.7,
    "Close": 1461.6
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `InstrumentID` | string | Instrument ID |
| `ExchangeSegment` | string | Exchange segment (`"NSECM"`) |
| `ExchangeInstrumentID` | uint64 | Exchange instrument ID |
| `InstrumentName` | string | Instrument name |
| `TimeStamp` | string | Tick timestamp (epoch seconds) |
| `LTP` | double | Last traded price |
| `LTQ` | int | Last traded quantity |
| `LTT` | string | Last traded time (epoch seconds) |
| `ATP` | double | Average traded price |
| `VTT` | string | Volume today |
| `TBQ` | string | Total buy quantity |
| `TSQ` | string | Total sell quantity |
| `Open` / `High` / `Low` / `Close` | double | OHLC prices |

### IncrementalUpdateMessage (MessageCode = 1504)

Depth deltas during fast markets. Each frame carries a list of `MDEntries` entries; `MDUpdateAction` says what happened to the level and `MDEntryPosition` where it sits in the book:

```json
{
  "MessageCode": 1504,
  "IncrementalUpdateMessage": {
    "MDEntriesList": [
      {
        "MDUpdateAction": 0,
        "MDEntryType": 0,
        "InstrumentID": "110010000002885",
        "ExchangeSegment": "NSECM",
        "ExchangeInstrumentID": 2885,
        "InstrumentName": "RELIANCE",
        "MDEntryPrice": 1463.8,
        "MDEntrySize": 420,
        "MDEntryTime": 1455185634,
        "MDEntryPosition": 1,
        "OI": 0
      },
      {
        "MDUpdateAction": 0,
        "MDEntryType": 1,
        "InstrumentID": "110010000002885",
        "ExchangeSegment": "NSECM",
        "ExchangeInstrumentID": 2885,
        "InstrumentName": "RELIANCE",
        "MDEntryPrice": 1464.0,
        "MDEntrySize": 380,
        "MDEntryTime": 1455185634,
        "MDEntryPosition": 1,
        "OI": 0
      }
    ]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `MDUpdateAction` | int | Update action for the level (`0` = new/update) |
| `MDEntryType` | int | Entry type (`0` = bid, `1` = ask; trade/OI variants per FIX conventions) |
| `InstrumentID` | string | Instrument ID |
| `ExchangeSegment` | string | Exchange segment (`"NSECM"`) |
| `ExchangeInstrumentID` | uint64 | Exchange instrument ID |
| `InstrumentName` | string | Instrument name |
| `MDEntryPrice` | double | Level price |
| `MDEntrySize` | int | Quantity at the level |
| `MDEntryTime` | int | Entry time (epoch seconds) |
| `MDEntryPosition` | int | Position in the book (1-based) |
| `OI` | int64 | Open interest (derivatives) |

!!! note "Which frames you actually receive"
    The `1502` and `1505` examples above are verbatim UAT captures. Tick (`1501`/`1506`), touchline (`1507`) and incremental (`1504`) frames flow only for high-activity instruments during market bursts; their structures above are complete per the server protobuf schema ([marketdata.proto](../../api/market-data/marketdata.proto)), with representative values. Proto3 omits fields equal to zero (e.g. `NetChangeIndicator`, `OI`) from the JSON.
