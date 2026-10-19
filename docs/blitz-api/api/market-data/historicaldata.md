# Historical Data API

The **Historical Data API** allows you to retrieve past market data for a specific instrument over a defined time interval.

Using this API, you can fetch OHLC (Open, High, Low, Close) data along with volume and timestamp information, which is essential for backtesting, technical analysis, and building charting applications.

> **Tip: Common Use Case** — Use this endpoint to generate historical candlestick charts or run quantitative backtests before executing live trading strategies.

**Base URL:** `https://{baseurl}/md-api`

## Endpoint

| Method | URL                             |
|--------|----------------------------------|
| POST   | `/marketfeed/historicalData`      |

## Headers

| Header        | Value                 | Required |
|---------------|-----------------------|----------|
| Content-Type  | application/json      | Yes      |
| Authorization | Bearer {access_token} | Yes      |

## Request Payload

You must provide the instrument identifier and the desired time interval for the candles.

### Parameters

| Parameter    | Type   | Required | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| `instrument` | string | Yes      | The exact instrument name (e.g., `IRFC`, `RELIANCE`).          |
| `interval`   | string | Yes      | Time interval of the candle data.                              |

## Supported Intervals & Examples

Below are the supported intervals. Select an interval to see the exact cURL request and a sample JSON response.

### 1 Min

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/historicalData' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "instrument": "IRFC",
    "interval": "1"
  }'
```

**Response**

```json
[
  {
    "open": 115.9,
    "high": 115.97,
    "low": 115.8,
    "close": 115.94,
    "volume": 82563,
    "oi": 0,
    "timestamp": "10-02-2026 09:32:00"
  },
  {
    "open": 115.99,
    "high": 115.99,
    "low": 115.83,
    "close": 115.9,
    "volume": 38996,
    "oi": 0,
    "timestamp": "10-02-2026 09:31:00"
  },
  {
    "open": 116,
    "high": 116,
    "low": 115.89,
    "close": 116,
    "volume": 130817,
    "oi": 0,
    "timestamp": "10-02-2026 09:30:00"
  }
]
```

### 5 Min

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/historicalData' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "instrument": "RELIANCE",
    "interval": "5"
  }'
```

**Response**

```json
[
  {
    "open": 1467.4,
    "high": 1467.6,
    "low": 1466.4,
    "close": 1466.9,
    "volume": 28183,
    "oi": 0,
    "timestamp": "10-02-2026 09:30:00"
  },
  {
    "open": 1465.1,
    "high": 1467.6,
    "low": 1464.5,
    "close": 1467.2,
    "volume": 63461,
    "oi": 0,
    "timestamp": "10-02-2026 09:25:00"
  },
  {
    "open": 1466.4,
    "high": 1466.4,
    "low": 1464.5,
    "close": 1464.8,
    "volume": 85113,
    "oi": 0,
    "timestamp": "10-02-2026 09:20:00"
  }
]
```

### 15 Min

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/historicalData' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "instrument": "TCS",
    "interval": "15"
  }'
```

**Response**

```json
[
  {
    "open": 2964.9,
    "high": 2968.5,
    "low": 2964.2,
    "close": 2965.9,
    "volume": 41763,
    "oi": 0,
    "timestamp": "10-02-2026 09:30:00"
  },
  {
    "open": 2957.8,
    "high": 2965.4,
    "low": 2954,
    "close": 2965,
    "volume": 138733,
    "oi": 0,
    "timestamp": "10-02-2026 09:15:00"
  }
]
```

### 30 Min

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/historicalData' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "instrument": "IRFC",
    "interval": "30"
  }'
```

**Response**

```json
[
  {
    "open": 147810,
    "high": 148195,
    "low": 147610,
    "close": 147880,
    "volume": 181,
    "oi": 0,
    "timestamp": "10-02-2026 09:45:00"
  },
  {
    "open": 147145,
    "high": 148290,
    "low": 147010,
    "close": 147820,
    "volume": 1228,
    "oi": 0,
    "timestamp": "10-02-2026 09:15:00"
  }
]
```

### 60 Min

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/historicalData' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "instrument": "RELIANCE",
    "interval": "60"
  }'
```

**Response**

```json
[
  {
    "open": 1419.2,
    "high": 1432.3,
    "low": 1415.7,
    "close": 1430.1,
    "volume": 7763478,
    "oi": 0,
    "timestamp": "29-04-2026 15:00:00"
  },
  {
    "open": 1415.6,
    "high": 1419.4,
    "low": 1411.5,
    "close": 1419.3,
    "volume": 3659628,
    "oi": 0,
    "timestamp": "29-04-2026 14:00:00"
  },
  {
    "open": 1415.3,
    "high": 1419.3,
    "low": 1414.3,
    "close": 1415.5,
    "volume": 3742500,
    "oi": 0,
    "timestamp": "29-04-2026 13:00:00"
  }
]
```

### 1 Day

```bash
curl -X POST 'https://{baseurl}/md-api/marketfeed/historicalData' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'Accept: */*' \
  -d '{
    "instrument": "NIFTY",
    "interval": "D"
  }'
```

**Response**

```json
[
  {
    "open": 24044.9,
    "high": 24181.75,
    "low": 23958.05,
    "close": 24016.5,
    "volume": 44752,
    "oi": 0,
    "timestamp": "28-04-2026 09:15:00"
  },
  {
    "open": 23965.35,
    "high": 24130.3,
    "low": 23952,
    "close": 24110.2,
    "volume": 44634,
    "oi": 0,
    "timestamp": "27-04-2026 09:15:00"
  },
  {
    "open": 24109.55,
    "high": 24203.35,
    "low": 23815.35,
    "close": 23903.95,
    "volume": 44573,
    "oi": 0,
    "timestamp": "24-04-2026 09:15:00"
  }
]
```

---

**Empty Payload for User**

```json
{
  "instrument": "",
  "interval": ""
}
```

---

## Understanding the Response

A successful request returns an array of OHLC candle objects sorted chronologically.

### Response Field Descriptions

| Field       | Type   | Description                                                  |
|-------------|--------|--------------------------------------------------------------|
| `open`      | number | Opening price of the interval                                |
| `high`      | number | Highest price traded during the interval                     |
| `low`       | number | Lowest price traded during the interval                      |
| `close`     | number | Closing price of the interval                                |
| `volume`    | number | Total number of shares or contracts traded during this period|
| `oi`        | number | Open interest                                                |
| `timestamp` | string | Date and start time of the candle (`DD-MM-YYYY HH:mm:ss`)    |
