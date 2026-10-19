# Instrument Detail

The **Instrument Detail API** allows you to retrieve detailed information about a trading instrument using its instrument ID or instrument name.

Using this API, you can fetch instrument metadata such as symbol details, exchange information, price bands, tick size, lot size, and other trading parameters.

---
**Base URL:** `https://{baseurl}/v1`

## Endpoint

| Method | URL                             |
|--------|----------------------------------|
| GET    | `/api/instruments/{id/name}`     |

## Path Parameters

| Parameter  | Type          | Description                                                                                                 |
|------------|---------------|-------------------------------------------------------------------------------------------------------------|
| `id/name`  | `string/long` | You can retrieve details using either the numeric **Instrument ID** (e.g., `110010000014366`) or by **Name** (e.g., `NSECM:RELIANCE`). |

> **Instrument Naming Convention**
> When fetching by Instrument Name, the string follows a specific pattern based on the segment:
>
> **Cash Market (Equity):** `[ExchangeSegment]:[Symbol]`
> - *Example:* `NSECM:RELIANCE`
>
> **Futures & Options:** `[ExchangeSegment]:[Symbol][Date][Strike][OptionType]`
> - **Date Format:** `DDMMMYY` (e.g., `28APR26` represents **28 April 2026**)
> - **Month Codes:** `JAN` `FEB` `MAR` `APR` `MAY` `JUN` `JUL` `AUG` `SEP` `OCT` `NOV` `DEC`
> - **Future Example:** `NSEFO:NIFTY28APR26FUT` — NIFTY Future expiring 28 Apr 2026
> - **Monthly Option Example:** `NSEFO:NIFTY28APR2625000CE` — NIFTY Call Option, 28 Apr 2026 (monthly expiry), strike 25000. Use `PE` for Put Options.
> - **Weekly Option Example:** `NSEFO:NIFTY25MAR261030CE` — NIFTY Call Option, 25 Mar 2026 (**weekly expiry**), strike 1030. Use `PE` for Put Options.
>
> > The expiry date in the name indicates whether the contract belongs to the **weekly** or **monthly** series. Weekly options expire on a specific weekday every week; monthly options expire on the last trading day of the month.

## Headers

| Header        | Value                | Required |
|---------------|----------------------|----------|
| Content-Type  | application/json     | Yes      |
| Authorization | Bearer {access_token}| Yes      |
| Accept        | `*/*`                | Yes      |

## Examples

### By Instrument ID

#### Equity

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/110010000001134' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

**Response**

```json
{
    "status": "success",
    "message": "request processed successfully",
    "data": {
        "instrumentId": 110010000001134,
        "exchange": "NSE",
        "symbol": "APOLLO",
        "ticker": "APOLLO MICRO SYSTEMS LTD",
        "exchangeSegment": "NSECM",
        "instrumentType": "Equity",
        "instrumentName": "APOLLO",
        "exchangeInstrumentId": 1134,
        "marketInstrumentId": 110010000001134,
        "series": "EQ",
        "tickSize": 0.05,
        "isin": "INE713T01028",
        "priceBandHigh": 356.55,
        "priceBandLow": 237.75,
        "multiplier": 1,
        "bvp": 1,
        "priceType": "1.0",
        "priceNumerator": 1,
        "priceDenominator": 1,
        "freezeQty": 999999,
        "lotSize": 1,
        "expiryDate": null,
        "strikePrice": null,
        "optionType": null,
        "assetToken": null,
        "underlyingInstrumentId": null,
        "open": 0,
        "high": 0,
        "low": 0,
        "close": 0,
        "ltp": 0,
        "reserve1": "",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
}
```

#### Futures

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/261810000062367' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

**Response**

