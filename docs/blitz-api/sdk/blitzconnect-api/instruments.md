# Instruments

The instruments API provides a consolidated, import-ready gzipped JSON dump of all tradable instruments.

## SDK

=== "Python"

    ```python
    from blitzsdk.marketdata import InstrumentManager

    InstrumentManager.load("https://{baseurl}/v1/api/instruments/gz/download")
    instr_id = InstrumentManager.resolve(symbol="NSECM|NIFTY")
    print(f"NIFTY instrument ID: {instr_id}")
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var manager = new BlitzInstrumentManager();
    await manager.LoadInstrumentsAsync("https://{baseurl}/v1/api/instruments/gz/download");
    var id = manager.GetInstrumentId("NIFTY");
    Console.WriteLine($"NIFTY instrument ID: {id}");
    ```
    
    The client also exposes the raw fetch directly:
    
    ```csharp
    var instruments = await mdClient.GetInstrumentsAsync();
    ```

---

## Download instrument master

```bash
curl -O "https://{baseurl}/v1/api/instruments/gz/download" \
  -H "Authorization: Bearer YOUR_TOKEN"

gunzip download
```

### Response

```json
[
    {
        "instrumentId": 110010000014366,
        "exchange": "NSE",
        "symbol": "IDEA",
        "ticker": "VODAFONE IDEA LIMITED",
        "exchangeSegment": "NSECM",
        "instrumentType": "Equity",
        "instrumentName": "IDEA",
        "exchangeInstrumentId": 14366,
        "marketInstrumentId": 110010000014366,
        "series": "EQ",
        "tickSize": 0.01,
        "isin": "INE669E01016",
        "priceBandHigh": 12.73,
        "priceBandLow": 10.43,
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
        "open": 14.11,
        "high": 14.14,
        "low": 13.66,
        "close": 13.71,
        "ltp": 13.71,
        "reserve1": "",
        "reserve2": "",
        "reserve3": "",
        "reserve4": "",
        "reserve5": "",
        "reserve6": "",
        "reserve7": "",
        "reserve8": ""
    }
]
```

The master dump is an array of such instrument detail objects. A single-object variant is also returned by `GET /v1/api/instruments/{id or name}`.

### Columns

| Field | Type | Description |
|-------|------|-------------|
| `instrumentId` | long | Unique instrument identifier |
| `exchange` | string | Exchange code |
| `symbol` | string | Trading symbol |
| `ticker` | string | Full company or instrument name |
| `exchangeSegment` | string | Exchange segment |
| `instrumentType` | string | Type (`INDEX`, `Equity`, `Futures`, `Options`) |
| `instrumentName` | string | Human-readable name |
| `exchangeInstrumentId` | int | Instrument identifier assigned by exchange |
| `marketInstrumentId` | long | Market instrument identifier |
| `series` | string | Series code (e.g. `EQ`) |
| `tickSize` | double | Minimum price tick |
| `isin` | string | ISIN code |
| `priceBandHigh` | double | Upper price band limit |
| `priceBandLow` | double | Lower price band limit |
| `multiplier` | int | Contract multiplier |
| `bvp` | int | Board value per point |
| `priceType` | string | Price type |
| `priceNumerator` | int | Price numerator |
| `priceDenominator` | int | Price denominator |
| `freezeQty` | int | Freeze quantity |
| `lotSize` | int | Lot size |
| `expiryDate` | string | Expiry date (`YYYY-MM-DD`) |
| `strikePrice` | double | Strike price (for options) |
| `optionType` | string | Option type (`CE`, `PE`) |
| `assetToken` | string | Asset token |
| `underlyingInstrumentId` | long | Underlying instrument ID |
| `open` | double | Opening price |
| `high` | double | Highest traded price |
| `low` | double | Lowest traded price |
| `close` | double | Closing price |
| `ltp` | double | Last traded price |
| `reserve1`..`reserve8` | string | Reserved fields |

### Symbol format reference

| Product | Format | Example |
|---------|--------|---------|
| INDEX | `SYMBOL` | `NIFTY` |
| EQUITY | `SYMBOL` | `TCS` |
| FUTURES | `SYMBOL+EXPIRY+TYPE` | `NIFTY26JUNFUT` |
| OPTIONS (CE) | `SYMBOL+EXPIRY+STRIKE+CE` | `NIFTY26JUN18000CE` |
| OPTIONS (PE) | `SYMBOL+EXPIRY+STRIKE+PE` | `NIFTY26JUN18000PE` |
