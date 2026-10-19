# Response structure

All responses from the API are JSON with the content-type `application/json` unless explicitly stated otherwise.

## Successful request

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status": "success",
    "data": {}
}
```

A successful `200 OK` response always has a JSON response body with a `status` key with the value `"success"`. The `data` key contains the full response payload.

## Failed request

```
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "status": "error",
    "message": "Error description",
    "error_type": "InputException"
}
```

A failure response is preceded by the corresponding `40x` or `50x` HTTP header. The `status` key contains `"error"`. The `message` key contains a textual description and `error_type` contains the exception name.

## SDK return format

=== "Python"

    Methods on `InteractiveApiClient` / `MarketDataApiClient` return the **parsed JSON** exactly as the server sent it — no extra wrapper:

    ```python
    resp = client.place_order(order)
    # resp == {"status": "success", "message": "...",
    #          "data": {"blitzOrderId": 218100636600000022, "correlationOrderId": "..."}}
    blitz_order_id = resp["data"]["blitzOrderId"]

    orders = client.get_orders()   # list[dict] of order objects
    hist = client.get_historical_data("RELIANCE", "D")   # list of candle dicts
    ```

    Failures raise exceptions instead of returning error payloads:

    ```python
    from blitzsdk.common.exceptions import BlitzSDKException, AuthenticationError, RequestError

    try:
        client.get_orders()
    except AuthenticationError as e:      # login failed (bad appKey/userId)
        ...
    except RequestError as e:             # HTTP failure; e.status_code available
        print(e.status_code)              # 0 = transport failure / timeout
    except BlitzSDKException:
        ...                               # base class for both
    ```

    `ValueError` is raised for invalid local input (unknown instrument symbol, malformed `EXCHANGE|NAME` string).

=== "C#"

    Each method returns a **strongly-typed model**. Envelope-style calls return `BlitzApiResponse<T>`:

    ```csharp
    public class BlitzApiResponse<T>
    {
        public string Status { get; }      // "success" | "error"
        public T? Data { get; }            // strongly-typed payload
        public string? Message { get; }
    }
    ```

    while list-oriented endpoints unwrap into dedicated response types:

    | Method | Return type |
    |---|---|
    | `GetOrdersAsync` / `GetOpenOrdersAsync` | `OrdersResponse` (`List<OrderEntry> Data`) |
    | `GetPositionsAsync` | `PositionsResponse` (`Dictionary<string, List<Position>> Data`) |
    | `GetTradesAsync` | `TradesResponse` (`List<JsonElement> Data`) |
    | `GetStatisticsAsync` | `StrategyStatisticsResponse` (`List<StrategyStatistics> Data`) |
    | `GetStatisticsByInstanceAsync` | `StrategyInstanceStatisticsResponse` (`Dictionary<string, List<StrategyStatistics>> Data`) |
    | `GetLtpAsync` | `LtpResponse` (`Dictionary<string, LtpEntry> Data`) |
    | `GetMarketQuoteAsync` | `MarketQuoteResponse` (`Dictionary<string, MarketQuoteEntry> Data`) |
    | `PlaceOrderAsync` | `BlitzApiResponse<PlaceOrderData>` (`BlitzOrderId`, `CorrelationOrderId`) |
    | `ModifyOrderAsync` / `CancelOrderAsync` / `SendSignalsAsync` | `GatewayResponse` / `ModifyOrderResponse` (`Status`, `Message`, `Data`) |

    Any non-success HTTP status, empty body, or deserialization failure throws `BlitzConnectException`:

    ```csharp
    try
    {
        var orders = await client.GetOrdersAsync();
    }
    catch (BlitzConnectException ex)
    {
        Console.WriteLine(ex.HttpStatusCode);     // 0 when transport-level
        Console.WriteLine(ex.RawResponseBody);    // raw server body, if any
    }
    ```

## Data types

| JSON type       | Notes                                              |
| --------------- | -------------------------------------------------- |
| `string`        | Text values                                        |
| `int`           | Integer values (instrument IDs, quantities)         |
| `long`          | Large integer values (Blitz order IDs, timestamps) |
| `float` / `double` | Decimal values (prices, LTP)                     |
| `bool`          | Boolean flags                                      |

Timestamp strings are represented in the format `dd-mm-yyyy HH:mm:ss` in Indian timezone (IST, UTC+5:30). Numeric timestamps are epoch milliseconds.