```json
{
    "status": "success",
    "message": "request processed successfully",
    "data": {
        "instrumentId": 261810000062367,
        "exchange": "NSE",
        "symbol": "ASIANPAINT",
        "ticker": "ASIANPAINT 30JUN26 FUT",
        "exchangeSegment": "NSEFO",
        "instrumentType": "Futures",
        "instrumentName": "ASIANPAINT30JUN26FUT",
        "exchangeInstrumentId": 62367,
        "marketInstrumentId": 210010000062367,
        "series": "XX",
        "tickSize": 0.1,
        "isin": "",
        "priceBandHigh": 2698.7,
        "priceBandLow": 2208.1,
        "multiplier": 1,
        "bvp": 1,
        "priceType": "1.0",
        "priceNumerator": 1,
        "priceDenominator": 1,
        "freezeQty": 10000,
        "lotSize": 250,
        "expiryDate": "2026-06-30T00:00:00",
        "strikePrice": 0,
        "optionType": "XX",
        "assetToken": 236,
        "underlyingInstrumentId": null,
        "open": 0,
        "high": 0,
        "low": 0,
        "close": 0,
        "ltp": 0,
        "reserve1": "",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
}
```

#### Options

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/261810000069489' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

**Response**

```json
{
    "status": "success",
    "message": "request processed successfully",
    "data": {
        "instrumentId": 261810000069489,
        "exchange": "NSE",
        "symbol": "ASIANPAINT",
        "ticker": "ASIANPAINT 30JUN26 1820 CE",
        "exchangeSegment": "NSEFO",
        "instrumentType": "Options",
        "instrumentName": "ASIANPAINT30JUN261820CE",
        "exchangeInstrumentId": 69489,
        "marketInstrumentId": 210010000069489,
        "series": "XX",
        "tickSize": 0.05,
        "isin": "",
        "priceBandHigh": 752.2,
        "priceBandLow": 531.2,
        "multiplier": 1,
        "bvp": 1,
        "priceType": "1.0",
        "priceNumerator": 1,
        "priceDenominator": 1,
        "freezeQty": 10000,
        "lotSize": 250,
        "expiryDate": "2026-06-30T00:00:00",
        "strikePrice": 1820,
        "optionType": "CE",
        "assetToken": 236,
        "underlyingInstrumentId": null,
        "open": 0,
        "high": 0,
        "low": 0,
        "close": 0,
        "ltp": 0,
        "reserve1": "3.5",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
}
```

### By Instrument Name

#### Equity

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/NSECM:RELIANCE' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

**Response**

```json
{
    "status": "success",
    "message": "request processed successfully",
    "data": {
        "instrumentId": 110010000002885,
        "exchange": "NSE",
        "symbol": "RELIANCE",
        "ticker": "RELIANCE INDUSTRIES LTD",
        "exchangeSegment": "NSECM",
        "instrumentType": "Equity",
        "instrumentName": "RELIANCE",
        "exchangeInstrumentId": 2885,
        "marketInstrumentId": 110010000002885,
        "series": "EQ",
        "tickSize": 0.1,
        "isin": "INE002A01018",
        "priceBandHigh": 1607.7,
        "priceBandLow": 1315.5,
        "multiplier": 1,
        "bvp": 1,
        "priceType": "1.0",
        "priceNumerator": 1,
        "priceDenominator": 1,
        "freezeQty": 999999,
        "lotSize": 1,
        "expiryDate": null,
        "strikePrice": null,
        "optionType": null,
        "assetToken": null,
        "underlyingInstrumentId": null,
        "open": 1357,
        "high": 1358.2,
        "low": 1328,
        "close": 1350.5,
        "ltp": 1350.9,
        "reserve1": "",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
}
```

#### Options

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/NSEFO:RELIANCE24FEB26680CE' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

**Response**

```json
{
    "status": "success",
    "message": "request processed successfully",
    "data": {
        "instrumentId": 260550000062583,
        "exchange": "NSE",
        "symbol": "RELIANCE",
        "ticker": "RELIANCE 24FEB26 680 CE",
        "exchangeSegment": "NSEFO",
        "instrumentType": "Options",
        "instrumentName": "RELIANCE24FEB26680CE",
        "exchangeInstrumentId": 62583,
        "marketInstrumentId": 210010000062583,
        "series": "XX",
        "tickSize": 0.05,
        "isin": "",
        "priceBandHigh": 840.25,
        "priceBandLow": 726.35,
        "multiplier": 1,
        "bvp": 1,
        "priceType": "1.0",
        "priceNumerator": 1,
        "priceDenominator": 1,
        "freezeQty": 15000,
        "lotSize": 500,
        "expiryDate": "2026-02-24T00:00:00",
        "strikePrice": 680,
        "optionType": "CE",
        "assetToken": 2885,
        "underlyingInstrumentId": null,
        "open": 0,
        "high": 0,
        "low": 0,
        "close": 0,
        "ltp": 0,
        "reserve1": "5.25",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
}
```

