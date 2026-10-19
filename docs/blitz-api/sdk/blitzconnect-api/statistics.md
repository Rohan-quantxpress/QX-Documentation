# Statistics

Retrieve strategy-level trading statistics and performance metrics.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    stats = client.get_statistics()
    # Response is keyed by ExchangeClientId: {"Algo123": [{...}, ...]}
    for s in stats.get("Algo123", []):
        print(s["StrategyName"], s["NetPosition"], s["UnRealized"], s["Realized"])

    # Per-instance statistics for one strategy instance
    instance_stats = client.get_statistics_by_instance(
        strategy_name="Manual Trading",
        strategy_instance_name="NSECM|IDEA",
    )
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var stats = await client.GetStatisticsAsync();
    foreach (var s in stats)
    {
        Console.WriteLine($"{s.StrategyName} NetPos:{s.NetPosition} UnReal:{s.UnRealized} Real:{s.Realized}");
    }

    // Per-instance statistics for one strategy instance
    var instanceStats = await client.GetStatisticsByInstanceAsync(
        strategyName: "Manual Trading",
        strategyInstanceName: "NSECM|IDEA",
    );
    ```

---

## Endpoint

```
GET /strategy/statistics
```

### Request

```bash
curl "https://{baseurl}/api_interactive/api/v1/strategy/statistics" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

The response is a top-level array of per-strategy-instance statistics:

```json
[
  {
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyName": "Manual Trading",
    "StrategyInstanceId": "23335e0b-4386-4c21-b76b-cbfd3891fc95",
    "StrategyInstanceName": "NSECM|IDEA",
    "ExchangeClientId": "Algo123",
    "InstrumentId": 110010000014366,
    "IVName": "IV",
    "ExchangeSegment": "NSECM",
    "InstrumentName": "IDEA",
    "NonAckBuyQuantity": 0,
    "NonAckSellQuantity": 0,
    "OpenBuyQuantity": 0,
    "OpenSellQuantity": 0,
    "ShortPosition": 0,
    "LongPosition": 0,
    "NetPosition": 0,
    "NetPositionInLot": 0,
    "OrderCount": 5,
    "TradeCount": 0,
    "PreviousNetPosition": 0,
    "TradeBuyValue": 0,
    "TradeSellValue": 0,
    "TradeNetValue": 0,
    "AvgBuy": 0,
    "AvgSell": 0,
    "AvgNet": 0,
    "OPBuyPrice": 0,
    "OPSellPrice": 0,
    "Exposure": 0,
    "UnRealized": 0,
    "Realized": 0,
    "LastTradedPrice": 0,
    "OpenPositionTradeValue": 0,
    "TurnoverValue": 0,
    "TurnoverValueOptions": 0,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed"
  },
  {
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyName": "Manual Trading",
    "StrategyInstanceId": "3f1098c0-a7c5-434c-bb2d-3d7bc57082a8",
    "StrategyInstanceName": "NSECM|RELIANCE",
    "ExchangeClientId": "Algo123",
    "InstrumentId": 110010000002885,
    "IVName": "IV",
    "ExchangeSegment": "NSECM",
    "InstrumentName": "RELIANCE",
    "NonAckBuyQuantity": 0,
    "NonAckSellQuantity": 0,
    "OpenBuyQuantity": 0,
    "OpenSellQuantity": 0,
    "ShortPosition": 0,
    "LongPosition": 0,
    "NetPosition": 0,
    "NetPositionInLot": 0,
    "OrderCount": 3,
    "TradeCount": 0,
    "PreviousNetPosition": 0,
    "TradeBuyValue": 0,
    "TradeSellValue": 0,
    "TradeNetValue": 0,
    "AvgBuy": 0,
    "AvgSell": 0,
    "AvgNet": 0,
    "OPBuyPrice": 0,
    "OPSellPrice": 0,
    "Exposure": 0,
    "UnRealized": 0,
    "Realized": 0,
    "LastTradedPrice": 0,
    "OpenPositionTradeValue": 0,
    "TurnoverValue": 0,
    "TurnoverValueOptions": 0,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed"
  },
  {
    "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
    "StrategyName": "Manual Trading",
    "StrategyInstanceId": "e3cdd839-6ffd-45c5-a290-7924ba631760",
    "StrategyInstanceName": "NSECM|TCS",
    "ExchangeClientId": "Algo123",
    "InstrumentId": 110010000011536,
    "IVName": "IV",
    "ExchangeSegment": "NSECM",
    "InstrumentName": "TCS",
    "NonAckBuyQuantity": 0,
    "NonAckSellQuantity": 0,
    "OpenBuyQuantity": 0,
    "OpenSellQuantity": 0,
    "ShortPosition": 0,
    "LongPosition": 0,
    "NetPosition": 0,
    "NetPositionInLot": 0,
    "OrderCount": 2,
    "TradeCount": 0,
    "PreviousNetPosition": 0,
    "TradeBuyValue": 0,
    "TradeSellValue": 0,
    "TradeNetValue": 0,
    "AvgBuy": 0,
    "AvgSell": 0,
    "AvgNet": 0,
    "OPBuyPrice": 0,
    "OPSellPrice": 0,
    "Exposure": 0,
    "UnRealized": 0,
    "Realized": 0,
    "LastTradedPrice": 0,
    "OpenPositionTradeValue": 0,
    "TurnoverValue": 0,
    "TurnoverValueOptions": 0,
    "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed"
  }
]
```

