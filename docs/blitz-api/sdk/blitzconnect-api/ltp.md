# Last Traded Price (LTP)

Fetch the last traded price for one or more instruments.

=== "Python"

    ```python
    from blitzsdk.marketdata import MarketDataApiClient

    client = MarketDataApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    ltps = client.get_ltp([110010000011536, 110010000002885])
    print(ltps)
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var mdClient = new BlitzMarketDataApiClient(config);
    await mdClient.LoginAsync();

    var ltps = await mdClient.GetLtpAsync(new List<long> { 110010002000001, 110010000002885 });
    Console.WriteLine(ltps);
    ```

---

## Endpoint

```
POST /marketfeed/ltp
```

### Request

```bash
curl -X POST "https://{baseurl}/md-api/marketfeed/ltp" \
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
      "ltp": 11.51
    }
  },
  "status": "success"
}
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `"success"` or `"error"` |
| `data` | object | Map of instrument ID to LTP data |
| `data.{instrumentId}` | object | Instrument ID as string key |
| `instrumentId` | integer | Instrument identifier |
| `ltp` | double | Last traded price |
