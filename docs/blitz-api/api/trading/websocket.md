# WebSocket Streaming (Interactive)

Receive real-time order, trade, position, and statistics updates.

!!! note "Canonical reference"
    This page documents the raw WebSocket protocol. **Complete, verified payload examples for every message code now live in the [BlitzConnect SDK WebSocket reference](../../sdk/blitzconnect-api/websocket.md)** — prefer that page for up-to-date message formats and SDK usage.

```
wss://{baseurl}/api_interactive/ws?access_token={token}
```

> On production: `wss://{baseurl}/api_interactive/ws`

!!! warning "**Token in URL — Use HTTPS/ WSS in Production**"
    The access token is passed as a query parameter. In production, always use `wss://` (secure WebSocket) to encrypt the connection and prevent credential leakage.

!!! tip "**Handle Reconnection**"
    WebSocket connections can drop. Implement reconnection logic in your client to re-subscribe to events (e.g., send `AllSubscribe` after reconnecting).

!!! note "**DecodedData is a parsed object**"
    On the current gateway build, `DecodedData` is delivered as an **already-parsed JSON object** inside the outer message — no second parse needed. Older builds sent it as a JSON *string* inside JSON; support both if you target legacy deployments.

# Connection

```javascript
var ws = new WebSocket("wss://{baseurl}/api_interactive/ws?access_token=YOUR_TOKEN");
```

## Connection Acknowledgment

Immediately after connecting, the server sends a plain-JSON ack (no `MessageCode`):

```json
{
    "type": "connection",
    "status": "connected",
    "sessionId": "5cef71999ef1495ab39a99ea28ea7632"
}
```

After sending `AllSubscribe`, the server replays the current session state: a `50000` ack followed by `70000` order events and `80000`/`90000` statistics snapshots.

# Actions

```json
{"action": "AllSubscribe"}
```

| Action                          | Event codes subscribed                  |
| ------------------------------- | --------------------------------------- |
| `OrderSubscribe`                | 70000                                   |
| `OrderUnsubscribe`              | Unsubscribes 70000                      |
| `StatisticSubscribe`            | 50000                                   |
| `StatisticUnsubscribe`          | Unsubscribes 50000                      |
| `StrategyStatisticSubscribe`    | 80000                                   |
| `StrategyStatisticUnsubscribe`  | Unsubscribes 80000                      |
| `InstrumentStatisticSubscribe`  | 90000                                   |
| `InstrumentStatisticUnsubscribe`| Unsubscribes 90000                      |
| `AllSubscribe`                  | Subscribes 50000, 70000, 80000, 90000   |
| `AllUnsubscribe`                | Unsubscribes all                        |

# Message Format

```json
{
    "MessageCode": 70000,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "DecodedData": { ... }
}
```

# Message Fields

| Field          | Type   | Description                                                       |
| -------------- | ------ | ----------------------------------------------------------------- |
| `MessageCode`  | int    | 50000=Statistic, 70000=Order, 80000=StrategyStatistic, 90000=InstrumentStatistic |
| `EntityId`     | string | Entity identifier                                                 |
| `DecodedData`  | object | Parsed payload object (Order/Statistic data)                      |

# Statistic Messages (50000)

Code `50000` is observed in two forms on UAT:

**Bare acknowledgment** — right after subscribing, with no payload:

```json
{
    "MessageCode": 50000,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed"
}
```

