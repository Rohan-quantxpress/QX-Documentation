# Download Instruments (GZ) API

The **Download Instruments API** allows you to retrieve the *entire master list* of tradable instruments across all exchanges in one single request. Because this list is massive (containing hundreds of thousands of instruments), the response is compressed into a `.gz` (GZIP) file.

> **Tip: Fetch Once Daily** — You should only call this endpoint **once per day** (ideally early in the morning before market open).

---
**Base URL:** `https://{baseurl}/v1`

## Endpoint

| Method | URL                           |
|--------|-------------------------------|
| GET    | `/api/instruments/gz/download` |

---

## How to Download and Extract

The downloaded file is a compressed GZIP file containing a massive JSON array. If you try to open the raw response directly as text, it will look like gibberish. You must decompress it first.

Here are a few ways to download and read the file programmatically:

### cURL (Command Line)

To download the file directly to your computer using your terminal:

```bash
curl -X 'GET' \
  'https://{baseurl}/v1/api/instruments/gz/download' \
  --output instruments.json.gz
```

> After downloading, you can extract it using terminal commands like `gunzip instruments.json.gz` or using GUI extraction tools like WinRAR or 7-Zip.

### Python

This script downloads the file, unzips it in memory, and parses the JSON seamlessly without needing to save it to your hard drive.

```python
import requests
import gzip
import io
import json

url = "https://{baseurl}/v1/api/instruments/gz/download"
response = requests.get(url)

if response.status_code == 200:
    # Decompress directly from bytes
    compressed_data = io.BytesIO(response.content)
    with gzip.open(compressed_data, 'rt') as f:
        data = json.load(f)

    print(f"Successfully loaded {len(data)} instruments!")
    print("First instrument:", data[0])
else:
    print(f"Failed: {response.status_code} - {response.text}")
```

---

## Understanding the Response

Once decompressed, the file will contain a JSON array of objects. Below are sample objects representing different types of instruments.

### Equity

```json
{
  "instrumentId": 110010000002885,
  "exchangeInstrumentId": 2885,
  "instrumentName": "RELIANCE",
  "exchangeSegment": "NSECM",
  "exchange": "NSE",
  "expiryDate": "0001-01-01T00:00:00",
  "instrumentType": "Equity",
  "lotSize": 1,
  "optionType": "",
  "priceBandHigh": 1607.7,
  "priceBandLow": 1315.5,
  "strikePrice": 0,
  "symbol": "RELIANCE",
  "tickSize": 0.1,
  "freezeQty": 999999,
  "ticker": "RELIANCE INDUSTRIES LTD",
  "multiplier": 1,
  "open": 1357,
  "high": 1358.2,
  "low": 1328,
  "close": 1350.5,
  "ltp": 1350.9,
  "isin": "INE002A01018"
}
```

#### Equity Fields Explained

| Field                 | Description                                                              |
|-----------------------|--------------------------------------------------------------------------|
| `instrumentId`        | Numeric ID across the entire system. Use this for placing orders and fetching Live Market Data. |
| `exchangeInstrumentId`| Numeric ID assigned to the instrument by the exchange itself.              |
| `instrumentName`      | String name used as an alternative identifier.                             |
| `exchangeSegment`     | Identifies the market segment (Possibilities: `NSECM`, `BSECM`).          |
| `exchange`            | The exchange code (Possibilities: `NSE`, `BSE`).                          |
| `expiryDate`          | Returns `0001-01-01T00:00:00` since equities do not have an expiration date. |
| `instrumentType`      | The asset class (`Equity`).                                               |
| `lotSize`             | Always `1` for cash equities.                                             |
| `optionType`          | Empty (`""`) since equities are not options.                              |
| `priceBandHigh`       | The upper circuit limit. Orders placed above this will be rejected.        |
| `priceBandLow`        | The lower circuit limit. Orders placed below this will be rejected.        |
| `strikePrice`         | Returns `0` for equities.                                                 |
| `symbol`              | The base symbol (e.g., `RELIANCE`).                                       |
| `tickSize`            | The minimum price movement allowed for this instrument.                    |
| `freezeQty`           | Maximum quantity allowed in a single order by the exchange.                |
| `ticker`              | Full company name (e.g., `RELIANCE INDUSTRIES LTD`).                      |
| `multiplier`          | The contract multiplier (1 for cash equities).                             |
| `open`                | The opening price.                                                        |
| `high`                | The highest traded price.                                                 |
| `low`                 | The lowest traded price.                                                  |
| `close`               | The closing price.                                                        |
| `ltp`                 | The last traded price.                                                    |
| `isin`                | International Securities Identification Number (ISIN) for the instrument.  |

