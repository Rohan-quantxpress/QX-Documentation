# Holdings

Retrieve all portfolio / demat holdings of the client.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    holdings = client.get_holdings()
    # Returns a list: [{...}, ...]
    for h in holdings:
        print(h["Ticker"], h["Quantity"], h["BuyAvgPrice"])
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var holdings = await client.GetHoldingsAsync();
    foreach (var h in holdings.Data)
    {
        Console.WriteLine($"{h.Ticker} Qty:{h.Quantity} Avg:{h.BuyAvgPrice}");
    }
    ```

---

## Endpoint

```
GET /portfolio/holdings
```

### Request

```bash
curl "https://{baseurl}/api_interactive/api/v1/portfolio/holdings" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

The response is a **bare JSON array** of holding objects:

```json
[
  {
    "UserID": "SAM001",
    "ISIN": "INE002A01018",
    "InstrumentName": "RELIANCE",
    "Ticker": "RELIANCE",
    "InstrumentId": "110010000002885",
    "ExchangeSegment": "NSECM",
    "Quantity": 10,
    "BuyAvgPrice": 2980.5
  },
  {
    "UserID": "SAM001",
    "ISIN": "INE467B01029",
    "InstrumentName": "TCS",
    "Ticker": "TCS",
    "InstrumentId": "110010000011536",
    "ExchangeSegment": "NSECM",
    "Quantity": 2,
    "BuyAvgPrice": 3540.0
  }
]
```

### Holding fields

| Field | Type | Description |
|-------|------|-------------|
| `UserID` | string | User identifier |
| `ISIN` | string | International Securities Identification Number |
| `InstrumentName` | string | Instrument name |
| `Ticker` | string | Ticker symbol |
| `InstrumentId` | string | Internal instrument ID |
| `ExchangeSegment` | string | Exchange segment |
| `Quantity` | double | Total quantity held |
| `BuyAvgPrice` | double | Average buy price |