# Market Data Protobuf Encoding

Real-time market data streamed over the **Market Data WebSocket** (`/md-streaming/ws`) is **not JSON**. Each tick arrives as a compact binary payload encoded with [Protocol Buffers (protobuf)](https://protobuf.dev/) — Google's language-neutral, platform-neutral serialization format.

The schema is defined in **`marketdata.proto`**, which specifies every message type, field, and enum used on the wire.

---

## Proto definition

The canonical proto file lives in the Blitz Python SDK repository:

- **Repository:** [Quantxpress-Technologies/blitz-api-py](https://github.com/Quantxpress-Technologies/blitz-api-py)
- **Path:** [`blitzsdk/proto/marketdata.proto`](https://github.com/Quantxpress-Technologies/blitz-api-py/blob/master/blitzsdk/proto/marketdata.proto)
- **Generated Python module:** [`blitzsdk/proto/marketdata_pb2.py`](https://github.com/Quantxpress-Technologies/blitz-api-py/blob/master/blitzsdk/proto/marketdata_pb2.py)

A copy of the same `.proto` file is included alongside this documentation for reference: [`marketdata.proto`](marketdata.proto).

---

## Why protobuf?

| Aspect | REST APIs (LTP, Quote, etc.) | WebSocket streaming |
|--------|------------------------------|---------------------|
| Format | JSON | Protobuf (binary) |
| Use case | On-demand snapshots | High-frequency live ticks |
| Size | Larger text payloads | Compact binary frames |
| Parsing | Standard JSON parser | Protobuf deserializer |

Protobuf keeps bandwidth and latency low when thousands of ticks arrive per second across many instruments.

---

## Message envelope

Every WebSocket frame decodes into a single root message: **`MarketDataMessageBase`**.

```protobuf
message MarketDataMessageBase {
   int32 MessageCode = 1;
   uint64 ID = 2;
   int32 MessageSize = 3;
   oneof subtype {
      TickDataMessage TickDataMessage = 500;
      TouchLineDataMessage TouchLineDataMessage = 501;
      MarketDepthMessage MarketDepthMessage = 502;
      IndexDataMessage IndexDataMessage = 503;
      IndexDataListMessage IndexDataListMessage = 504;
      IncrementalUpdateMessage IncrementalUpdateMessage = 505;
      TickData TickData = 506;
   }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `MessageCode` | int32 | Wire message code (`1501`–`1507`, see table below) |
| `ID` | uint64 | Message or instrument identifier |
| `MessageSize` | int32 | Payload size hint |
| `subtype` | oneof | Exactly one nested message — see table below |

### Subtype → MessageCode mapping

The numbers **500–506** in the `.proto` snippet above are protobuf *oneof field tags* (wire-encoding detail only). The application-level codes carried in the JSON envelope are the **1500 series**:

| Oneof field tag | Proto message | MessageCode (JSON) | Description |
|-----------------|---------------|--------------------|-------------|
| 500 | `TickDataMessage` | **1501** | Tick with timestamp wrapper |
| 501 | `TouchLineDataMessage` | **1507** | Touchline quote (LTP, OHLC, OI, top-of-book) |
| 502 | `MarketDepthMessage` | **1502** | Full market depth (bid/ask levels) |
| 503 | `IndexDataMessage` | **1503** | Single index update |
| 504 | `IndexDataListMessage` | **1505** | Batch index updates |
| 505 | `IncrementalUpdateMessage` | **1504** | Incremental book/field updates |
| 506 | `TickData` | **1506** | Compact tick (no outer timestamp field) |

> Verified on UAT: live frames arrive with `"MessageCode": 1502` for depth and `"MessageCode": 1505` for index batches. After deserialization, prefer `WhichOneof("subtype")` (or the SDK typed callbacks) over switching on `MessageCode`.

---

## ExchangeSegment enum

Instrument exchange segments are encoded as numeric enum values:

| Value | Name | Segment |
|-------|------|---------|
| 0 | `NONE` | Unspecified |
| 1 | `NSECM` | NSE Cash Market |
| 2 | `NSEFO` | NSE F&O |
| 3 | `NSECD` | NSE Currency Derivatives |
| 4 | `NSECO` | NSE Commodity |
| 11 | `BSECM` | BSE Cash Market |
| 12 | `BSEFO` | BSE F&O |
| 13 | `BSECD` | BSE Currency Derivatives |
| 14 | `BSECO` | BSE Commodity |
| 21 | `NCDEX` | NCDEX |
| 31 | `MCXFO` | MCX F&O |

---

## Generate Python deserialization code

To deserialize market data in Python, compile `marketdata.proto` into a `*_pb2.py` module using the Protocol Buffers compiler.

### Prerequisites

```bash
pip install protobuf grpcio-tools
```

### Compile command

From the root of the [`blitz-api-py`](https://github.com/Quantxpress-Technologies/blitz-api-py) repository:

```bash
python -m grpc_tools.protoc \
  -I blitzsdk/proto \
  --python_out=blitzsdk/proto \
  blitzsdk/proto/marketdata.proto
```

This generates:

```
blitzsdk/proto/marketdata_pb2.py
```

> **Note:** Do not edit `marketdata_pb2.py` by hand. Re-run the command above whenever `marketdata.proto` changes.

### Windows (PowerShell)

```powershell
python -m grpc_tools.protoc `
  -I blitzsdk/proto `
  --python_out=blitzsdk/proto `
  blitzsdk/proto/marketdata.proto
```

---

## Deserialization process

When a binary WebSocket frame arrives, follow these steps:

```
WebSocket frame (base64 text or raw bytes)
        │
        ▼
  base64.b64decode()          ← if the frame is a base64 string
        │
        ▼
  marketdata_pb2.MarketDataMessageBase()
        │
        ▼
  .ParseFromString(binary)    ← populate the protobuf object
        │
        ▼
  Read MessageCode / subtype  ← TickData, TouchLineDataMessage, etc.
```

### Python example

```python
import base64
from blitzsdk.proto import marketdata_pb2
from google.protobuf.json_format import MessageToJson

def on_websocket_message(raw_message):
    # Step 1: decode base64 (server sends base64-encoded protobuf)
    binary = base64.b64decode(raw_message)

    # Step 2: create an empty protobuf object
    md = marketdata_pb2.MarketDataMessageBase()

    # Step 3: parse the binary payload into the object
    md.ParseFromString(binary)

    # Step 4: inspect the message type and fields
    # MessageCode 1506 = TickData (compact tick)
    if md.MessageCode == 1506 and md.HasField("TickData"):
        tick = md.TickData
        print(f"{tick.InstrumentName} LTP={tick.LTP}")

    # Optional: convert the full message to JSON for debugging
    print(MessageToJson(md))
```

This is the same flow used internally by `MarketDataWebSocketClient` in the Blitz SDK:

```python
decoded = base64.b64decode(message)
md = marketdata_pb2.MarketDataMessageBase()
md.ParseFromString(decoded)
```

---

## Inspecting which subtype was received

Protobuf `oneof` fields allow only one branch to be set. Use any of these approaches:

```python
# By MessageCode (1500-series, NOT the 500-series field tags)
if md.MessageCode == 1507:
    quote = md.TouchLineDataMessage

# By oneof name (recommended)
subtype = md.WhichOneof("subtype")  # e.g. "TickData", "TouchLineDataMessage"
payload = getattr(md, subtype) if subtype else None

# By HasField
if md.HasField("MarketDepthMessage"):
    depth = md.MarketDepthMessage
```

---

## Nested message reference

### TickData / TickDataMessage

Core price tick fields: `InstrumentID`, `ExchangeSegment`, `InstrumentName`, `LTP`, `LTQ`, `LTT`.

### TouchLineDataMessage / MarketDepthMessage

Full quote: `LTP`, `ATP`, `VTT`, `TBQ`, `TSQ`, `Open`, `High`, `Low`, `Close`, `OI`, plus:

```protobuf
message PriceDepthLevel {
   int32 Qty = 1;
   double Price = 2;
   int32 Orders = 3;
}
```

Repeated as `BestBidLevel` and `BestAskLevel`.

### IndexData / IndexDataMessage

Index-specific fields: `IndexName`, `Last`, `YearlyHigh`, `YearlyLow`, `NetChangeIndicator`.

### IncrementalUpdateMessage

Contains a repeated list of `MDEntries` for partial book or field updates.

---

## Related documentation

- [Market Quote API](marketquote.md) — REST endpoint returning JSON quotes
- [Last Traded Price (LTP)](ltp.md) — REST endpoint for snapshot LTP
- [Market Data Streaming (SDK)](../../sdk/blitzconnect-api/market-data-streaming.md) — WebSocket connection, subscribe/unsubscribe, and decoded message examples

---

## Quick checklist

1. Obtain `marketdata.proto` from [`blitz-api-py/blitzsdk/proto`](https://github.com/Quantxpress-Technologies/blitz-api-py/tree/master/blitzsdk/proto).
2. Run `python -m grpc_tools.protoc ...` to generate `marketdata_pb2.py`.
3. Connect to `wss://{baseurl}/md-streaming/ws?key={token}` and subscribe with instrument IDs.
4. On each binary frame: **base64-decode → `ParseFromString` → read `MessageCode` / subtype**.
5. Prefer the [Blitz SDK](https://github.com/Quantxpress-Technologies/blitz-api-py) if you do not need a custom client — decoding is handled automatically.
