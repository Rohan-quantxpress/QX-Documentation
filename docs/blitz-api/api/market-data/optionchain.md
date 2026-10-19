# Option Chain API

The **Option Chain API** provides the latest option chain data for a given symbol and expiry date. It returns detailed information about call and put options across multiple strike prices, including Live Market Prices, Open Interest, Implied Volatility, and the Option Greeks.

**Base URL:** `https://{baseurl}/md-api`

## Endpoint

| Method | URL                           |
|--------|-------------------------------|
| POST   | `/marketfeed/optionChain`     |

## Headers

| Header        | Value                 | Required |
|---------------|-----------------------|----------|
| Content-Type  | application/json      | Yes      |
| Authorization | Bearer {access_token} | Yes      |

## Request Payload

You must provide the underlying asset symbol and the specific expiration date you want to query.

```json
{
  "symbol": "NIFTY",
  "expiryDate": "2025-05-29"
}
```

### Parameters

| Parameter    | Type   | Required | Description                                                              |
|--------------|--------|----------|--------------------------------------------------------------------------|
| `symbol`     | string | Yes      | The underlying asset symbol (e.g., `NIFTY`, `BANKNIFTY`, or an equity symbol like `RELIANCE`). |
| `expiryDate` | string | Yes      | The exact expiry date formatted as `YYYY-MM-DD`.                         |

## Examples

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/optionChain' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -d '{
    "symbol": "NIFTY",
    "expiryDate": "2025-05-29"
  }'
```

**Empty Payload for User**

```json
{
  "symbol": "",
  "expiryDate": ""
}
```

---

## Understanding the Response

A successful request returns the spot price of the underlying asset, the At-The-Money (ATM) strike, and an array of `chains` containing Call (CE) and Put (PE) data for each available strike price.

```json
{
  "status": "success",
  "data": {
    "spotPrice": 24813.5,
    "expiryDate": "2025-05-29",
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

### Root Fields

| Field    | Type   | Description                                              |
|----------|--------|----------------------------------------------------------|
| `status` | string | Status of the request (`success` or `error`).            |
| `data`   | object | Contains the spot price and the option chain array.      |

### Error Response

When the requested `expiryDate` is invalid or no option chain is available for the given symbol and expiry, the API returns `400 Bad Request` using the HTTP Problem Details format:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "Bad Request",
  "status": 400,
  "traceId": "00-c54873e86f1a0ee04794569ece240bcb-bfe9a299393be357-00"
}
```

Verify that `expiryDate` matches a valid traded expiry (`YYYY-MM-DD`) and that options exist for the `symbol` on that date.

### Data Fields

| Field        | Type   | Description                                                             |
|--------------|--------|-------------------------------------------------------------------------|
| `spotPrice`  | float  | Current Live Market Price (LTP) of the underlying asset.                |
| `expiryDate` | string | Expiry date of the requested options contract (`YYYY-MM-DD`).           |
| `atm`        | float  | At-The-Money strike price (the strike closest to the `spotPrice`).      |
| `chains`     | array  | List of option data mapped by different strike prices.                  |

### Chain Fields

| Field        | Type   | Description                                                        |
|--------------|--------|--------------------------------------------------------------------|
| `strikePrice`| float  | Strike price of the option contract.                               |
| `callOption` | object | Call option (**CE**) data for this specific strike.                 |
| `putOption`  | object | Put option (**PE**) data for this specific strike.                 |

### Call / Put Option Fields (CE & PE)

> **Understanding Greeks:** Greeks (`delta`, `gamma`, `theta`, `vega`, `rho`) are calculated dynamically based on live market conditions and are key indicators for analyzing risk and price sensitivity in options trading.

| Field          | Type  | Description                                                    |
|----------------|-------|----------------------------------------------------------------|
| `ltp`          | float | Last traded price of the option.                               |
| `price`        | float | Last traded or closing price.                                  |
| `iv`           | float | Implied volatility of the option.                              |
| `delta`        | float | Sensitivity of option price to underlying price changes.       |
| `gamma`        | float | Rate of change of delta.                                       |
| `theta`        | float | Time decay of the option per day.                              |
| `vega`         | float | Sensitivity to volatility changes (1% change in IV).           |
| `rho`          | float | Sensitivity to interest rate changes.                          |
| `oi`           | float | Open interest.                                                 |
| `oiPercentage` | float | Percentage change in open interest.                            |
