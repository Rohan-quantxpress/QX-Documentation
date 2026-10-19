# Option Chain

Fetch option chain data for a given symbol and expiry date, including strike prices with Call and Put contract details and greeks.

=== "Python"

    ```python
    from blitzsdk import MarketDataApiClient

    client = MarketDataApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    chain = client.get_option_chain("NIFTY", "2026-07-28")
    print(chain)
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Common;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var mdClient = new BlitzMarketDataApiClient(config);
    await mdClient.LoginAsync();

    var chain = await mdClient.GetOptionChainAsync("NIFTY", "2026-07-28");
    foreach (var strike in chain.Data.Chains)
        Console.WriteLine($"{strike.StrikePrice}: call ltp {strike.CallOption?.Ltp}, put ltp {strike.PutOption?.Ltp}");
    ```

---

## Endpoint

```
POST /marketfeed/optionChain
```

### Request

```bash
curl -X POST "https://{baseurl}/md-api/marketfeed/optionChain" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"Symbol": "NIFTY", "ExpiryDate": "2026-07-28"}'
```

### Request body

| Field | Type | Description |
|-------|------|-------------|
| `Symbol` | string | Underlying instrument symbol |
| `ExpiryDate` | string | Expiry date (`YYYY-MM-DD`) |

### Response

A successful request returns the spot price of the underlying, the at-the-money (ATM) strike, and a `chains` array with `callOption` / `putOption` greeks per strike:

```json
{
  "status": "success",
  "data": {
    "spotPrice": 24813.5,
    "expiryDate": "2026-07-28",
    "atm": 24800,
    "chains": [
      {
        "strikePrice": 23800,
        "callOption": {
          "gamma": 0.001,
          "vega": 12.5,
          "theta": -5.2,
          "delta": 0.85,
          "oiPercentage": 2.5,
          "oi": 15000,
          "ltp": 1042.8,
          "iv": 0.2,
          "price": 1013.5,
          "rho": 0.1
        },
        "putOption": {
          "gamma": 0.001,
          "vega": 11.2,
          "theta": -4.8,
          "delta": -0.15,
          "oiPercentage": -1.2,
          "oi": 8000,
          "ltp": 8.85,
          "iv": 0.22,
          "price": 9.1,
          "rho": -0.05
        }
      }
    ]
  }
}
```

### Root fields

| Field | Type | Description |
|-------|------|-------------|
| `spotPrice` | double | Current spot price of the underlying |
| `expiryDate` | string | Expiry the chain was requested for |
| `atm` | double | At-the-money strike price |
| `chains` | object[] | Array of strike-level entries |

### Greeks fields (`callOption` / `putOption`)

| Field | Type | Description |
|-------|------|-------------|
| `ltp` | double | Last traded price |
| `price` | double | Theoretical option price |
| `iv` | double | Implied volatility |
| `delta` | double | Delta |
| `gamma` | double | Gamma |
| `theta` | double | Theta |
| `vega` | double | Vega |
| `rho` | double | Rho |
| `oi` | long | Open interest |
| `oiPercentage` | double | Open interest as a percentage |

### Error response

When the expiry is invalid, the live server returns a `400` Problem Details payload:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "Bad Request",
  "status": 400,
  "traceId": "00-c54873e86f1a0ee04794569ece240bcb-bfe9a299393be357-00"
}
```
