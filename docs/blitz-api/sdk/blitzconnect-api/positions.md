# Positions

Retrieve consolidated position details across all executed trades and active holdings.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    positions = client.get_positions()
    # Response is keyed by ClientId: {"Algo123": [{...}, ...]}
    for pos in positions.get("Algo123", []):
        print(pos["InstrumentId"], pos["NetPosition"], pos["UnRealized"])
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var positions = await client.GetPositionsAsync();
    foreach (var pos in positions)
    {
        Console.WriteLine($"{pos.InstrumentId} Qty:{pos.NetPosition} UnReal:{pos.UnRealized}");
    }
    ```

---

## Endpoint

```
GET /positions
```

### Request

```bash
curl "https://{baseurl}/api_interactive/api/v1/positions" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

The response is an object keyed by `ClientId`, with the position array for that client:

```json
{
  "Algo123": [
    {
      "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
      "ClientId": "Algo123",
      "RMSEntityCode": "",
      "InstrumentId": 110010000002885,
      "ExchangeSegment": "NSECM",
      "InstrumentName": "RELIANCE",
      "NonAckBuyQuantity": 0,
      "NonAckSellQuantity": 0,
      "OpenBuyQuantity": 1,
      "OpenSellQuantity": 0,
      "ShortPosition": 0,
      "LongPosition": 2,
      "NetPosition": 2,
      "NetPositionInLot": 2,
      "OrderCount": 13,
      "TradeCount": 2,
      "PreviousNetPosition": 0,
      "TradeBuyValue": 2717.2,
      "TradeSellValue": 0,
      "TradeNetValue": -2717.2,
      "AvgBuy": 1358.6,
      "AvgSell": 0,
      "AvgNet": -1358.6,
      "Exposure": 0,
      "UnRealized": -2717.2,
      "Realized": 0,
      "LastTradedPrice": 0,
      "OpenPositionTradeValue": 2717.2,
      "TurnoverValue": 2717.2,
      "TurnoverValueOptions": 0
    },
    {
      "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
      "ClientId": "Algo123",
      "RMSEntityCode": "",
      "InstrumentId": 110010000011536,
      "ExchangeSegment": "NSECM",
      "InstrumentName": "TCS",
      "NonAckBuyQuantity": 0,
      "NonAckSellQuantity": 0,
      "OpenBuyQuantity": 3,
      "OpenSellQuantity": 0,
      "ShortPosition": 0,
      "LongPosition": 3,
      "NetPosition": 3,
      "NetPositionInLot": 3,
      "OrderCount": 10,
      "TradeCount": 3,
      "PreviousNetPosition": 0,
      "TradeBuyValue": 6913.8,
      "TradeSellValue": 0,
      "TradeNetValue": -6913.8,
      "AvgBuy": 2304.6,
      "AvgSell": 0,
      "AvgNet": -2304.6,
      "Exposure": 0,
      "UnRealized": -6913.8,
      "Realized": 0,
      "LastTradedPrice": 0,
      "OpenPositionTradeValue": 6913.8,
      "TurnoverValue": 6913.8,
      "TurnoverValueOptions": 0
    },
    {
      "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
      "ClientId": "Algo123",
      "RMSEntityCode": "",
      "InstrumentId": 110010000014366,
      "ExchangeSegment": "NSECM",
      "InstrumentName": "IDEA",
      "NonAckBuyQuantity": 0,
      "NonAckSellQuantity": 0,
      "OpenBuyQuantity": 558,
      "OpenSellQuantity": 0,
      "ShortPosition": 0,
      "LongPosition": 1501,
      "NetPosition": 1501,
      "NetPositionInLot": 1501,
      "OrderCount": 131,
      "TradeCount": 29,
      "PreviousNetPosition": 0,
      "TradeBuyValue": 17261.52,
      "TradeSellValue": 0,
      "TradeNetValue": -17261.52,
      "AvgBuy": 11.500013324450366,
      "AvgSell": 0,
      "AvgNet": -11.500013324450366,
      "Exposure": 0,
      "UnRealized": -17261.52,
      "Realized": 0,
      "LastTradedPrice": 0,
      "OpenPositionTradeValue": 17261.52,
      "TurnoverValue": 17261.52,
      "TurnoverValueOptions": 0
    },
    {
      "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
      "ClientId": "Algo123",
      "RMSEntityCode": "",
      "InstrumentId": 260550000059489,
      "ExchangeSegment": "NSEFO",
      "InstrumentName": "TCS 24FEB2026",
      "NonAckBuyQuantity": 0,
      "NonAckSellQuantity": 0,
      "OpenBuyQuantity": 0,
      "OpenSellQuantity": 0,
      "ShortPosition": 0,
      "LongPosition": 0,
      "NetPosition": 0,
      "NetPositionInLot": 0,
      "OrderCount": 1,
      "TradeCount": 0,
      "PreviousNetPosition": 0,
      "TradeBuyValue": 0,
      "TradeSellValue": 0,
      "TradeNetValue": 0,
      "AvgBuy": 0,
      "AvgSell": 0,
      "AvgNet": 0,
      "Exposure": 0,
      "UnRealized": 0,
      "Realized": 0,
      "LastTradedPrice": 3002.6,
      "OpenPositionTradeValue": 0,
      "TurnoverValue": 0,
      "TurnoverValueOptions": 0
    }
  ]
}
```

### Position fields

| Field | Type | Description |
|-------|------|-------------|
| `EntityId` | string | Entity / user identifier |
| `ClientId` | string | Client identifier |
| `RMSEntityCode` | string | RMS entity code |
| `InstrumentId` | long | Internal instrument ID |
| `ExchangeSegment` | string | Exchange segment |
| `InstrumentName` | string | Instrument name |
| `NonAckBuyQuantity` | int | Unacknowledged buy quantity |
| `NonAckSellQuantity` | int | Unacknowledged sell quantity |
| `OpenBuyQuantity` | int | Open buy quantity |
| `OpenSellQuantity` | int | Open sell quantity |
| `ShortPosition` | int | Short position |
| `LongPosition` | int | Long position |
| `NetPosition` | int | Net position quantity |
| `NetPositionInLot` | int | Net position in lots |
| `OrderCount` | int | Total order count |
| `TradeCount` | int | Total trade count |
| `PreviousNetPosition` | int | Previous session net position |
| `TradeBuyValue` | double | Total buy trade value |
| `TradeSellValue` | double | Total sell trade value |
| `TradeNetValue` | double | Net trade value |
| `AvgBuy` | double | Average buy price |
| `AvgSell` | double | Average sell price |
| `AvgNet` | double | Average net price |
| `Exposure` | double | Current exposure |
| `UnRealized` | double | Unrealized P&L |
| `Realized` | double | Realized P&L |
| `LastTradedPrice` | double | Last traded price |
| `OpenPositionTradeValue` | double | Open position trade value |
| `TurnoverValue` | double | Total turnover |
| `TurnoverValueOptions` | double | Options turnover |
