# **Positions API**

The **Positions API** allows you to retrieve all current positions held by the client across different instruments.

Using this API, you can track your **net positions, quantities, average prices, and profit/loss**, enabling better portfolio and risk management.

---

# Base URL

```
https://{baseurl}/api_interactive/api/v1
```

---

# Headers

| Header        | Value                 | Required |
| ------------- | --------------------- | -------- |
| Content-Type  | application/json      | Yes      |
| Accept        | application/json      | Yes      |
| Authorization | Bearer {access_token} | Yes      |

---

# Endpoint

| **Type** | **Endpoint**  | **Description**                |
| -------- | ------------- | ------------------------------ |
| GET      | `/positions`  | Retrieve all current positions |

---

# Get Positions {#get-positions}

Returns all positions held by the client in the current trading session, along with relevant position details such as quantity, average traded price, and UnRealized/Realized P&L.

## What is the use of this API?

This endpoint is primarily used for **portfolio and risk management**. It provides a real-time snapshot of your trading Exposure. You would use this API to:

- **Calculate Mark-to-Market (MTM) P&L:** Track your UnRealized and Realized profit or loss (`UnRealized` and `Realized` fields) across all active and squared-off trades.
- **Risk & Margin Assessment:** Check your total `Exposure`, net positions (`NetPosition`), and turnover values to ensure you stay within your margin limits.
- **Square-Off Decisions:** Identify open positions (`OpenBuyQuantity`, `OpenSellQuantity`, `NetPosition`) that need to be closed before the end of the trading day.
- **Trade Reconciliation:** Review your average buy/sell prices and trade counts to verify that orders were executed as expected.

It provides a consolidated view of:
- Open positions (where buy quantity != sell quantity)
- Intraday positions (MIS/day trades)
- Carry-forward positions (if applicable)

!!! note "**Session-Based Data**"
    Positions are reset at the end of each trading session. Carry-forward positions persist across sessions where applicable.

!!! tip "**Use Positions for Risk Management**"
    Monitor `Exposure`, `UnRealized`, and `NetPosition` fields to stay within margin limits and make informed square-off decisions before market close.

---

## cURL Example

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/positions' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

## Response

### Success Response (200 OK)

<details>
<summary>Click to view Positions JSON Body</summary>

```json
{
    "SAM001":  [
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  110010000002885,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  0,
                       "LastTradedPrice":  0,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  0,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  110010000011536,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  -2.400000000000091,
                       "LastTradedPrice":  2207,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  4416.4,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  110010000014366,
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
                       "Exposure":  28.74,
                       "UnRealized":  0,
                       "Realized":  0,
                       "LastTradedPrice":  14.37,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  0,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  262650000056980,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  4465.5,
                       "LastTradedPrice":  201.8,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  151508.5,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  262720000068407,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  -8801,
                       "LastTradedPrice":  23321.3,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  51502373,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  1110010000500325,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  -0.7000000000000455,
                       "LastTradedPrice":  1251.9,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  2503.7,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  1110010000532540,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  0,
                       "LastTradedPrice":  0,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  0,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  1262600000862237,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  -24608,
                       "LastTradedPrice":  279.8,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  585208,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  1262600000862498,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  0,
                       "LastTradedPrice":  400.95,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  0,
                       "TurnoverValueOptions":  0
                   },
                   {
                       "EntityId":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
                       "ClientId":  "SAM001",
                       "RMSEntityCode":  "",
                       "InstrumentId":  1262600000869699,
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
                       "Exposure":  0,
                       "UnRealized":  0,
                       "Realized":  -17494,
                       "LastTradedPrice":  372.25,
                       "OpenPositionTradeValue":  0,
                       "TurnoverValue":  759294,
                       "TurnoverValueOptions":  0
                   }
               ]
}
```
</details>

## Response Field Descriptions

| Field                       | Type    | Description                                                           |
| --------------------------- | ------- | --------------------------------------------------------------------- |
| `{ClientId}`                     | object  | The root key is the dynamic `ClientId` (e.g., `"Algo123"`), containing an array of position objects. |
| `EntityId`                 | string  | Identifier for the entity                                             |
| `ClientId`                 | string  | Client identifier                                                     |
| `RMSEntityCode`           | string  | RMS entity code                                                       |
| `InstrumentId`             | integer | Internal ID of the instrument                                         |
| `ExchangeSegment`          | string  | Exchange segment (e.g., NSECM, NSEFO)                                 |
| `InstrumentName`           | string  | Name of the instrument                                                |
| `NonAckBuyQuantity`      | integer | Non-acknowledged buy quantity                                         |
| `NonAckSellQuantity`     | integer | Non-acknowledged sell quantity                                        |
| `OpenBuyQuantity`         | integer | Total open buy quantity                                               |
| `OpenSellQuantity`        | integer | Total open sell quantity                                              |
| `ShortPosition`            | integer | Total short position                                                  |
| `LongPosition`             | integer | Total long position                                                   |
| `NetPosition`              | integer | Net position in the instrument                                        |
| `NetPositionInLot`       | integer | Net position measured in lots                                         |
| `OrderCount`               | integer | Total order count                                                     |
| `TradeCount`               | integer | Total trade count                                                     |
| `PreviousNetPosition`     | integer | Net position from the previous trading session                        |
| `TradeBuyValue`           | number  | Total buy value                                                       |
| `TradeSellValue`          | number  | Total sell value                                                      |
| `TradeNetValue`           | number  | Net trade value                                                       |
| `AvgBuy`                   | number  | Average buy price                                                     |
| `AvgSell`                  | number  | Average sell price                                                    |
| `AvgNet`                   | number  | Average net price                                                     |
| `Exposure`                  | number  | Total Exposure                                                        |
| `UnRealized`                | number  | UnRealized profit/loss                                                |
| `Realized`                  | number  | Realized profit/loss                                                  |
| `LastTradedPrice`         | number  | Last traded price (LTP) of the instrument                             |
| `OpenPositionTradeValue` | number  | Trade value of the open position                                      |
| `TurnoverValue`            | number  | Total turnover value                                                  |
| `TurnoverValueOptions`    | number  | Total turnover value for options                                      |




