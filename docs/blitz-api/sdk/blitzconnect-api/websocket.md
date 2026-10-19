# WebSocket streaming

BlitzConnect provides two WebSocket endpoints — one for **Interactive** events (order updates, positions, statistics) and one for **Market Data** ticks (live prices).

---

## Interactive WebSocket

Receive real-time order, trade, position, and statistics updates.

```
ws://{host}/api_interactive/ws?access_token={token}
```

> The WebSocket URL is configured in `config.py` as `WS_URL`. On production the path is `wss://{baseurl}/api_interactive/ws`.

### Connection

```javascript
var ws = new WebSocket("wss://{baseurl}/api_interactive/ws?access_token=YOUR_TOKEN");
```

### Actions

```json
{"action": "AllSubscribe"}
```

| Action | Event codes subscribed |
|--------|----------------------|
| `OrderSubscribe` | 70000 |
| `StatisticSubscribe` | 50000 |
| `StrategyStatisticSubscribe` | 80000 |
| `InstrumentStatisticSubscribe` | 90000 |
| `AllSubscribe` | Subscribes 50000, 70000, 80000, 90000 |

To unsubscribe, call `unsubscribe_action("ActionName")` — this sends the corresponding `*Unsubscribe` command to the server and removes the action from the auto-reconnect list.

After sending `AllSubscribe`, the server replays the current session state: statistic snapshots followed by order events.

### Message format

Every subsequent message carries a `MessageCode`. `DecodedData` is delivered as an already-parsed JSON object — no double parsing needed.

| Code | Meaning | Notes |
|------|---------|-------|
| `50000` | Statistic | Bare ack right after subscribe; full statistics snapshot during activity |
| `70000` | Order update | One message per order state change (`PendingNew`, `New`, `Filled`, `Cancelled`, ...) |
| `80000` | Strategy statistics | Position/P&L snapshot per strategy instance |
| `90000` | Instrument statistics (RMS) | Per-instrument position/RMS data |

Complete examples below are real frames captured on UAT during a full order lifecycle (place → acknowledge → fill).

#### Connection acknowledgement

```json
{
  "type": "connection",
  "status": "connected",
  "sessionId": "5cef71999ef1495ab39a99ea28ea7632"
}
```

#### Statistic — bare ack (50000)

```json
{
  "MessageCode": 50000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed"
}
```

#### Statistic — snapshot (50000)

Same layout as `80000`, pushed under code `50000` during active trading:

```json
{
  "MessageCode": 50000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "DecodedData": {
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyName": "Manual Trading",
    "StrategyInstanceId": "83780c13-2129-4910-b22e-02b3d80d23bb",
    "StrategyInstanceName": "NSECM|IDEA",
    "ExchangeClientId": "Algo123",
    "InstrumentId": 110010000014366,
    "ExchangeSegment": "NSECM",
    "InstrumentName": "IDEA",
    "NonAckBuyQuantity": 0,
    "NonAckSellQuantity": 0,
    "OpenBuyQuantity": 4,
    "OpenSellQuantity": 0,
    "ShortPosition": 0,
    "LongPosition": 1,
    "NetPosition": 1,
    "NetPositionInLot": 1,
    "OrderCount": 12,
    "TradeCount": 1,
    "PreviousNetPosition": 0,
    "TradeBuyValue": 11.51,
    "TradeSellValue": 0,
    "TradeNetValue": -11.51,
    "AvgBuy": 11.51,
    "AvgSell": 0,
    "AvgNet": -11.51,
    "Exposure": 103.59,
    "UnRealized": 0,
    "Realized": 0,
    "LastTradedPrice": 11.51,
    "OpenPositionTradeValue": 11.51,
    "TurnoverValue": 11.51,
    "TurnoverValueOptions": 0
  }
}
```

#### Order placed — PendingNew (70000)

```json
{
  "MessageCode": 70000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "DecodedData": {
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "CorrelationOrderId": "test_1783415809433",
    "Account": "",
    "CTCLId": "1111111112",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "UserId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "UserLoginId": "",
    "ExchangeOrderId": "",
    "ExecutionId": "",
    "InstrumentName": "IDEA",
    "ExchangeSegment": "NSECM",
    "OrderStatus": "PendingNew",
    "RejectTypeReason": "",
    "LastTradedQuantity": 0,
    "LeavesQuantity": 1,
    "OrderPrice": 13.84,
    "OrderTriggerPrice": 0,
    "OrderStopPrice": 0,
    "OrderQuantity": 1,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "AverageTradedPrice": 0,
    "BlitzOrderId": 27120017200000022,
    "IsOrderCompleted": false,
    "ExchangeInstrumentId": 110010000014366,
    "InstrumentType": "Equity",
    "ExecutionType": "None",
    "LastTradedPrice": 0,
    "CumulativeQuantity": 0,
    "TimeInForce": "GFD",
    "OrderExpiryDate": 1783415811292,
    "OrderGeneratedDateTime": 1783415811292,
    "LastRequestDateTime": 1783415811292,
    "ExchangeTransactTime": 1783415811292,
    "OrderModificationCount": 0,
    "OrderTradeCount": 0,
    "AverageTradedValue": 0,
    "IsFictiveOrder": false,
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyInstanceId": "83780c13-2129-4910-b22e-02b3d80d23bb",
    "StrategyName": "Manual Trading",
    "StrategyInstanceName": "NSECM|IDEA",
    "IVObjectName": "IV",
    "InstrumentId": 110010000014366,
    "OrderSide": "Buy",
    "OrderTag": "NormalOrder",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "UserText": "",
    "RMSTag": "General",
    "SequenceNumber": 42
  }
}
```

