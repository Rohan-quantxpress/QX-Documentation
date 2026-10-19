# Instrument Detail

Fetch metadata for a specific instrument by ID or by `exchange:name` symbol.

Both SDKs resolve instrument details **locally** from the downloaded instrument master (gz file) — no per-call HTTP round-trip. Download the master once via [Instruments](instruments.md); the SDKs do this automatically on first use.

=== "Python"

    ```python
    from blitzsdk import MarketDataApiClient

    client = MarketDataApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")
    data = client.get_instrument_by_symbol("NSECM:RELIANCE")
    by_id = client.get_instrument_by_id(110010000002885)
    print(data, by_id)
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Common;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var mdClient = new BlitzMarketDataApiClient(config);

    var resp = await mdClient.GetInstrumentDetailsAsync("RELIANCE");   // by symbol "exchange:name"
    var byId = await mdClient.GetInstrumentDetailsAsync(110010000002885);   // or by id
    var detail = resp.Data;
    Console.WriteLine($"{detail.Symbol} lot={detail.LotSize}");
    ```

---

## Endpoint

```
GET /v1/api/instruments/{id}
```

The raw REST endpoint also accepts an `exchange:name` pair in place of `{id}`.

### Request

```bash
curl "https://{baseurl}/v1/api/instruments/110010000001134" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

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
        "ltp": 0
    }
}
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `instrumentId` | long | Unique internal instrument identifier |
| `exchange` | string | Exchange code (`NSE`) |
| `symbol` | string | Exchange symbol |
| `ticker` | string | Full company or instrument name |
| `exchangeSegment` | string | Exchange segment (`NSECM`, `NSEFO`, ...) |
| `instrumentType` | string | Type (`Equity`, `Futures`, `Options`, `Index`) |
| `instrumentName` | string | Exchange trading name |
| `exchangeInstrumentId` | int | Instrument identifier assigned by the exchange |
| `marketInstrumentId` | long | Market-wide instrument identifier |
| `series` | string | Instrument series (`EQ`, `XX`, ...) |
| `tickSize` | double | Minimum price tick |
| `isin` | string | ISIN code (equities) |
| `priceBandHigh` / `priceBandLow` | double | Price band limits |
| `multiplier` | int | Contract multiplier |
| `bvp` | number | Base price value |
| `freezeQty` | int | Freeze quantity |
| `lotSize` | int | Market lot / lot size |
| `expiryDate` | string | Expiry date (derivatives; `null` otherwise) |
| `strikePrice` | double | Strike price (options) |
| `optionType` | string | Option type (`CE`/`PE`; futures carry `XX`) |
| `assetToken` / `underlyingInstrumentId` | long | Underlying reference (derivatives) |
| `open` / `high` / `low` / `close` / `ltp` | double | Snapshot prices |

> Field values for Futures and Options variants differ only in the derivative fields (`expiryDate`, `strikePrice`, `optionType`, `assetToken`, `underlyingInstrumentId`) — see the [REST instrument detail](../../api/market-data/instrumentDetail.md) page for complete examples of each type.