**Statistics snapshot** — during active trading, carrying the strategy statistics snapshot. The field layout is identical to [Strategy Statistics (80000)](#strategy-statistics-fields); handle both forms in your client.

# Order Updates (70000) {#order-updates}

Sent on every order state change (`PendingNew`, `New`, `Filled`, `Cancelled`, ...) for orders covered by `OrderSubscribe` / `AllSubscribe`.

The `DecodedData` object has the same shape as the [Order Object Fields](orders.md#order-object-fields). Real example — an order cancelled at the exchange:

```json
{
    "MessageCode": 70000,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "DecodedData": {
        "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
        "CorrelationOrderId": "order_1787551916122",
        "Account": "",
        "CTCLId": "1111111112",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "UserId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
        "UserLoginId": "",
        "ExchangeOrderId": "3359587169630840",
        "ExecutionId": "3359587169661271",
        "InstrumentName": "RELIANCE",
        "ExchangeSegment": "NSECM",
        "OrderStatus": "Cancelled",
        "RejectTypeReason": "",
        "LastTradedQuantity": 0,
        "LeavesQuantity": 1,
        "OrderPrice": 1600,
        "OrderTriggerPrice": 0,
        "OrderStopPrice": 0,
        "OrderQuantity": 1,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "AverageTradedPrice": 0,
        "BlitzOrderId": 224095020950000010,
        "IsOrderCompleted": true,
        "ExchangeInstrumentId": 110010000002885,
        "InstrumentType": "Futures",
        "ExecutionType": "Cancelled",
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "TimeInForce": "GFD",
        "OrderExpiryDate": 1787551922520,
        "OrderGeneratedDateTime": 1787551922520,
        "LastRequestDateTime": 1787551922520,
        "ExchangeTransactTime": 1787551922000,
        "OrderModificationCount": 0,
        "OrderTradeCount": 0,
        "AverageTradedValue": 0,
        "IsFictiveOrder": false,
        "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyInstanceId": "2449971d-bfce-4c79-b5be-3762d851ef40",
        "StrategyName": "Manual Trading",
        "StrategyInstanceName": "NSECM|RELIANCE",
        "IVObjectName": "IV",
        "InstrumentId": 110010000002885,
        "OrderSide": "Sell",
        "OrderTag": "NormalOrder",
        "AlgoId": "123456789",
        "AlgoCategoryId": "123654789123",
        "ClearingFirmId": "",
        "PANId": "PLOSA0987F",
        "UserText": "",
        "RMSTag": "General",
        "SequenceNumber": 49
    }
}
```

# Strategy Statistics (80000) {#strategy-statistics}

Per-strategy-instance position and trading statistics. Pushed after `StrategyStatisticSubscribe` / `AllSubscribe` and on every subsequent change.

```json
{
    "MessageCode": 80000,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "DecodedData": {
        "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
        "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName": "Manual Trading",
        "StrategyInstanceId": "2449971d-bfce-4c79-b5be-3762d851ef40",
        "StrategyInstanceName": "NSECM|RELIANCE",
        "ExchangeClientId": "Algo123",
        "InstrumentId": 110010000002885,
        "IVName": "IV",
        "ExchangeSegment": "NSECM",
        "InstrumentName": "RELIANCE",
        "NonAckBuyQuantity": 0,
        "NonAckSellQuantity": 0,
        "OpenBuyQuantity": 0,
        "OpenSellQuantity": 0,
        "ShortPosition": 0,
        "LongPosition": 0,
        "NetPosition": 0,
        "NetPositionInLot": 0,
        "OrderCount": 4,
        "TradeCount": 0,
        "PreviousNetPosition": 0,
        "TradeBuyValue": 0,
        "TradeSellValue": 0,
        "TradeNetValue": 0,
        "AvgBuy": 0,
        "AvgSell": 0,
        "AvgNet": 0,
        "OPBuyPrice": 0,
        "OPSellPrice": 0,
        "Exposure": 0,
        "UnRealized": 0,
        "Realized": 0,
        "LastTradedPrice": 1467,
        "OpenPositionTradeValue": 0,
        "TurnoverValue": 0,
        "TurnoverValueOptions": 0
    }
}
```

## Strategy Statistics Fields

Key fields of `DecodedData`:

| Field                   | Type   | Description                                  |
| ----------------------- | ------ | -------------------------------------------- |
| `StrategyId`            | string | Strategy identifier                          |
| `StrategyInstanceId`    | string | Strategy instance identifier                 |
| `StrategyInstanceName`  | string | Instance name (e.g. `NSECM\|RELIANCE`)       |
| `ExchangeClientId`      | string | Exchange client ID                           |
| `InstrumentId`          | long   | Internal instrument ID                       |
| `InstrumentName`        | string | Instrument name                              |
| `NonAckBuyQuantity`     | int    | Unacknowledged buy quantity                  |
| `OpenBuyQuantity`       | int    | Open buy quantity                            |
| `LongPosition`          | int    | Long position                                |
| `ShortPosition`         | int    | Short position                               |
| `NetPosition`           | int    | Net position                                 |
| `OrderCount`            | int    | Order count for the instance                 |
| `TradeCount`            | int    | Trade count for the instance                 |
| `AvgBuy` / `AvgSell` / `AvgNet` | number | Average traded prices                |
| `Exposure`              | number | Current exposure value                       |
| `UnRealized` / `Realized` | number | Unrealized / realized P&L                  |
| `LastTradedPrice`       | number | Last traded price of the instrument          |

# Instrument Statistics / RMS (90000) {#instrument-statistics}

Per-instrument position and RMS data. Pushed after `InstrumentStatisticSubscribe` / `AllSubscribe` and on position changes.

```json
{
    "MessageCode": 90000,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "DecodedData": {
        "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
        "ExchangeClientId": "Algo123",
        "RMSEntityCode": "",
        "InstrumentId": 110010000002885,
        "ExchangeSegment": "NSECM",
        "InstrumentName": "RELIANCE",
        "NonAckBuyQuantity": 0,
        "NonAckSellQuantity": 0,
        "OpenBuyQuantity": 1,
        "OpenSellQuantity": 0,
        "ShortPosition": 0,
        "LongPosition": 1,
        "NetPosition": 1,
        "NetPositionInLot": 1,
        "OrderCount": 8,
        "TradeCount": 1,
        "PreviousNetPosition": 0,
        "TradeBuyValue": 1460.4,
        "TradeSellValue": 0,
        "TradeNetValue": -1460.4,
        "AvgBuy": 1460.4,
        "AvgSell": 0,
        "AvgNet": -1460.4,
        "Exposure": 4400.1,
        "UnRealized": 6.3,
        "Realized": 0,
        "LastTradedPrice": 1466.7,
        "OpenPositionTradeValue": 1460.4,
        "TurnoverValue": 1460.4,
        "TurnoverValueOptions": 0
    }
}
```

Field layout is the same as [Strategy Statistics Fields](#strategy-statistics-fields), plus `RMSEntityCode`.
