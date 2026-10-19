# Orders

The order APIs let you place orders of different types, modify and cancel pending orders, and retrieve the daily order book.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient
    from blitzsdk.interactive.models import OrderRequest

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    # Place an order
    order = OrderRequest(
        instrument_id=110010000002885,
        quantity=50,
        price=150.00,
        order_side="Buy",
        order_type="Limit",
        product="MIS",
        client_id="Algo123",
    )
    resp = client.place_order(order)
    print(resp["data"]["blitzOrderId"])

    # Get all orders
    orders = client.get_orders()

    # Get open orders only (PendingNew, New, partially filled)
    open_orders = client.get_open_orders()

    # Get the full lifecycle history for one order
    history = client.get_order_by_blitz_id(217105131760000028)

    # Modify an open/pending order
    client.modify_order({
        "BlitzOrderId": 217105131760000028,
        "ModifiedOrderQuantity": 75,
        "Price": 155.00,
        "StopPrice": 0,
        "ExchangeSegment": "NSEFO",
        "InstrumentId": 110010000002885,
    })

    # Cancel an order (exchange segment + instrument id resolve the exchange instrument id)
    client.cancel_order(
        instrument_id=110010000012018,
        blitz_order_id=227134718870000009,
        exchange_segment="NSECM",
    )
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Common;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    // Place an order
    var resp = await client.PlaceOrderAsync(new PlaceOrderRequest
    {
        Product = "MIS",
        OrderSide = "Buy",
        InstrumentId = 110010000002885,
        ExchangeSegment = "NSEFO",
        OrderType = "Limit",
        Quantity = 50,
        Price = 150.00,
        TifGtdDate = "2025-10-10",
        ClientId = "Algo123"
    });
    Console.WriteLine(resp.Data.BlitzOrderId);

    // Get all orders
    var orders = await client.GetOrdersAsync();

    // Get open orders only (PendingNew, New, partially filled)
    var openOrders = await client.GetOpenOrdersAsync();

    // Get the full lifecycle history for one order
    var history = await client.GetOrderByIdAsync(217105131760000028);

    // Modify an open/pending order
    await client.ModifyOrderAsync(new ModifyOrderRequest
    {
        BlitzOrderId = 217105131760000028,
        ModifiedOrderQuantity = 75,
        Price = 155.00,
        StopPrice = 0,
        ExchangeSegment = "NSEFO",
        InstrumentId = 110010000002885
    });

    // Cancel an order (ExchangeSegment + ExchangeInstrumentId required)
    await client.CancelOrderAsync(new CancelOrderRequest
    {
        BlitzOrderId = 227134718870000009,
        ExchangeSegment = "NSECM",
        ExchangeInstrumentId = 12018
    });
    ```

---

## Endpoint summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/orders/placeOrder` | Place an order |
| PUT | `/orders/modifyOrder` | Modify an open or pending order |
| DELETE | `/orders/cancelOrder` | Cancel an open or pending order |
| GET | `/orders` | Retrieve all orders for the day |
| GET | `/orders/openOrders` | Retrieve open orders |
| GET | `/orders/{BlitzOrderId}` | Retrieve a specific order by Blitz order ID |

## Placing an order

```
POST /orders/placeOrder
```

=== "By InstrumentId"

    ```bash
    curl -X POST "https://{baseurl}/api_interactive/api/v1/orders/placeOrder" \
      -H "Authorization: Bearer YOUR_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "CorrelationOrderId": "order_1787046999088",
        "InstrumentId": 110010000002885,
        "ExchangeSegment": "NSEFO",
        "Product": "MIS",
        "TIF": "GFD",
        "OrderType": "Limit",
        "Quantity": 50,
        "Price": 150.00,
        "OrderSide": "Buy",
        "DisclosedQuantity": 0,
        "StopPrice": 0,
        "ClientId": "Algo123"
      }'
    ```

