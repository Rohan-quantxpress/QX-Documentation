# **Statistics API**

The **Statistics API** allows you to retrieve performance metrics and analytics related to your trading strategies.

Using this API, you can monitor **profit/loss, trade performance, win/loss ratios, and overall strategy effectiveness**, helping you make data-driven trading decisions.

These endpoints let you **analyze trading performance and optimize strategies**.

```
BASE URL : https://{baseurl}/api_interactive/api/v1
```

| **Type** | **Endpoint**                                           | **Description**                              |
| -------- | ------------------------------------------------------ | -------------------------------------------- |
| GET      | [strategy/statistics](#get-statistics)                 | Retrieve strategy performance statistics     |
| GET      | [strategy/statistics/instance](#get-statistics-by-instance) | Retrieve statistics grouped by strategy instance |

# Get Statistics {#get-statistics}

Retrieve performance statistics for trading strategies.

This endpoint provides insights into:
- Overall performance
- Profit and loss
- Trade count and success ratio
- Strategy efficiency

!!! tip "**Analyze Performance Over Time**"
    Use statistics to compare strategy performance across different time periods. Track metrics like `UnRealized`, `Realized`, and `TradeCount` to evaluate effectiveness.

!!! note "**Data Scope**"
    Statistics are calculated based on the current session's activity. Previous session data is reflected in `PreviousNetPosition`.

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/strategy/statistics' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

## Response Field Descriptions

| Field                       | Type    | Description                                  |
| --------------------------- | ------- | -------------------------------------------- |
| `EntityId`                 | string  | Identifier for the entity                    |
| `StrategyId`               | string  | Strategy identifier                          |
| `StrategyName`             | string  | Name of the strategy                         |
| `StrategyInstanceId`      | string  | Identifier for the strategy instance         |
| `StrategyInstanceName`    | string  | Name of the strategy instance                |
| `ExchangeClientId`        | string  | Client ID for the exchange                   |
| `InstrumentId`             | integer | Internal ID of the instrument                |
| `IVName`                   | string  | IV name                                      |
| `ExchangeSegment`          | string  | Exchange segment                             |
| `InstrumentName`           | string  | Name of the instrument                       |
| `NonAckBuyQuantity`      | integer | Non-acknowledged buy quantity                |
| `NonAckSellQuantity`     | integer | Non-acknowledged sell quantity               |
| `OpenBuyQuantity`         | integer | Open buy quantity                            |
| `OpenSellQuantity`        | integer | Open sell quantity                           |
| `ShortPosition`            | integer | Short position                               |
| `LongPosition`             | integer | Long position                                |
| `NetPosition`              | integer | Net position                                 |
| `NetPositionInLot`       | integer | Net position in lots                         |
| `OrderCount`               | integer | Total order count                            |
| `TradeCount`               | integer | Total trade count                            |
| `PreviousNetPosition`     | integer | Net position from previous session           |
| `TradeBuyValue`           | number  | Total buy trade value                        |
| `TradeSellValue`          | number  | Total sell trade value                       |
| `TradeNetValue`           | number  | Net trade value                              |
| `AvgBuy`                   | number  | Average buy price                            |
| `AvgSell`                  | number  | Average sell price                           |
| `AvgNet`                   | number  | Average net price                            |
| `OPBuyPrice`              | number  | Open buy price                               |
| `OPSellPrice`             | number  | Open sell price                              |
| `Exposure`                  | number  | Exposure                                     |
| `UnRealized`                | number  | UnRealized profit/loss                       |
| `Realized`                  | number  | Realized profit/loss                         |
| `LastTradedPrice`         | number  | Last traded price                            |
| `OpenPositionTradeValue` | number  | Trade value of open position                 |
| `TurnoverValue`            | number  | Turnover value                               |
| `TurnoverValueOptions`    | number  | Turnover value for options                   |

## Response Structure

<details>
<summary>Click to view Statistics JSON Body</summary>

```json
[
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "01b6de26-f60c-4c77-9d7a-55c31b19c6fd",
        "StrategyInstanceName":  "BSEFO|SENSEX17SEP2674300CE",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  1262600000869699,
        "IVName":  "IV",
        "ExchangeSegment":  "BSEFO",
        "InstrumentName":  "SENSEX 17SEPT2026 CE 74300",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  1000,
        "LongPosition":  1000,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  2,
        "TradeCount":  11,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  388394,
        "TradeSellValue":  370900,
        "TradeNetValue":  -17494,
        "AvgBuy":  388.394,
        "AvgSell":  370.9,
        "AvgNet":  -17494,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  -17494,
        "LastTradedPrice":  371.85,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  759294,
        "TurnoverValueOptions":  759294,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "0c04f8ab-a849-4dc3-baea-536b12a385e5",
        "StrategyInstanceName":  "NSECM|RELIANCE",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  110010000002885,
        "IVName":  "IV",
        "ExchangeSegment":  "NSECM",
        "InstrumentName":  "RELIANCE",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  0,
        "LongPosition":  0,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  1,
        "TradeCount":  0,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  0,
        "TradeSellValue":  0,
        "TradeNetValue":  0,
        "AvgBuy":  0,
        "AvgSell":  0,
        "AvgNet":  0,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  0,
        "LastTradedPrice":  0,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  0,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "102ec09c-a077-429b-b269-b4afee5cdb90",
        "StrategyInstanceName":  "BSEFO|SENSEX17SEP2674200CE",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  1262600000862498,
        "IVName":  "IV",
        "ExchangeSegment":  "BSEFO",
        "InstrumentName":  "SENSEX 17SEPT2026 CE 74200",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  0,
        "LongPosition":  0,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  0,
        "TradeCount":  0,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  0,
        "TradeSellValue":  0,
        "TradeNetValue":  0,
        "AvgBuy":  0,
        "AvgSell":  0,
        "AvgNet":  0,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  0,
        "LastTradedPrice":  401.6,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  0,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "127bd6ad-9c77-4b6c-a7ae-090386abf4c6",
        "StrategyInstanceName":  "NSECM|TCS",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  110010000011536,
        "IVName":  "IV",
        "ExchangeSegment":  "NSECM",
        "InstrumentName":  "TCS",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  1,
        "LongPosition":  1,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  4,
        "TradeCount":  2,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  2209.4,
        "TradeSellValue":  2207,
        "TradeNetValue":  -2.400000000000091,
        "AvgBuy":  2209.4,
        "AvgSell":  2207,
        "AvgNet":  -2.400000000000091,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  -2.400000000000091,
        "LastTradedPrice":  2207.3,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  4416.4,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "2cbd78fb-bbb5-4a5b-aa04-bc610cbedecd",
        "StrategyInstanceName":  "NSECM|IDEA",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  110010000014366,
        "IVName":  "IV",
        "ExchangeSegment":  "NSECM",
        "InstrumentName":  "IDEA",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  1,
        "OpenSellQuantity":  0,
        "ShortPosition":  0,
        "LongPosition":  0,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  1,
        "TradeCount":  0,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  0,
        "TradeSellValue":  0,
        "TradeNetValue":  0,
        "AvgBuy":  0,
        "AvgSell":  0,
        "AvgNet":  0,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  28.74,
        "UnRealized":  0,
        "Realized":  0,
        "LastTradedPrice":  14.37,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  0,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "a44faf4c-61bc-4445-a68e-687ba4b5e7e7",
        "StrategyInstanceName":  "BSECM|TCS",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  1110010000532540,
        "IVName":  "IV",
        "ExchangeSegment":  "BSECM",
        "InstrumentName":  "TCS",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  0,
        "LongPosition":  0,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  1,
        "TradeCount":  0,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  0,
        "TradeSellValue":  0,
        "TradeNetValue":  0,
        "AvgBuy":  0,
        "AvgSell":  0,
        "AvgNet":  0,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  0,
        "LastTradedPrice":  0,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  0,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "aa4e3543-43c4-40ad-a2eb-04c0d34f6875",
        "StrategyInstanceName":  "NSEFO|NIFTY29SEP26FUT",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  262720000068407,
        "IVName":  "IV",
        "ExchangeSegment":  "NSEFO",
        "InstrumentName":  "NIFTY 29SEPT2026",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  1105,
        "LongPosition":  1105,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  6,
        "TradeCount":  17,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  25755587,
        "TradeSellValue":  25746786,
        "TradeNetValue":  -8801,
        "AvgBuy":  23308.223529411764,
        "AvgSell":  23300.258823529413,
        "AvgNet":  -8801,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  -8801,
        "LastTradedPrice":  23327.9,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  51502373,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "afb9ecde-a00e-4c68-901e-cc7521e5a3ec",
        "StrategyInstanceName":  "NSEFO|NIFTY22SEP2623200CE",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  262650000056980,
        "IVName":  "IV",
        "ExchangeSegment":  "NSEFO",
        "InstrumentName":  "NIFTY 22SEPT2026 CE 23200",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  390,
        "LongPosition":  390,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  4,
        "TradeCount":  8,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  73521.5,
        "TradeSellValue":  77987,
        "TradeNetValue":  4465.5,
        "AvgBuy":  188.51666666666668,
        "AvgSell":  199.96666666666667,
        "AvgNet":  4465.5,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  4465.5,
        "LastTradedPrice":  203,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  151508.5,
        "TurnoverValueOptions":  151508.5,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "d6ad7830-325a-4a48-a460-257fbbf6afff",
        "StrategyInstanceName":  "BSEFO|SENSEX17SEP2674300PE",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  1262600000862237,
        "IVName":  "IV",
        "ExchangeSegment":  "BSEFO",
        "InstrumentName":  "SENSEX 17SEPT2026 PE 74300",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  1000,
        "LongPosition":  1000,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  2,
        "TradeCount":  13,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  304908,
        "TradeSellValue":  280300,
        "TradeNetValue":  -24608,
        "AvgBuy":  304.908,
        "AvgSell":  280.3,
        "AvgNet":  -24608,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  -24608,
        "LastTradedPrice":  280.65,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  585208,
        "TurnoverValueOptions":  585208,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    },
    {
        "StrategyId":  "36250ee3-ad08-4e09-862f-83deee8cdef9",
        "StrategyName":  "Manual Trading",
        "StrategyInstanceId":  "e1182297-e9a0-467a-83e1-85e009f7da0b",
        "StrategyInstanceName":  "BSECM|RELIANCE",
        "ExchangeClientId":  "SAM001",
        "InstrumentId":  1110010000500325,
        "IVName":  "IV",
        "ExchangeSegment":  "BSECM",
        "InstrumentName":  "RELIANCE",
        "NonAckBuyQuantity":  0,
        "NonAckSellQuantity":  0,
        "OpenBuyQuantity":  0,
        "OpenSellQuantity":  0,
        "ShortPosition":  1,
        "LongPosition":  1,
        "NetPosition":  0,
        "NetPositionInLot":  0,
        "OrderCount":  2,
        "TradeCount":  2,
        "PreviousNetPosition":  0,
        "TradeBuyValue":  1252.2,
        "TradeSellValue":  1251.5,
        "TradeNetValue":  -0.7000000000000455,
        "AvgBuy":  1252.2,
        "AvgSell":  1251.5,
        "AvgNet":  -0.7000000000000455,
        "OPBuyPrice":  0,
        "OPSellPrice":  0,
        "Exposure":  0,
        "UnRealized":  0,
        "Realized":  -0.7000000000000455,
        "LastTradedPrice":  1251.9,
        "OpenPositionTradeValue":  0,
        "TurnoverValue":  2503.7,
        "TurnoverValueOptions":  0,
        "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf"
    }
]
```
</details>

---

# Get Statistics by Instance {#get-statistics-by-instance}

Retrieve strategy statistics for a given client or strategy instance.

This endpoint returns the statistics grouped by `ExchangeClientId`, allowing you to see how each strategy instance is performing per client.

!!! tip "**Filter Results**"
    You can narrow down the results using the optional query parameters `strategyName` and `strategyInstanceName`. When neither is provided, all statistics for your entity are returned, grouped by client.

!!! note "**Response Shape**"
    Unlike the [Get Statistics](#get-statistics) endpoint, the response `data` here is an **object** keyed by `ExchangeClientId`, where each key maps to a list of statistics records. When no matching instance is available, it returns an empty object `{}`.

## Query Parameters

| Parameter              | Type   | Required | Description                                              |
| ---------------------- | ------ | -------- | -------------------------------------------------------- |
| `strategyName`         | string | No       | Filter statistics by strategy name.                      |
| `strategyInstanceName` | string | No       | Filter statistics by strategy instance name.             |

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/strategy/statistics/instance?strategyName=Manual%20Trading&strategyInstanceName=NSEFO%7CRELIANCE24FEB261030CE' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

## Response Structure

<details>
<summary>Click to view Instance Statistics JSON Body</summary>

```json
{

}
```
</details>

The record fields are identical to those described in the [Response Field Descriptions](#response-field-descriptions) above.