### Per-instance statistics

```
GET /strategy/statistics/instance?strategyName=&strategyInstanceName=
```

Returns an **object keyed by `ExchangeClientId`**, each holding that client's instance statistics list — or `{}` when no matching instance exists:

```json
{
  "Algo123": [
    {
      "StrategyId": "36250ee3-ad08-4e09-862f-83deee8cdef9",
      "StrategyName": "Manual Trading",
      "StrategyInstanceId": "23335e0b-4386-4c21-b76b-cbfd3891fc95",
      "StrategyInstanceName": "NSECM|IDEA",
      "ExchangeClientId": "Algo123",
      "InstrumentId": 110010000014366,
      "IVName": "IV",
      "ExchangeSegment": "NSECM",
      "InstrumentName": "IDEA",
      "NonAckBuyQuantity": 0,
      "NonAckSellQuantity": 0,
      "OpenBuyQuantity": 0,
      "OpenSellQuantity": 0,
      "ShortPosition": 0,
      "LongPosition": 0,
      "NetPosition": 0,
      "NetPositionInLot": 0,
      "OrderCount": 5,
      "TradeCount": 0,
      "PreviousNetPosition": 0,
      "TradeBuyValue": 0,
      "TradeSellValue": 0,
      "TradeNetValue": 0,
      "AvgBuy": 0,
      "AvgSell": 0,
      "AvgNet": 0,
      "OPBuyPrice": 0,
      "OPSellPrice": 0,
      "Exposure": 0,
      "UnRealized": 0,
      "Realized": 0,
      "LastTradedPrice": 0,
      "OpenPositionTradeValue": 0,
      "TurnoverValue": 0,
      "TurnoverValueOptions": 0,
      "EntityId": "85abbeeb-2e7f-44a5-b668-84720d29ceed"
    }
  ]
}
```

### Statistics fields (per-strategy instance)

| Field | Type | Description |
|-------|------|-------------|
| `EntityId` | string | Entity / user identifier |
| `StrategyId` | string | Strategy identifier |
| `StrategyName` | string | Strategy name |
| `StrategyInstanceId` | string | Strategy instance identifier |
| `StrategyInstanceName` | string | Strategy instance name |
| `ExchangeClientId` | string | Exchange client ID |
| `InstrumentId` | long | Internal instrument ID |
| `IVName` | string | IV object name |
| `ExchangeSegment` | string | Exchange segment |
| `InstrumentName` | string | Instrument name |
| `NonAckBuyQuantity` | int | Unacknowledged buy quantity |
| `NonAckSellQuantity` | int | Unacknowledged sell quantity |
| `OpenBuyQuantity` | int | Open buy quantity |
| `OpenSellQuantity` | int | Open sell quantity |
| `ShortPosition` | int | Short position |
| `LongPosition` | int | Long position |
| `NetPosition` | int | Net position |
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
| `OPBuyPrice` | double | Open buy price |
| `OPSellPrice` | double | Open sell price |
| `Exposure` | double | Current exposure |
| `UnRealized` | double | Unrealized P&L |
| `Realized` | double | Realized P&L |
| `LastTradedPrice` | double | Last traded price |
| `OpenPositionTradeValue` | double | Open position trade value |
| `TurnoverValue` | double | Total turnover |
| `TurnoverValueOptions` | double | Options turnover |
