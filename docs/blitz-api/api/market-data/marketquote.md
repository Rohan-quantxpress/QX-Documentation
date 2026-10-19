# Market Quote

The **Market Quote API** allows you to retrieve detailed real-time market data for one or more instruments.

Using this API, you can fetch comprehensive quote information including last traded price (LTP), last traded quantity (LTQ), volume, OHLC data, open interest, and market depth (bid/ask levels).

**Base URL:** `https://{baseurl}/md-api`

## Endpoint

| Method | URL                  |
|--------|----------------------|
| POST   | `/marketfeed/quote`  |

## Headers

| Header        | Value                 | Required |
|---------------|-----------------------|----------|
| Content-Type  | application/json      | Yes      |
| Authorization | Bearer {access_token} | Yes      |
| Accept        | `*/*`                 | Yes      |

## Request Payload & Examples

You can request market quotes by providing either a list of **Instrument IDs** or a list of **Instrument Names**.

### By Instrument ID

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/quote' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "InstrumentIds": [110010000014366]
  }'
```

**Empty Payload for User**

```json
{
  "InstrumentIds": []
}
```

**Response**

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
        {"index": 1, "price": 11.5, "quantity": 855978, "orders": 28},
        {"index": 2, "price": 11.49, "quantity": 1544481, "orders": 78},
        {"index": 3, "price": 11.48, "quantity": 2673732, "orders": 380},
        {"index": 4, "price": 11.47, "quantity": 979037, "orders": 127},
        {"index": 5, "price": 11.46, "quantity": 837048, "orders": 126}
      ],
      "askLevel": [
        {"index": 1, "price": 11.51, "quantity": 1014281, "orders": 63},
        {"index": 2, "price": 11.52, "quantity": 815256, "orders": 70},
        {"index": 3, "price": 11.53, "quantity": 1156649, "orders": 70},
        {"index": 4, "price": 11.54, "quantity": 1867230, "orders": 120},
        {"index": 5, "price": 11.55, "quantity": 1500815, "orders": 218}
      ]
    }
  },
  "status": "success"
}
```

### By Instrument Name

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/quote' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "InstrumentNames": ["NSECM:RELIANCE", "NSEFO:NIFTY28APR26FUT"]
  }'
```

**Empty Payload for User**

```json
{
  "InstrumentNames": []
}
```

**Response**

Identical structure to the response shown under [By Instrument ID](#by-instrument-id) — the quote data is keyed by instrument ID regardless of whether you queried by `InstrumentIds` or `InstrumentNames`.

---

## Parameters

| Parameter        | Type          | Required | Description                                                                  |
|------------------|---------------|----------|------------------------------------------------------------------------------|
| `InstrumentIds`  | array of Long | No*      | List of numeric instrument identifiers. (*Required if `InstrumentNames` is not provided). |
| `InstrumentNames`| array of String | No*    | List of exact instrument names. (*Required if `InstrumentIds` is not provided). |

> **Instrument Naming Convention**
>
> When fetching by Instrument Name, the string follows a specific pattern based on the segment:
>
> **Cash Market (Equity):** `[ExchangeSegment]:[Symbol]`
> - *Example:* `NSECM:RELIANCE`
>
> **Futures & Options:** `[ExchangeSegment]:[Symbol][Date][Strike][OptionType]`
> - **Date Format:** `DDMMMYY` (e.g., `28APR26` represents **28 April 2026**)
> - **Month Codes:** January (`JAN`), February (`FEB`), March (`MAR`), April (`APR`), May (`MAY`), June (`JUN`), July (`JUL`), August (`AUG`), September (`SEP`), October (`OCT`), November (`NOV`), December (`DEC`)
> - **Future Example:** `NSEFO:NIFTY28APR26FUT` (NIFTY Future, expiring 28-Apr-2026)
> - **Option Example:** `NSEFO:NIFTY28APR2625000CE` (NIFTY Option, expiring 28-Apr-2026, 25000 Strike Price, Call Option. Similarly for PUT options use 'PE')

---

## Response Field Descriptions

### Root Fields

| Field  | Type   | Description                                        |
|--------|--------|----------------------------------------------------|
| status | string | Status of the request (success or error)           |
| data   | object | Contains quote data mapped by instrument ID        |

### Quote Fields

| Field                | Type    | Description                              |
|----------------------|---------|------------------------------------------|
| instrumentId         | integer | Unique instrument identifier             |
| instrumentName       | string  | Name of the instrument                   |
| exchangeSegment      | integer | Exchange segment identifier              |
| exchangeInstrumentId | integer | Exchange-specific instrument ID          |
| timestamp            | integer | Last update timestamp (epoch seconds)    |
| ltp                  | float   | Last traded price                        |
| ltq                  | float   | Last traded quantity                     |
| ltt                  | integer | Last traded time (epoch seconds)         |
| atp                  | float   | Average traded price                     |
| vtt                  | float   | Volume traded today                      |
| tbq                  | float   | Total buy quantity                       |
| tsq                  | float   | Total sell quantity                      |
| oi                   | float   | Open interest                            |
| open                 | float   | Opening price                            |
| high                 | float   | Highest price                            |
| low                  | float   | Lowest price                             |
| close                | float   | Previous closing price                   |

---

## Market Depth Fields

Each level in `bidLevel` / `askLevel` has the following fields:

| Field    | Type    | Description                     |
|----------|---------|---------------------------------|
| index    | integer | Level number (1 = top of book)  |
| price    | float   | Bid/Ask price at the level      |
| quantity | float   | Quantity available              |
| orders   | integer | Number of orders at that level  |