#### Order acknowledged by exchange — New (70000)

```json
{
  "MessageCode": 70000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "DecodedData": {
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "CorrelationOrderId": "test_1783415809433",
    "Account": "",
    "CTCLId": "1111111112",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "UserId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "UserLoginId": "",
    "ExchangeOrderId": "3318217531817222",
    "ExecutionId": "3318217531898406",
    "InstrumentName": "IDEA",
    "ExchangeSegment": "NSECM",
    "OrderStatus": "New",
    "RejectTypeReason": "",
    "LastTradedQuantity": 0,
    "LeavesQuantity": 1,
    "OrderPrice": 13.84,
    "OrderTriggerPrice": 0,
    "OrderStopPrice": 0,
    "OrderQuantity": 1,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "AverageTradedPrice": 0,
    "BlitzOrderId": 27120017200000022,
    "IsOrderCompleted": false,
    "ExchangeInstrumentId": 110010000014366,
    "InstrumentType": "Equity",
    "ExecutionType": "New",
    "LastTradedPrice": 0,
    "CumulativeQuantity": 0,
    "TimeInForce": "GFD",
    "OrderExpiryDate": 1783415811292,
    "OrderGeneratedDateTime": 1783415811292,
    "LastRequestDateTime": 1783415811292,
    "ExchangeTransactTime": 1783435611000,
    "OrderModificationCount": 0,
    "OrderTradeCount": 0,
    "AverageTradedValue": 0,
    "IsFictiveOrder": false,
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyInstanceId": "83780c13-2129-4910-b22e-02b3d80d23bb",
    "StrategyName": "Manual Trading",
    "StrategyInstanceName": "NSECM|IDEA",
    "IVObjectName": "IV",
    "InstrumentId": 110010000014366,
    "OrderSide": "Buy",
    "OrderTag": "NormalOrder",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "UserText": "",
    "RMSTag": "General",
    "SequenceNumber": 43
  }
}
```

#### Order filled (70000)

```json
{
  "MessageCode": 70000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "DecodedData": {
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "CorrelationOrderId": "test_1783415809433",
    "Account": "",
    "CTCLId": "1111111112",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "UserId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "UserLoginId": "",
    "ExchangeOrderId": "3318243893211821",
    "ExecutionId": "3318243893280411",
    "InstrumentName": "IDEA",
    "ExchangeSegment": "NSECM",
    "OrderStatus": "Filled",
    "RejectTypeReason": "",
    "LastTradedQuantity": 1,
    "LeavesQuantity": 0,
    "OrderPrice": 13.84,
    "OrderTriggerPrice": 0,
    "OrderStopPrice": 0,
    "OrderQuantity": 1,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "AverageTradedPrice": 11.51,
    "BlitzOrderId": 27120017200000022,
    "IsOrderCompleted": true,
    "ExchangeInstrumentId": 110010000014366,
    "InstrumentType": "Equity",
    "ExecutionType": "Fill",
    "LastTradedPrice": 11.51,
    "CumulativeQuantity": 1,
    "TimeInForce": "GFD",
    "OrderExpiryDate": 1783415811293,
    "OrderGeneratedDateTime": 1783415811293,
    "LastRequestDateTime": 1783415811293,
    "ExchangeTransactTime": 1783415811000,
    "OrderModificationCount": 0,
    "OrderTradeCount": 0,
    "AverageTradedValue": 0,
    "IsFictiveOrder": false,
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyInstanceId": "83780c13-2129-4910-b22e-02b3d80d23bb",
    "StrategyName": "Manual Trading",
    "StrategyInstanceName": "NSECM|IDEA",
    "IVObjectName": "IV",
    "InstrumentId": 110010000014366,
    "OrderSide": "Buy",
    "OrderTag": "NormalOrder",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "UserText": "",
    "RMSTag": "General",
    "SequenceNumber": 51
  }
}
```

#### Strategy statistics (80000)