=== "By Symbol (Weekly Option)"

    ```bash
    curl -X POST "https://{baseurl}/api_interactive/api/v1/orders/placeOrder" \
      -H "Authorization: Bearer YOUR_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "CorrelationOrderId": "order_1787046999088",
        "Symbol": "NSEFO|NIFTY25MAR261030CE",
        "ExchangeSegment": "NSEFO",
        "Product": "MIS",
        "TIF": "GFD",
        "OrderType": "Limit",
        "Quantity": 75,
        "Price": 85.00,
        "OrderSide": "Buy",
        "DisclosedQuantity": 0,
        "StopPrice": 0,
        "ClientId": "Algo123",
        "TiF_GTD_Date": "2025-10-10"
      }'
    ```

> The `TiF_GTD_Date` field is **required** for every order. A weekly-expiry option trading symbol follows `[ExchangeSegment]|[Symbol][DDMMMYY][Strike][OptionType]` — e.g. `NSEFO|NIFTY25MAR261030CE` is the NIFTY **1030 CE weekly** option expiring **25 Mar 2026**. The expiry date indicates whether the contract is weekly or monthly.

=== "By ExchangeInstrumentId"

    ```bash
    curl -X POST "https://{baseurl}/api_interactive/api/v1/orders/placeOrder" \
      -H "Authorization: Bearer YOUR_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "CorrelationOrderId": "order_1787046999088",
        "ExchangeInstrumentId": 2885,
        "ExchangeSegment": "NSEFO",
        "Product": "MIS",
        "TIF": "GTD",
        "OrderType": "Market",
        "Quantity": 500,
        "Price": 10.1,
        "OrderSide": "Buy",
        "DisclosedQuantity": 0,
        "StopPrice": 0,
        "ClientId": "Algo123",
        "TiF_GTD_Date": "2025-10-10"
      }'
    ```

### Place order fields

