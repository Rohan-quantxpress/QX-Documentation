# Market Quote

Fetch a full market quote with OHLC, volume, and bid/ask depth for one or more instruments.

=== "Python"

    ```python
    from blitzsdk.marketdata import MarketDataApiClient

    client = MarketDataApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    quotes = client.get_quote([110010000011536, 110010000002885])
    print(quotes)
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var mdClient = new BlitzMarketDataApiClient(config);
    await mdClient.LoginAsync();

    var quotes = await mdClient.GetMarketQuoteAsync(new List<long> { 110010002000001, 110010000002885 });
    Console.WriteLine(quotes);
    ```

---

## Endpoint

```
POST /marketfeed/quote
```

### Request

```bash
curl -X POST "https://{baseurl}/md-api/marketfeed/quote" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"InstrumentIds": [110010000014366]}'
```

### Request body

| Field | Type | Description |
|-------|------|-------------|
| `InstrumentIds` | integer[] | List of instrument IDs |

### Response

```json
{
  "data": {
    "110010000014366": {
      "instrumentId": 110010000014366,
      "exchangeSegment": 1,
      "exchangeInstrumentId": 14366,
      "instrumentName": "IDEA",
      "timestamp": 1770728956,
      "ltp": 11.51,
      "ltq": 2,
      "ltt": 1770728955,
      "atp": 11.55,
      "vtt": 268781653,
      "tbq": 28591060,
      "tsq": 80834031,
      "oi": 0,
      "open": 11.58,
      "high": 11.67,
      "low": 11.48,
      "close": 11.58,
      "bidLevel": [
        { "index": 1, "price": 11.5, "quantity": 855978, "orders": 28 },
        { "index": 2, "price": 11.49, "quantity": 1544481, "orders": 78 },
        { "index": 3, "price": 11.48, "quantity": 2673732, "orders": 380 },
        { "index": 4, "price": 11.47, "quantity": 979037, "orders": 127 },
        { "index": 5, "price": 11.46, "quantity": 837048, "orders": 126 }
      ],
      "askLevel": [
        { "index": 1, "price": 11.51, "quantity": 1014281, "orders": 63 },
        { "index": 2, "price": 11.52, "quantity": 815256, "orders": 70 },
        { "index": 3, "price": 11.53, "quantity": 1156649, "orders": 70 },
        { "index": 4, "price": 11.54, "quantity": 1867230, "orders": 120 },
        { "index": 5, "price": 11.55, "quantity": 1500815, "orders": 218 }
      ]
    }
  },
  "status": "success"
}
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `"success"` or `"error"` |
| `data` | object | Map of instrument ID to quote data |
| `data.{instrumentId}` | object | Instrument ID as string key |
| `instrumentId` | integer | Instrument identifier |
| `exchangeSegment` | int | Exchange segment ID |
| `exchangeInstrumentId` | int | Exchange instrument ID |
| `instrumentName` | string | Instrument name |
| `timestamp` | long | Tick timestamp (epoch) |
| `ltp` | double | Last traded price |
| `ltq` | int | Last traded quantity |
| `ltt` | long | Last traded time (epoch) |
| `atp` | double | Average traded price |
| `vtt` | long | Volume today |
| `tbq` | long | Total buy quantity |
| `tsq` | long | Total sell quantity |
| `oi` | long | Open interest |
| `open` | double | Opening price |
| `high` | double | Highest price |
| `low` | double | Lowest price |
| `close` | double | Closing price |
| `bidLevel` | array | Best bid depth levels (5 levels) |
| `askLevel` | array | Best ask depth levels (5 levels) |

### Depth level fields

| Field | Type | Description |
|-------|------|-------------|
| `index` | int | Level number (1 = best) |
| `price` | double | Price at this level |
| `quantity` | long | Total quantity at this level |
| `orders` | int | Number of orders at this level |
