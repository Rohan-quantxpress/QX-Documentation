# **Holdings API**

The **Holdings API** allows you to retrieve all **portfolio / demat holdings** of the client across different instruments.

Using this API, you can track your **positions held overnight**, along with quantity, average buy price, and instrument details for better portfolio management.

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

| **Type** | **Endpoint**          | **Description**                              |
| -------- | --------------------- | -------------------------------------------- |
| GET      | `/portfolio/holdings` | Retrieve all portfolio / demat holdings      |

---

# Get Holdings {#get-holdings}

Returns all the client holdings, i.e., instruments that have been purchased and are held in the account, along with the quantity and average buy price.

## What is the use of this API?

- **Long-Term Portfolio Tracking:** See which instruments you own (`ISIN`, `Ticker`, `InstrumentName`) and in what quantity.
- **Cost Basis:** Review `Quantity` and `BuyAvgPrice` for each holding to understand your average entry cost.
- **Portfolio Validation:** Confirm that deliveries/settled purchases appear in your holdings before taking further decisions.

## cURL Example

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/portfolio/holdings' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

## Response

### Success Response (200 OK)

<details>
<summary>Click to view Holdings JSON Body</summary>

```json
[
    {
        "UserID":  "SAM001",
        "ISIN":  "INE002A01018",
        "InstrumentName":  "RELIANCE",
        "Ticker":  "RELIANCE INDUSTRIES LTD",
        "InstrumentId":  "110010000002885",
        "ExchangeSegment":  "1",
        "Quantity":  20,
        "BuyAvgPrice":  1388.29
    },
    {
        "UserID":  "SAM001",
        "ISIN":  "INE040H01021",
        "InstrumentName":  "SUZLON",
        "Ticker":  "SUZLON ENERGY LIMITED",
        "InstrumentId":  "110010000012018",
        "ExchangeSegment":  "1",
        "Quantity":  20,
        "BuyAvgPrice":  53.23
    },
    {
        "UserID":  "SAM001",
        "ISIN":  "INE092T01019",
        "InstrumentName":  "IDFCFIRSTB",
        "Ticker":  "IDFC FIRST BANK LIMITED",
        "InstrumentId":  "110010000011184",
        "ExchangeSegment":  "1",
        "Quantity":  20,
        "BuyAvgPrice":  73.29
    },
    {
        "UserID":  "SAM001",
        "ISIN":  "INE733E01010",
        "InstrumentName":  "NTPC",
        "Ticker":  "NTPC LTD",
        "InstrumentId":  "110010000011630",
        "ExchangeSegment":  "1",
        "Quantity":  20,
        "BuyAvgPrice":  335.5
    },
    {
        "UserID":  "SAM001",
        "ISIN":  "INE775A01035",
        "InstrumentName":  "MOTHERSON",
        "Ticker":  "SAMVRDHNA MTHRSN INTL LTD",
        "InstrumentId":  "110010000004204",
        "ExchangeSegment":  "1",
        "Quantity":  20,
        "BuyAvgPrice":  103.03
    }
]
```
</details>

## Response Field Descriptions

| Field             | Type    | Description                                         |
| ----------------- | ------- | --------------------------------------------------- |
| `UserID`          | string  | User identifier                                     |
| `ISIN`            | string  | International Securities Identification Number       |
| `InstrumentName`  | string  | Name of the instrument                              |
| `Ticker`          | string  | Ticker symbol of the instrument                     |
| `InstrumentId`    | string  | Internal ID of the instrument                       |
| `ExchangeSegment` | string  | Exchange segment (e.g., NSECM, NSEFO)               |
| `Quantity`        | number  | Total quantity held                                 |
| `BuyAvgPrice`     | number  | Average buy price of the holding                    |