### Future

```json
{
  "instrumentId": 260550000059175,
  "exchangeInstrumentId": 59175,
  "instrumentName": "BANKNIFTY24FEB26FUT",
  "exchangeSegment": "NSEFO",
  "exchange": "NSE",
  "expiryDate": "2026-02-24T05:30:00+05:30",
  "instrumentType": "Futures",
  "lotSize": 30,
  "optionType": "XX",
  "priceBandHigh": 66800.8,
  "priceBandLow": 54655.2,
  "strikePrice": 0,
  "symbol": "BANKNIFTY",
  "tickSize": 0.2,
  "freezeQty": 600,
  "ticker": "BANKNIFTY 24FEB26 FUT",
  "multiplier": 1,
  "open": 0,
  "high": 0,
  "low": 0,
  "close": 0,
  "ltp": 0,
  "isin": ""
}
```

#### Future Fields Explained

| Field                 | Description                                                              |
|-----------------------|--------------------------------------------------------------------------|
| `instrumentId`        | Numeric ID across the entire system. Use this for placing orders and fetching Live Market Data. |
| `exchangeInstrumentId`| Numeric ID assigned to the instrument by the exchange itself.              |
| `instrumentName`      | String name representing the contract (e.g., `BANKNIFTY24FEB26FUT`).      |
| `exchangeSegment`     | Identifies the market segment (Possibilities: `NSEFO`, `BSEFO`).          |
| `exchange`            | The exchange code (Possibilities: `NSE`, `BSE`).                          |
| `expiryDate`          | The specific expiration date of the futures contract.                      |
| `instrumentType`      | The asset class (`Futures`).                                              |
| `lotSize`             | The contract lot size. Order quantities must be a multiple of this number. |
| `optionType`          | `"XX"` for `NSEFO` futures, `""` for `BSEFO` futures.                    |
| `priceBandHigh`       | The upper circuit limit for the contract.                                  |
| `priceBandLow`        | The lower circuit limit for the contract.                                  |
| `strikePrice`         | Returns `0` since futures don't have strike prices.                       |
| `symbol`              | The base symbol of the underlying asset (e.g., `BANKNIFTY`).              |
| `tickSize`            | The minimum price movement allowed for this instrument.                    |
| `freezeQty`           | Maximum quantity allowed in a single order by the exchange.                |
| `ticker`              | Display name of the contract (e.g., `BANKNIFTY 24FEB26 FUT`).             |
| `multiplier`          | The contract multiplier.                                                  |
| `open`                | The opening price.                                                        |
| `high`                | The highest traded price.                                                 |
| `low`                 | The lowest traded price.                                                  |
| `close`               | The closing price.                                                        |
| `ltp`                 | The last traded price.                                                    |
| `isin`                | Usually empty (`""`) for derivatives.                                     |

### Option

```json
{
  "instrumentId": 260550000059522,
  "exchangeInstrumentId": 59522,
  "instrumentName": "BANKNIFTY24FEB2636500CE",
  "exchangeSegment": "NSEFO",
  "exchange": "NSE",
  "expiryDate": "2026-02-24T05:30:00+05:30",
  "instrumentType": "Options",
  "lotSize": 30,
  "optionType": "CE",
  "priceBandHigh": 25766.9,
  "priceBandLow": 22750.7,
  "strikePrice": 36500,
  "symbol": "BANKNIFTY",
  "tickSize": 0.05,
  "freezeQty": 600,
  "ticker": "BANKNIFTY 24FEB26 36500 CE",
  "multiplier": 1,
  "open": 0,
  "high": 0,
  "low": 0,
  "close": 0,
  "ltp": 0,
  "isin": ""
}
```

#### Option Fields Explained

