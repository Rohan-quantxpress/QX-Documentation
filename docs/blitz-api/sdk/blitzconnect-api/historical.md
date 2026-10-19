# Historical Candle Data

Fetch historical OHLC candle data for a specified instrument and time interval.

=== "Python"

    ```python
    from blitzsdk.marketdata import MarketDataApiClient

    client = MarketDataApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    hist = client.get_historical_data("NIFTY", "D")
    print(hist)
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var mdClient = new BlitzMarketDataApiClient(config);
    await mdClient.LoginAsync();

    var hist = await mdClient.GetHistoricalDataAsync("NIFTY", "D");
    Console.WriteLine(hist);
    ```

---

## Endpoint

```
POST /marketfeed/historicalData
```

### Request

```bash
curl -X POST "https://{baseurl}/md-api/marketfeed/historicalData" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"Instrument": "NIFTY", "interval": "D"}'
```

### Request fields

| Field | Type | Description |
|-------|------|-------------|
| `Instrument` | string | Instrument symbol (e.g. `NIFTY`, `TCS`) |
| `interval` | string | Candle interval |

### Supported intervals

| Interval | Description |
|----------|-------------|
| `1` | 1 minute |
| `5` | 5 minutes |
| `15` | 15 minutes |
| `30` | 30 minutes |
| `60` | 1 hour |
| `D` | Daily |

> The daily interval (`"D"`) is verified against UAT. JSON keys are case-insensitive on the server — the SDKs send `"Instrument"` (capitalized) with `"interval"` (lowercase).

### Response

When no data is available (e.g. invalid instrument or unbounded interval), the live server returns an empty array:

```json
[]
```

Otherwise, the response is an array of candles:

```json
[
    {
        "open": 18490.00,
        "high": 18520.50,
        "low": 18480.10,
        "close": 18510.30,
        "volume": 1250000,
        "oi": 850000,
        "timestamp": "06-07-2026 09:15:00"
    },
    {
        "open": 18510.30,
        "high": 18535.00,
        "low": 18505.00,
        "close": 18530.45,
        "volume": 980000,
        "oi": 850500,
        "timestamp": "06-07-2026 09:16:00"
    }
]
```

### Response fields (per candle)

| Field | Type | Description |
|-------|------|-------------|
| `open` | double | Opening price |
| `high` | double | Highest price |
| `low` | double | Lowest price |
| `close` | double | Closing price |
| `volume` | long | Volume traded |
| `oi` | long | Open interest |
| `timestamp` | string | Candle time (`DD-MM-YYYY HH:mm:ss`) |
