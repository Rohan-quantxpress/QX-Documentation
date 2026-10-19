# Trades

Returns trade-level execution information for completed or partially completed orders.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    trades = client.get_trades()
    for trade in trades:
        print(trade["BlitzOrderId"], trade["InstrumentName"], trade["OrderStatus"])
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var trades = await client.GetTradesAsync();
    foreach (var trade in trades)
    {
        Console.WriteLine($"{trade.BlitzOrderId} {trade.InstrumentName} {trade.OrderStatus}");
    }
    ```

---

## Endpoint

```
GET /trades
```

### Request

```bash
curl "https://{baseurl}/api_interactive/api/v1/trades" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

The trades endpoint returns a bare JSON array of order-log entries — one row per fill event. The shape is identical to [`GET /orders`](orders.md#get-orders); each entry is a full order-log object.

When there are no trades yet, the endpoint returns an empty array:

```json
[]
```

When trades exist, the response contains every fill event as a full order-log entry:

```json
[
  {
    "Id": 35,
    "CorrelationOrderId": "doc-sample-trade-001",
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyInstanceId": "23335e0b-4386-4c21-b76b-cbfd3891fc95",
    "StrategyInstanceName": "NSECM|IDEA",
    "StrategyName": "Manual Trading",
    "IVObjectName": "IV",
    "InstrumentId": 110010000014366,
    "ExchangeSegment": "NSECM",
    "ExchangeInstrumentId": 14366,
    "InstrumentName": "IDEA",
    "InstrumentType": "Equity",
    "BlitzOrderId": 217105131760000028,
    "ExchangeOrderId": "3380361399023766",
    "ExecutionId": "3380361399023772",
    "Account": "",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "OrderSide": "Buy",
    "OrderStatus": "Filled",
    "OrderQuantity": 50,
    "OrderPrice": 11.50,
    "OrderStopPrice": 0,
    "OrderTriggerPrice": 0,
    "LastTradedQuantity": 50,
    "LastTradedPrice": 11.51,
    "CumulativeQuantity": 50,
    "LeavesQuantity": 0,
    "TIF": "GFD",
    "OrderExpiryDate": 1789624536849,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "OrderGeneratedDateTime": 1789624536849,
    "LastRequestDateTime": 1789624536849,
    "ExchangeTransactTime": 1789644336000,
    "OrderModificationCount": 0,
    "OrderTradeCount": 1,
    "AverageTradedPrice": 11.51,
    "AverageTradedValue": 575.50,
    "IsFictiveOrder": false,
    "RejectType": "None",
    "RejectTypeReason": "",
    "OrderTag": "NormalOrder",
    "CTCLId": "1111111112",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "IsOrderCompleted": true,
    "UserText": "",
    "ExecutionType": "Fill",
    "StrategyTag": "General",
    "SequenceNumber": 51
  }
]
```

Each entry carries the full order object — the fields that carry trade information are highlighted below.

### Key trade-related fields

| Field | Type | Description |
|-------|------|-------------|
| `BlitzOrderId` | long | System-assigned order ID |
| `InstrumentId` | long | Internal instrument ID |
| `InstrumentName` | string | Instrument name |
| `ExchangeSegment` | string | Exchange segment |
| `OrderSide` | string | Buy or Sell |
| `OrderStatus` | string | `Filled` (partial fill: `PartiallyFilled`) |
| `OrderQuantity` | int | Total order quantity |
| `CumulativeQuantity` | int | Filled quantity |
| `LastTradedQuantity` | int | Quantity filled in this trade event |
| `LastTradedPrice` | double | Execution price of this fill |
| `AverageTradedPrice` | double | Volume-weighted average fill price |
| `AverageTradedValue` | double | Total value of all fills |
| `OrderTradeCount` | int | Number of individual trades (fills) so far |
| `ExecutionType` | string | `Fill` for trades, `New`/`Replace`/`Cancelled` for non-trade events |
| `ExchangeTransactTime` | long | Exchange execution timestamp (epoch ms) |
| `SequenceNumber` | int | Monotonically increasing sequence number (unique across all events) |

---

# Trades by BlitzOrderId

Returns trade-level execution records for a single order, identified by its `blitzOrderId`.

```python
trades = client.get_trades_by_blitz_id(217105131760000028)
print(trades)
```

```csharp
var trades = await client.GetTradesByBlitzOrderIdAsync(217105131760000028);
Console.WriteLine(trades.Data.Count);
```

## Endpoint

```
GET /trades/{blitzOrderId}
```

### Request

```bash
curl "https://{baseurl}/api_interactive/api/v1/trades/217105131760000028" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

Returns a bare JSON array of order-log entries — one row per fill event for that order — in the same shape as [`GET /trades`](#trades). When no fills exist for the `blitzOrderId`, the endpoint returns `404 Not Found` with a message like `{"message": "No trades found for BlitzOrderId 217105131760000028."}`.

```json
[
  {
    "Id": 35,
    "CorrelationOrderId": "doc-sample-trade-001",
    "BlitzOrderId": 217105131760000028,
    "InstrumentName": "IDEA",
    "OrderSide": "Buy",
    "OrderStatus": "Filled",
    "CumulativeQuantity": 50,
    "AverageTradedPrice": 11.51,
    "AverageTradedValue": 575.50,
    "OrderTradeCount": 1,
    "ExecutionType": "Fill",
    "SequenceNumber": 51
  }
]
```

Entries are returned in ascending order of `Id` (oldest first).