| Field                 | Description                                                              |
|-----------------------|--------------------------------------------------------------------------|
| `instrumentId`        | Numeric ID across the entire system. Use this for placing orders and fetching Live Market Data. |
| `exchangeInstrumentId`| Numeric ID assigned to the instrument by the exchange itself.              |
| `instrumentName`      | String name representing the contract (e.g., `BANKNIFTY24FEB2636500CE`).  |
| `exchangeSegment`     | Identifies the market segment (Possibilities: `NSEFO`, `BSEFO`).          |
| `exchange`            | The exchange code (Possibilities: `NSE`, `BSE`).                          |
| `expiryDate`          | The expiration date of the options contract.                               |
| `instrumentType`      | The asset class (`Options`).                                              |
| `lotSize`             | The contract lot size. Order quantities must be a multiple of this number. |
| `optionType`          | The option type (Possibilities: `CE` for Call, `PE` for Put).             |
| `priceBandHigh`       | The upper price limit. Orders placed above this will be rejected.          |
| `priceBandLow`        | The lower price limit. Orders placed below this will be rejected.          |
| `strikePrice`         | The strike price of the options contract (e.g., `36500`).                  |
| `symbol`              | The base symbol of the underlying asset (e.g., `BANKNIFTY`).              |
| `tickSize`            | The minimum price movement allowed for this instrument.                    |
| `freezeQty`           | Maximum quantity allowed in a single order by the exchange.                |
| `ticker`              | Display name of the contract (e.g., `BANKNIFTY 24FEB26 36500 CE`).        |
| `multiplier`          | The contract multiplier.                                                  |
| `open`                | The opening price.                                                        |
| `high`                | The highest traded price.                                                 |
| `low`                 | The lowest traded price.                                                  |
| `close`               | The closing price.                                                        |
| `ltp`                 | The last traded price.                                                    |
| `isin`                | Usually empty (`""`) for derivatives.                                     |

### Index

```json
{
  "instrumentId": 110010002000002,
  "exchangeInstrumentId": 2000002,
  "instrumentName": "NIFTY BANK",
  "exchangeSegment": "NSECM",
  "exchange": "NSE",
  "expiryDate": "0001-01-01T00:00:00",
  "instrumentType": "INDEX",
  "lotSize": 1,
  "optionType": "",
  "priceBandHigh": 0,
  "priceBandLow": 0,
  "strikePrice": 0,
  "symbol": "BANKNIFTY",
  "tickSize": 0,
  "freezeQty": 1,
  "ticker": "NIFTY BANK",
  "multiplier": 1,
  "open": 0,
  "high": 0,
  "low": 0,
  "close": 0,
  "ltp": 0,
  "isin": ""
}
```

#### Index Fields Explained

| Field                 | Description                                                              |
|-----------------------|--------------------------------------------------------------------------|
| `instrumentId`        | Numeric ID across the entire system. Use this for placing orders and fetching Live Market Data. |
| `exchangeInstrumentId`| Numeric ID assigned to the instrument by the exchange itself.              |
| `instrumentName`      | String name representing the index (e.g., `NIFTY BANK`).                  |
| `exchangeSegment`     | Identifies the market segment (Possibilities: `NSECM`, `BSECM`).          |
| `exchange`            | The exchange code (Possibilities: `NSE`, `BSE`).                          |
| `expiryDate`          | Returns `0001-01-01T00:00:00` since base indices do not have an expiration date. |
| `instrumentType`      | The asset class (`INDEX`).                                                |
| `lotSize`             | The lot size (typically `1` for the base index).                          |
| `optionType`          | Empty (`""`) since this is an index.                                      |
| `priceBandHigh`       | The upper price limit (typically `0` for an index).                       |
| `priceBandLow`        | The lower price limit (typically `0` for an index).                       |
| `strikePrice`         | Returns `0` since an index does not have a strike price.                  |
| `symbol`              | The base symbol of the underlying index (e.g., `BANKNIFTY`).              |
| `tickSize`            | The minimum price movement allowed for this instrument.                    |
| `freezeQty`           | Maximum quantity allowed in a single order by the exchange.                |
| `ticker`              | Display name of the index (e.g., `NIFTY BANK`).                           |
| `multiplier`          | The contract multiplier (1 for indices).                                  |
| `open`                | The opening price.                                                        |
| `high`                | The highest traded price.                                                 |
| `low`                 | The lowest traded price.                                                  |
| `close`               | The closing price.                                                        |
| `ltp`                 | The last traded price.                                                    |
| `isin`                | Usually empty (`""`) for indices.                                         |