```json
{
  "MessageCode": 80000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "DecodedData": {
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyName": "Manual Trading",
    "StrategyInstanceId": "83780c13-2129-4910-b22e-02b3d80d23bb",
    "StrategyInstanceName": "NSECM|IDEA",
    "ExchangeClientId": "Algo123",
    "InstrumentId": 110010000014366,
    "IVName": "IV",
    "ExchangeSegment": "NSECM",
    "InstrumentName": "IDEA",
    "NonAckBuyQuantity": 0,
    "NonAckSellQuantity": 0,
    "OpenBuyQuantity": 4,
    "OpenSellQuantity": 0,
    "ShortPosition": 0,
    "LongPosition": 1,
    "NetPosition": 1,
    "NetPositionInLot": 1,
    "OrderCount": 12,
    "TradeCount": 1,
    "PreviousNetPosition": 0,
    "TradeBuyValue": 11.51,
    "TradeSellValue": 0,
    "TradeNetValue": -11.51,
    "AvgBuy": 11.51,
    "AvgSell": 0,
    "AvgNet": -11.51,
    "OPBuyPrice": 0,
    "OPSellPrice": 0,
    "Exposure": 103.59,
    "UnRealized": 0,
    "Realized": 0,
    "LastTradedPrice": 11.51,
    "OpenPositionTradeValue": 11.51,
    "TurnoverValue": 11.51,
    "TurnoverValueOptions": 0
  }
}
```

#### Instrument statistics / RMS (90000)

```json
{
  "MessageCode": 90000,
  "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "DecodedData": {
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "ExchangeClientId": "Algo123",
    "RMSEntityCode": "",
    "InstrumentId": 110010000014366,
    "ExchangeSegment": "NSECM",
    "InstrumentName": "IDEA",
    "NonAckBuyQuantity": 0,
    "NonAckSellQuantity": 0,
    "OpenBuyQuantity": 1,
    "OpenSellQuantity": 0,
    "ShortPosition": 0,
    "LongPosition": 1,
    "NetPosition": 1,
    "NetPositionInLot": 1,
    "OrderCount": 2,
    "TradeCount": 1,
    "PreviousNetPosition": 0,
    "TradeBuyValue": 11.51,
    "TradeSellValue": 0,
    "TradeNetValue": -11.51,
    "AvgBuy": 11.51,
    "AvgSell": 0,
    "AvgNet": -11.51,
    "Exposure": 34.53,
    "UnRealized": 0,
    "Realized": 0,
    "LastTradedPrice": 11.51,
    "OpenPositionTradeValue": 11.51,
    "TurnoverValue": 11.51,
    "TurnoverValueOptions": 0
  }
}
```

---

## SDK

=== "Python"

    ```python
    # Interactive WS
    from blitzsdk import InteractiveWebSocketClient

    def on_message(msg):
        # msg is a parsed JSON dict: {"MessageCode": ..., "DecodedData": ...}
        print("Code:", msg["MessageCode"], "Data:", msg.get("DecodedData"))
        # Connection ack: {"type":"connection","status":"connected","sessionId":"..."}
        if msg.get("type") == "connection":
            print("WS fully connected, session:", msg.get("sessionId"))

    def on_connect():
        print("WebSocket TCP connected!")

    def on_close(code, reason):
        print(f"WebSocket disconnected: code={code} reason={reason}")

    def on_error(err):
        print("WebSocket error:", err)

    iws = InteractiveWebSocketClient(app_key="YOUR_KEY", user_id="YOUR_USER")
    iws.set_on_message(on_message)
    iws.set_on_connect(on_connect)
    iws.set_on_close(on_close)
    iws.set_on_error(on_error)
    iws.start()
    iws.subscribe_action("AllSubscribe")

    # To unsubscribe a specific action:
    # iws.unsubscribe_action("AllSubscribe")

    # Keep running (Ctrl+C to exit)
    try:
        import time
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        iws.stop()
    ```

=== "C#"

    ```csharp
    // Interactive WS
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var iws = new BlitzWebSocketClient(config);
    iws.OnMessage = (msg) => Console.WriteLine($"Code:{msg.MessageCode} Data:{msg.DecodedData}");
    iws.OnConnect = () => Console.WriteLine("Connected!");
    iws.OnClose = () => Console.WriteLine("Disconnected");
    iws.OnError = (ex) => Console.WriteLine($"Error: {ex.Message}");
    iws.Start();
    await iws.SubscribeActionAsync("AllSubscribe");

    // Keep running
    await Task.Delay(Timeout.Infinite);
    ```

### Reconnection

Both SDKs auto-reconnect on disconnect and automatically re-send subscribed actions after each reconnect. The Python client uses a fixed 5-second delay; the C# client backs off exponentially from 1 s up to 30 s. The Python client also sends a `"ping"` text frame every 30 seconds as a keep-alive; incoming `ping` frames from the server are filtered out by both SDKs.