#### Futures

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/NSEFO:NIFTY28APR26FUT' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

**Response**

```json
{
    "status": "success",
    "message": "request processed successfully",
    "data": {
        "instrumentId": 110010000003457,
        "exchange": "NSEFO",
        "symbol": "NIFTY",
        "ticker": "NIFTY Future",
        "exchangeSegment": "NSEFO",
        "instrumentType": "Futures",
        "instrumentName": "NIFTY28APR26FUT",
        "exchangeInstrumentId": 3457,
        "marketInstrumentId": 110010000003457,
        "series": "XX",
        "tickSize": 0.05,
        "isin": "INE002A01018",
        "priceBandHigh": 25000.0,
        "priceBandLow": 23000.0,
        "multiplier": 1,
        "bvp": 1,
        "priceType": "1.0",
        "priceNumerator": 1,
        "priceDenominator": 1,
        "freezeQty": 999999,
        "lotSize": 50,
        "expiryDate": "2026-04-28",
        "strikePrice": null,
        "optionType": null,
        "assetToken": "1234",
        "underlyingInstrumentId": 110010000001234,
        "open": 24000.0,
        "high": 24100.0,
        "low": 23900.0,
        "close": 24050.0,
        "ltp": 24080.0,
        "reserve1": "",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
}
```

---

## Response Field Descriptions

### Root Response Fields

| Field    | Type   | Description                                      |
|----------|--------|--------------------------------------------------|
| status   | string | Status of the request (`success` or `error`)      |
| message  | string | Response message                                  |
| data     | object | Contains instrument details                       |

### Instrument Fields Explained

#### Equity

| Field                  | Type    | Description                                             |
|------------------------|---------|---------------------------------------------------------|
| instrumentId           | integer | Unique internal identifier of the instrument            |
| exchange               | string  | Exchange where instrument is traded (e.g., `NSE`, `BSE`)|
| symbol                 | string  | Trading symbol of the instrument                        |
| ticker                 | string  | Full company or instrument name                         |
| exchangeSegment        | string  | Exchange segment                                        |
| instrumentType         | string  | Type of instrument                                      |
| instrumentName         | string  | Name of the instrument                                  |
| exchangeInstrumentId   | integer | Instrument identifier assigned by exchange              |
| marketInstrumentId     | integer | Market-wide instrument identifier                       |
| series                 | string  | Instrument series                                       |
| tickSize               | float   | Minimum price movement allowed                          |
| isin                   | string  | International Securities Identification Number          |
| priceBandHigh          | float   | Upper price band limit                                  |
| priceBandLow           | float   | Lower price band limit                                  |
| multiplier             | float   | Contract multiplier                                     |
| bvp                    | integer | Book value per share / buy value point                  |
| priceType              | string  | Price type configuration                                |
| priceNumerator         | integer | Price numerator                                         |
| priceDenominator       | integer | Price denominator                                       |
| freezeQty              | integer | Freeze quantity                                         |
| lotSize                | integer | Size of the lot                                         |
| expiryDate             | null    | Not applicable for equities                             |
| strikePrice            | null    | Not applicable for equities                             |
| optionType             | null    | Not applicable for equities                             |
| assetToken             | null    | Not applicable for equities                             |
| underlyingInstrumentId | null    | Not applicable for equities                             |
| open                   | float   | Opening price                                           |
| high                   | float   | Highest traded price                                    |
| low                    | float   | Lowest traded price                                     |
| close                  | float   | Closing price                                           |
| ltp                    | float   | Last traded price                                       |
| reserve1–reserve8      | string  | Reserved fields                                         |

#### Futures