| Field | Type | Description |
|-------|------|-------------|
| `CorrelationOrderId` | string | Optional. Client-generated ID used to correlate the order across systems; auto-generated if omitted. |
| `ExchangeSegment` | string | **Mandatory.** Exchange segment (e.g. `NSECM`, `NSEFO`). |
| `Symbol` | string | Optional. Trading symbol (e.g. `NSEFO\|NIFTY25MAR261030CE`). |
| `InstrumentId` | long | Optional. Internal Blitz instrument ID. |
| `ExchangeInstrumentId` | long | Optional. Exchange instrument ID / universal ID. |
| `Product` | string | Product type (`MIS`, `NRML`, `CNC`, `MTF`) |
| `OrderSide` | string | Order side (`Buy`, `Sell`) |
| `OrderType` | string | Order type (`Market`, `Limit`, `Stop`, `StopLimit`, `MarketIfTouched`, `MarketLimit`, `Pegged`) |
| `Quantity` | int | Order quantity |
| `Price` | double | Order price (for Limit / StopLimit orders) |
| `StopPrice` | double | Trigger price (for Stop / StopLimit orders) |
| `DisclosedQuantity` | int | Disclosed quantity |
| `TIF` | string | Time in force (`GFD`, `GTC`, `IOC`, `FOK`, `GTD`, `COL`) |
| `TiF_GTD_Date` | string | **Required.** Expiry date in `YYYY-MM-DD` format. Required for **every** order — even `GFD`/`GTC` orders must include it (e.g. today's date). GTD orders are auto-cancelled after this date. |
| `ClientId` | string | **Required.** Exchange client ID for the order. Validated against the session type (Individual / Proprietary / Dealer). |

> **Instrument identification**
> `ExchangeSegment` is always **mandatory**. Supply **exactly one** of `Symbol`, `InstrumentId` or `ExchangeInstrumentId` along with it:
> - `ExchangeSegment` + `Symbol` — e.g. `NSEFO` + `NSEFO\|RELIANCE24FEB261030CE`
> - `ExchangeSegment` + `InstrumentId` — e.g. `NSEFO` + `110010000002885`
> - `ExchangeSegment` + `ExchangeInstrumentId` — e.g. `NSEFO` + `2885`
>
> If the identifier is not found, the API returns an error with the reason (e.g. instrument not found).

> **Authentication**
> `UserId` is **not required** — it is stored in the API key session. `ClientId` is **required** for every order.

> **Case sensitivity**
> All string values are **case-sensitive** and must be sent exactly as documented.

### Place order response

```json
{
  "status": "success",
  "message": "request processed successfully",
  "data": {
    "blitzOrderId": 218100636600000022,
    "correlationOrderId": "order_1787046999088"
  }
}
```

## Get orders

```
GET /orders
```

```bash
curl "https://{baseurl}/api_interactive/api/v1/orders" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Response — an array of order objects (full field reference in [Order object](#order-object)):

```json
[
  {
    "Id": 28,
    "CorrelationOrderId": "postman-1789624534",
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
    "BlitzOrderId": 217105131760000009,
    "ExchangeOrderId": "3380361399023766",
    "ExecutionId": "3380361399023772",
    "Account": "",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "OrderSide": "Buy",
    "OrderStatus": "Cancelled",
    "OrderQuantity": 3,
    "OrderPrice": 11.01,
    "OrderStopPrice": 0,
    "OrderTriggerPrice": 0,
    "LastTradedQuantity": 0,
    "LastTradedPrice": 0,
    "CumulativeQuantity": 0,
    "LeavesQuantity": 3,
    "TIF": "GFD",
    "OrderExpiryDate": 1789624753383,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "OrderGeneratedDateTime": 1789624753383,
    "LastRequestDateTime": 1789624753383,
    "ExchangeTransactTime": 1789624753000,
    "OrderModificationCount": 1,
    "OrderTradeCount": 0,
    "AverageTradedPrice": 0,
    "AverageTradedValue": 0,
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
    "ExecutionType": "Cancelled",
    "StrategyTag": "General",
    "SequenceNumber": 23
  },
  {
    "Id": 27,
    "CorrelationOrderId": "postman-1789624534",
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
    "StrategyInstanceId": "23335e0b-4386-4c21-b76b-cbfd3891fc95",
    "StrategyInstanceName": "NSECM|IDEA",
    "StrategyName": "Manual Trading",
    "IVObjectName": "IV",
    "InstrumentId": 110010000014366,
    "ExchangeSegment": "NSECM",
    "ExchangeInstrumentId": 14366,
    "InstrumentName": "IDEA",
    "InstrumentType": "Equity",
    "BlitzOrderId": 217105131760000009,
    "ExchangeOrderId": "3380361399023766",
    "ExecutionId": "3380361399023770",
    "Account": "",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "OrderSide": "Buy",
    "OrderStatus": "PendingCancel",
    "OrderQuantity": 3,
    "OrderPrice": 11.01,
    "OrderStopPrice": 0,
    "OrderTriggerPrice": 0,
    "LastTradedQuantity": 0,
    "LastTradedPrice": 0,
    "CumulativeQuantity": 0,
    "LeavesQuantity": 3,
    "TIF": "GFD",
    "OrderExpiryDate": 1789624753383,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "OrderGeneratedDateTime": 1789624753383,
    "LastRequestDateTime": 1789624753383,
    "ExchangeTransactTime": 1789624753383,
    "OrderModificationCount": 1,
    "OrderTradeCount": 0,
    "AverageTradedPrice": 0,
    "AverageTradedValue": 0,
    "IsFictiveOrder": false,
    "RejectType": "None",
    "RejectTypeReason": "",
    "OrderTag": "NormalOrder",
    "CTCLId": "1111111112",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "IsOrderCompleted": false,
    "UserText": "",
    "ExecutionType": "Replace",
    "StrategyTag": "General",
    "SequenceNumber": 22
  }
]
```

## Get open orders

```
GET /orders/openOrders
```

```bash
curl "https://{baseurl}/api_interactive/api/v1/orders/openOrders" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

The response is the same array shape as [`GET /orders`](#get-orders) and contains only orders with an open status (`PendingNew`, `New`, partially filled). Each entry is the full order object:

```json
[
  {
    "Id": 54,
    "CorrelationOrderId": "doc-sample-open-001",
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
    "BlitzOrderId": 217105131760000023,
    "ExchangeOrderId": "",
    "ExecutionId": "",
    "Account": "",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "OrderSide": "Buy",
    "OrderStatus": "PendingNew",
    "OrderQuantity": 1,
    "OrderPrice": 9,
    "OrderStopPrice": 0,
    "OrderTriggerPrice": 0,
    "LastTradedQuantity": 0,
    "LastTradedPrice": 0,
    "CumulativeQuantity": 0,
    "LeavesQuantity": 1,
    "TIF": "GFD",
    "OrderExpiryDate": 1789624536849,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "OrderGeneratedDateTime": 1789624536849,
    "LastRequestDateTime": 1789624536849,
    "ExchangeTransactTime": 1789624536849,
    "OrderModificationCount": 0,
    "OrderTradeCount": 0,
    "AverageTradedPrice": 0,
    "AverageTradedValue": 0,
    "IsFictiveOrder": false,
    "RejectType": "None",
    "RejectTypeReason": "",
    "OrderTag": "NormalOrder",
    "CTCLId": "1111111112",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "IsOrderCompleted": false,
    "UserText": "",
    "ExecutionType": "New",
    "StrategyTag": "General",
    "SequenceNumber": 43
  },
  {
    "Id": 55,
    "CorrelationOrderId": "doc-sample-open-001",
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
    "BlitzOrderId": 217105131760000023,
    "ExchangeOrderId": "3380361399023766",
    "ExecutionId": "3380361399023769",
    "Account": "",
    "ClientId": "Algo123",
    "OrderType": "Limit",
    "OrderSide": "Buy",
    "OrderStatus": "New",
    "OrderQuantity": 1,
    "OrderPrice": 9,
    "OrderStopPrice": 0,
    "OrderTriggerPrice": 0,
    "LastTradedQuantity": 0,
    "LastTradedPrice": 0,
    "CumulativeQuantity": 0,
    "LeavesQuantity": 1,
    "TIF": "GFD",
    "OrderExpiryDate": 1789624536871,
    "OrderDisclosedQuantity": 0,
    "MinimumQuantity": 0,
    "OrderGeneratedDateTime": 1789624536871,
    "LastRequestDateTime": 1789624536871,
    "ExchangeTransactTime": 1789644336000,
    "OrderModificationCount": 0,
    "OrderTradeCount": 0,
    "AverageTradedPrice": 0,
    "AverageTradedValue": 0,
    "IsFictiveOrder": false,
    "RejectType": "None",
    "RejectTypeReason": "",
    "OrderTag": "NormalOrder",
    "CTCLId": "1111111112",
    "AlgoId": "123456789",
    "AlgoCategoryId": "123654789123",
    "ClearingFirmId": "",
    "PANId": "PLOSA0987F",
    "IsOrderCompleted": false,
    "UserText": "",
    "ExecutionType": "New",
    "StrategyTag": "General",
    "SequenceNumber": 44
  }
]
```

Entries are returned in ascending order of `Id` (oldest first). If there are no open or pending orders, the array is empty (`[]`).

## Get order by Blitz ID

```
GET /orders/{blitz_order_id}
```

```bash
curl "https://{baseurl}/api_interactive/api/v1/orders/225121947920000028" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

The response is an array containing the order history entries for the Blitz order ID — each `OrderStatus` change produces one entry, in ascending order of `Id` (oldest first, same array shape as [`GET /orders`](#get-orders)).

Example — the full lifecycle of an order that was placed, modified, and cancelled:

```json
[
    {
        "Id": 17,
        "CorrelationOrderId": "postman-1789624534",
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
        "BlitzOrderId": 217105131760000009,
        "ExchangeOrderId": "",
        "ExecutionId": "",
        "Account": "",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "OrderSide": "Buy",
        "OrderStatus": "PendingNew",
        "OrderQuantity": 1,
        "OrderPrice": 11,
        "OrderStopPrice": 0,
        "OrderTriggerPrice": 0,
        "LastTradedQuantity": 0,
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "LeavesQuantity": 1,
        "TIF": "GFD",
        "OrderExpiryDate": 1789624536849,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "OrderGeneratedDateTime": 1789624536849,
        "LastRequestDateTime": 1789624536849,
        "ExchangeTransactTime": 1789624536849,
        "OrderModificationCount": 0,
        "OrderTradeCount": 0,
        "AverageTradedPrice": 0,
        "AverageTradedValue": 0,
        "IsFictiveOrder": false,
        "RejectType": "None",
        "RejectTypeReason": "",
        "OrderTag": "NormalOrder",
        "CTCLId": "1111111112",
        "AlgoId": "123456789",
        "AlgoCategoryId": "123654789123",
        "ClearingFirmId": "",
        "PANId": "PLOSA0987F",
        "IsOrderCompleted": false,
        "UserText": "",
        "ExecutionType": "None",
        "StrategyTag": "General",
        "SequenceNumber": 16
    },
    {
        "Id": 18,
        "CorrelationOrderId": "postman-1789624534",
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
        "BlitzOrderId": 217105131760000009,
        "ExchangeOrderId": "3380361399023766",
        "ExecutionId": "3380361399023769",
        "Account": "",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "OrderSide": "Buy",
        "OrderStatus": "New",
        "OrderQuantity": 1,
        "OrderPrice": 11,
        "OrderStopPrice": 0,
        "OrderTriggerPrice": 0,
        "LastTradedQuantity": 0,
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "LeavesQuantity": 1,
        "TIF": "GFD",
        "OrderExpiryDate": 1789624536871,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "OrderGeneratedDateTime": 1789624536871,
        "LastRequestDateTime": 1789624536871,
        "ExchangeTransactTime": 1789644336000,
        "OrderModificationCount": 0,
        "OrderTradeCount": 0,
        "AverageTradedPrice": 0,
        "AverageTradedValue": 0,
        "IsFictiveOrder": false,
        "RejectType": "None",
        "RejectTypeReason": "",
        "OrderTag": "NormalOrder",
        "CTCLId": "1111111112",
        "AlgoId": "123456789",
        "AlgoCategoryId": "123654789123",
        "ClearingFirmId": "",
        "PANId": "PLOSA0987F",
        "IsOrderCompleted": false,
        "UserText": "",
        "ExecutionType": "New",
        "StrategyTag": "General",
        "SequenceNumber": 17
    },
    {
        "Id": 19,
        "CorrelationOrderId": "postman-1789624534",
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
        "BlitzOrderId": 217105131760000009,
        "ExchangeOrderId": "3380361399023766",
        "ExecutionId": "3380361399023769",
        "Account": "",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "OrderSide": "Buy",
        "OrderStatus": "PendingReplace",
        "OrderQuantity": 3,
        "OrderPrice": 11.01,
        "OrderStopPrice": 0,
        "OrderTriggerPrice": 0,
        "LastTradedQuantity": 0,
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "LeavesQuantity": 1,
        "TIF": "GFD",
        "OrderExpiryDate": 1789624608910,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "OrderGeneratedDateTime": 1789624608910,
        "LastRequestDateTime": 1789624608910,
        "ExchangeTransactTime": 1789624608910,
        "OrderModificationCount": 0,
        "OrderTradeCount": 0,
        "AverageTradedPrice": 0,
        "AverageTradedValue": 0,
        "IsFictiveOrder": false,
        "RejectType": "None",
        "RejectTypeReason": "",
        "OrderTag": "NormalOrder",
        "CTCLId": "1111111112",
        "AlgoId": "123456789",
        "AlgoCategoryId": "123654789123",
        "ClearingFirmId": "",
        "PANId": "PLOSA0987F",
        "IsOrderCompleted": false,
        "UserText": "",
        "ExecutionType": "New",
        "StrategyTag": "General",
        "SequenceNumber": 18
    },
    {
        "Id": 20,
        "CorrelationOrderId": "postman-1789624534",
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
        "BlitzOrderId": 217105131760000009,
        "ExchangeOrderId": "3380361399023766",
        "ExecutionId": "3380361399023770",
        "Account": "",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "OrderSide": "Buy",
        "OrderStatus": "Replaced",
        "OrderQuantity": 3,
        "OrderPrice": 11.01,
        "OrderStopPrice": 0,
        "OrderTriggerPrice": 0,
        "LastTradedQuantity": 0,
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "LeavesQuantity": 3,
        "TIF": "GFD",
        "OrderExpiryDate": 1789624608912,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "OrderGeneratedDateTime": 1789624608912,
        "LastRequestDateTime": 1789624608912,
        "ExchangeTransactTime": 1789624608000,
        "OrderModificationCount": 1,
        "OrderTradeCount": 0,
        "AverageTradedPrice": 0,
        "AverageTradedValue": 0,
        "IsFictiveOrder": false,
        "RejectType": "None",
        "RejectTypeReason": "",
        "OrderTag": "NormalOrder",
        "CTCLId": "1111111112",
        "AlgoId": "123456789",
        "AlgoCategoryId": "123654789123",
        "ClearingFirmId": "",
        "PANId": "PLOSA0987F",
        "IsOrderCompleted": false,
        "UserText": "",
        "ExecutionType": "Replace",
        "StrategyTag": "General",
        "SequenceNumber": 19
    },
    {
        "Id": 25,
        "CorrelationOrderId": "postman-1789624534",
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
        "BlitzOrderId": 217105131760000009,
        "ExchangeOrderId": "3380361399023766",
        "ExecutionId": "3380361399023770",
        "Account": "",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "OrderSide": "Buy",
        "OrderStatus": "PendingCancel",
        "OrderQuantity": 3,
        "OrderPrice": 11.01,
        "OrderStopPrice": 0,
        "OrderTriggerPrice": 0,
        "LastTradedQuantity": 0,
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "LeavesQuantity": 3,
        "TIF": "GFD",
        "OrderExpiryDate": 1789624753383,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "OrderGeneratedDateTime": 1789624753383,
        "LastRequestDateTime": 1789624753383,
        "ExchangeTransactTime": 1789624753383,
        "OrderModificationCount": 1,
        "OrderTradeCount": 0,
        "AverageTradedPrice": 0,
        "AverageTradedValue": 0,
        "IsFictiveOrder": false,
        "RejectType": "None",
        "RejectTypeReason": "",
        "OrderTag": "NormalOrder",
        "CTCLId": "1111111112",
        "AlgoId": "123456789",
        "AlgoCategoryId": "123654789123",
        "ClearingFirmId": "",
        "PANId": "PLOSA0987F",
        "IsOrderCompleted": false,
        "UserText": "",
        "ExecutionType": "Replace",
        "StrategyTag": "General",
        "SequenceNumber": 22
    },
    {
        "Id": 26,
        "CorrelationOrderId": "postman-1789624534",
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
        "BlitzOrderId": 217105131760000009,
        "ExchangeOrderId": "3380361399023766",
        "ExecutionId": "3380361399023772",
        "Account": "",
        "ClientId": "Algo123",
        "OrderType": "Limit",
        "OrderSide": "Buy",
        "OrderStatus": "Cancelled",
        "OrderQuantity": 3,
        "OrderPrice": 11.01,
        "OrderStopPrice": 0,
        "OrderTriggerPrice": 0,
        "LastTradedQuantity": 0,
        "LastTradedPrice": 0,
        "CumulativeQuantity": 0,
        "LeavesQuantity": 3,
        "TIF": "GFD",
        "OrderExpiryDate": 1789624753383,
        "OrderDisclosedQuantity": 0,
        "MinimumQuantity": 0,
        "OrderGeneratedDateTime": 1789624753383,
        "LastRequestDateTime": 1789624753383,
        "ExchangeTransactTime": 1789624753000,
        "OrderModificationCount": 1,
        "OrderTradeCount": 0,
        "AverageTradedPrice": 0,
        "AverageTradedValue": 0,
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
        "ExecutionType": "Cancelled",
        "StrategyTag": "General",
        "SequenceNumber": 23
    }
]
```

Each entry carries the full order object — the example above highlights only the fields that change across the lifecycle.

## Modify an order

```
PUT /orders/modifyOrder
```

```bash
curl -X PUT "https://{baseurl}/api_interactive/api/v1/orders/modifyOrder" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "BlitzOrderId": 225121947920000028,
    "ModifiedOrderQuantity": 75,
    "Price": 155.00,
    "StopPrice": 0,
    "ExchangeSegment": "NSEFO",
    "InstrumentId": 110010000002885
  }'
```

### Modify order fields

| Field | Type | Description |
|-------|------|-------------|
| `BlitzOrderId` | long | Order ID to modify |
| `ModifiedOrderQuantity` | int | New quantity |
| `Price` | double | New price |
| `StopPrice` | double | New stop/trigger price |
| `DisclosedQuantity` | int | New disclosed quantity |
| `OrderType` | string | New order type |
| `TIF` | string | New time in force |
| `TiF_GTD_Date` | string | New GTD expiry date |
| `ExchangeSegment` | string | **Mandatory.** Exchange segment of the instrument |
| `InstrumentId` / `Symbol` | long / string | Instrument identification (one of them) |

Only open or pending orders can be modified; already-executed orders return **403 Forbidden**.

### Modify order response

```json
{
  "status": "success",
  "message": "request processed successfully",
  "data": "successfully send modify order request to blitz server"
}
```

## Cancel an order

```
DELETE /orders/cancelOrder
```

!!! warning "Verified behavior"
    Cancelling requires `ExchangeSegment` + `ExchangeInstrumentId` + `blitzOrderId`. Passing the internal `instrumentId` instead returns `400` with `"invalid instrument"` — the gateway resolves the contract by its **exchange** instrument ID only.

```bash
curl -X DELETE "https://{baseurl}/api_interactive/api/v1/orders/cancelOrder?ExchangeSegment=NSECM&ExchangeInstrumentId=12018&blitzOrderId=227134718870000009" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Cancel order query parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `ExchangeSegment` | string | **Required.** Exchange segment (e.g. `NSECM`) |
| `ExchangeInstrumentId` | long | **Required.** Exchange instrument ID / universal ID |
| `blitzOrderId` | long | **Required.** System-assigned order ID to cancel |

### Cancel order response

```json
{
  "status": "success",
  "message": "request processed successfully",
  "data": "successfully send cancel order request to blitz server"
}
```

## Order object

Key fields on each returned order entry (all shown in the [Get orders](#get-orders) example above):

| Field | Type | Description |
|-------|------|-------------|
| `BlitzOrderId` | long | Unique order ID |
| `InstrumentId` | long | Internal instrument ID |
| `ExchangeSegment` / `ExchangeInstrumentId` | string / long | Instrument identification |
| `Product` | string | Product type |
| `OrderSide` | string | Buy or sell |
| `OrderType` | string | Order type |
| `OrderStatus` | string | Lifecycle state (`PendingNew`, `New`, `Filled`, `Cancelled`, `Rejected`) |
| `ExecutionType` | string | Last event applied (`None`, `New`, `Fill`, ...) |
| `OrderQuantity` / `CumulativeQuantity` / `LeavesQuantity` | int | Total / filled / remaining quantity |
| `OrderPrice` / `AverageTradedPrice` / `LastTradedPrice` | double | Prices |
| `OrderTriggerPrice` / `OrderStopPrice` | double | Trigger prices |
| `RejectType` / `RejectTypeReason` | string | Rejection info (if any) |
| `IsOrderCompleted` | bool | Terminal state flag |
| `OrderGeneratedDateTime` | long | Order creation time (epoch milliseconds) |

## Glossary

### Order types

| Value | Description |
|-------|-------------|
| `Market` | Market order |
| `Limit` | Limit order |
| `Stop` | Stop-loss order (triggers a market execution at the stop price) |
| `StopLimit` | Stop-limit order (triggers a limit order at the stop price) |
| `MarketIfTouched` | Market order that executes once a trigger price is touched |
| `MarketLimit` | Order that combines market and limit pricing |
| `Pegged` | Price pegged to a reference such as the exchange best bid/offer |

### Products

| Value | Description |
|-------|-------------|
| `MIS` | Intraday square-off |
| `NRML` | Normal / carry forward |
| `CNC` | Cash & carry (equity delivery) |
| `MTF` | Margin Trading Facility |

### Sides

| Value | Description |
|-------|-------------|
| `Buy` | Buy |
| `Sell` | Sell |

### Time in force (TIF)

| Value | Description |
|-------|-------------|
| `GFD` | Good for day (default for day orders) |
| `GTC` | Good till cancel |
| `IOC` | Immediate or cancel |
| `FOK` | Fill or kill |
| `GTD` | Good till date |
| `COL` | Good till cancel (exchange-specific variant) |

### Order statuses

| Value | Description |
|-------|-------------|
| `PendingNew` | Order accepted by OMS, not yet sent to exchange |
| `New` | Acknowledged by the exchange |
| `Filled` | Fully executed |
| `Cancelled` | Cancelled (by user or system) |
| `Rejected` | Rejected by OMS, RMS or the exchange |