| Field                  | Type          | Description                                             |
|------------------------|---------------|---------------------------------------------------------|
| instrumentId           | integer       | Unique internal identifier of the instrument            |
| exchange               | string        | Exchange where instrument is traded (e.g., `NSE`, `BSE`)|
| symbol                 | string        | Trading symbol of the instrument                        |
| ticker                 | string        | Full company or instrument name                         |
| exchangeSegment        | string        | Exchange segment (e.g., `NSEFO`)                        |
| instrumentType         | string        | Type of instrument (`Futures`)                          |
| instrumentName         | string        | Name of the instrument                                  |
| exchangeInstrumentId   | integer       | Instrument identifier assigned by exchange              |
| marketInstrumentId     | integer       | Market-wide instrument identifier                       |
| series                 | string        | Instrument series (e.g., `XX`)                          |
| tickSize               | float         | Minimum price movement allowed                          |
| isin                   | string        | International Securities Identification Number          |
| priceBandHigh          | float         | Upper price band limit                                  |
| priceBandLow           | float         | Lower price band limit                                  |
| multiplier             | float         | Contract multiplier                                     |
| priceType              | string        | Price type configuration                                |
| priceNumerator         | integer       | Price numerator                                         |
| priceDenominator       | integer       | Price denominator                                       |
| freezeQty              | integer       | Freeze quantity                                         |
| lotSize                | integer       | Size of the lot                                         |
| expiryDate             | string        | Expiry date of the future contract                      |
| strikePrice            | float         | `0` for futures                                         |
| optionType             | string        | `"XX"` for futures                                      |
| assetToken             | string/integer| Asset token reference                                   |
| underlyingInstrumentId | integer/null  | Underlying instrument ID                                |
| open                   | float         | Opening price                                           |
| high                   | float         | Highest traded price                                    |
| low                    | float         | Lowest traded price                                     |
| close                  | float         | Closing price                                           |
| ltp                    | float         | Last traded price                                       |
| reserve1–reserve8      | string        | Reserved fields                                         |

#### Options

| Field                  | Type          | Description                                             |
|------------------------|---------------|---------------------------------------------------------|
| instrumentId           | integer       | Unique internal identifier of the instrument            |
| exchange               | string        | Exchange where instrument is traded (e.g., `NSE`, `BSE`)|
| symbol                 | string        | Trading symbol of the instrument                        |
| ticker                 | string        | Full company or instrument name                         |
| exchangeSegment        | string        | Exchange segment (e.g., `NSEFO`)                        |
| instrumentType         | string        | Type of instrument (`Options`)                          |
| instrumentName         | string        | Name of the instrument                                  |
| exchangeInstrumentId   | integer       | Instrument identifier assigned by exchange              |
| marketInstrumentId     | integer       | Market-wide instrument identifier                       |
| series                 | string        | Instrument series (e.g., `XX`)                          |
| tickSize               | float         | Minimum price movement allowed                          |
| isin                   | string        | International Securities Identification Number          |
| priceBandHigh          | float         | Upper price band limit                                  |
| priceBandLow           | float         | Lower price band limit                                  |
| multiplier             | integer       | Contract multiplier                                     |
| priceType              | string        | Price type configuration                                |
| priceNumerator         | integer       | Price numerator                                         |
| priceDenominator       | integer       | Price denominator                                       |
| freezeQty              | integer       | Freeze quantity                                         |
| lotSize                | integer       | Size of the lot                                         |
| expiryDate             | string        | Expiry date of the option contract                      |
| strikePrice            | float         | Strike price for the option                             |
| optionType             | string        | Option type (`CE` for Call, `PE` for Put)               |
| assetToken             | string/integer| Asset token reference                                   |
| underlyingInstrumentId | integer/null  | Underlying instrument ID                                |
| open                   | float         | Opening price                                           |
| high                   | float         | Highest traded price                                    |
| low                    | float         | Lowest traded price                                     |
| close                  | float         | Closing price                                           |
| ltp                    | float         | Last traded price                                       |
| reserve1–reserve8      | string        | Reserved fields                                         |

> **Notes**
> - Use instrument ID to fetch trading metadata.
> - Derivative-specific fields like `expiryDate`, `strikePrice`, and `optionType` may be `null` for equities.
> - Price band values define allowed trading range.
> - Reserved fields are placeholders for future system extensions.
