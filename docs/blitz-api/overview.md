# **Welcome to the BlitzTrader Ecosystem**

The **BlitzTrader API** ecosystem provides all the tools you need. Our platform is designed to be user-friendly and easy to integrate.

The ecosystem is built around three core pillars, each handling a crucial part of the trading lifecycle:

| Component       | What it does                                               | Who it's for                                          |
| --------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| **BlitzTrader API** | Direct REST-based endpoints for trading and market data. | Users who want full control over their HTTP requests. |
| **Blitz SDK**   | Pre-built libraries to quickly connect to Blitz without writing boilerplate code. | Users who want to build features fast in their preferred language. |
| **TPOMS**       | A built-in dashboard feature where you can configure and manage your external broker connections. | Any user who needs to link their trading account to external brokers. |

---

## **1. BlitzTrader API**

The **BlitzTrader API** gives you direct, unmediated access to our platform via standard HTTP requests. Everything you can do on a trading terminal, you can do through our API.

### What can you build with it?

- **Order Management:** Place, modify, cancel, and track orders in real-time.
- **Portfolio Tracking:** Retrieve your daily trades, open orders, and current positions.
- **Market Data:** Access live streaming quotes or pull historical data for backtesting.

### Why use the API?

- **Universally Compatible:** Since it uses JSON over REST, you can connect to it using any programming language or tool (like Postman or curl).
- **Predictable:** Uses standard HTTP methods (GET, POST) and clear status codes.

➡️ *Dive into the **[API Reference](api/API_Structure.md)** for detailed endpoint documentation.*

---

## **2. Blitz SDK: Build Faster**

Don't want to handle the low-level HTTP networking yourself? The **Blitz SDK** provides ready-to-use client libraries that wrap the API for you.

### How the SDK makes your life easier:

- **Automatic Authentication:** Handles login and token management automatically.
- **Built-in Error Handling:** Parses errors cleanly so your app doesn't crash unexpectedly.
- **Data Formatting:** Automatically formats your requests and responses into native objects.

Instead of writing networking code, you can focus entirely on your trading strategy and business logic.

➡️ *Check out the **[Blitz SDK](sdk/blitzconnect/index.md)** section to get started.*

---

## **3. TPOMS: Centralized Broker Management**

**TPOMS** stands for *Third-Party Order Management System*. It is a built-in feature available directly from the Blitz UI that allows you to connect and manage your external market brokers (like Zerodha, Motilal, etc.) in one place.

➡️ *Learn more about broker integrations in the **[TPOMS Overview](tpoms/overview.md)**.*

---

## **Before You Begin — Get Your Credentials**

To call the APIs, you need a `userId` and an `appKey`. Here's how to get them:

| Step | What to do | Guide |
|------|-----------|-------|
| 1 | **Create an account** on the BlitzTrader portal | [Create Account](userSetup/createAccount.md) |
| 2 | **Log in** and create an App in the **My Apps** dashboard | [Generate API Key](userSetup/generateKey.md) |
| 3 | Your `appKey` is shown once after key generation — **copy and store it securely** | — |
| 4 | Use your chosen `userId` (set during sign-up) and the `appKey` to get an access token | [Authentication API](api/authentication.md) |

!!! tip "Forgot your password?"
    Use the [Forgot Password](userSetup/forgotPassword.md) link on the login screen to reset it.

---

## **Quickstart: Your First Trade**

Ready to see it in action? Here is a simple three-step process to place your first order using raw HTTP requests.

### Step 1: Generate an Access Token

Before you can interact with the system, you must log in to verify your identity. This request returns a secure token that you must include in all future requests.

```bash
curl -X 'POST' \
  'https://{baseurl}/api_gateway/v1/api/app_login' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "userId": "Algo123",
  "appKey": "CPXotUwWdlIpgJbYrguSnVyu05IZawswKaXKNQ"
}'
```

#### Response

```json
{
    "status": "success",
    "message": "request processed successfully.",
    "data": {
        "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcmVmZXJyZWRfdXNlcm5hbWUiOiJBbGdvMTIzIiwiYXBwTmFtZSI6IkFsZ29UZXN0IiwiaXBBZGRyZXNzIjoiMTcyLjE3LjAuMSIsImVudGl0eUlkIjoiODVhYmJlZWItMmU3Zi00NGE1LWI2NjgtODQ3MjBkMjljZWVkIiwidXNlcklkIjoiQWxnbzEyMyIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6IkFwcCIsImF1ZCI6WyJHYXRld2F5U2VydmVyIiwiQVBJU2VydmVyIl0sImV4cCI6MTc3OTg2Nzg0MCwiaXNzIjoiQXV0aFNlcnZlciJ9.zqaA2CHQpwzV-iZfq3h6mt1NaBm1AQdbUDw4kJuJgVI"
    }
}
```

### Step 2: Place an Order

Using the access token you just received (replace `{access_token}`), you can place an order. `ExchangeSegment` is **mandatory**, together with exactly one instrument identifier. *(Make sure to replace placeholder values with actual trading parameters when testing).*

```bash
curl -X 'POST' \
  'https://{baseurl}/api_interactive/api/v1/orders/placeOrder' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -d '{
  "CorrelationOrderId": "corr_458",
  "Quantity": 500,
  "Product": "MIS",
  "TIF": "GTD",
  "Price": 10.1,
  "OrderType": "Market",
  "ExchangeSegment": "NSEFO",
  "ExchangeInstrumentId": 2885,
  "OrderSide": "BUY",
  "DisclosedQuantity": 0,
  "StopPrice": 0,
  "ClientId": "Algo123",
  "TiF_GTD_Date": "2025-10-10"  // replace with your GTD expiry
}'
```

```json
{
  "status": "success",
  "message": "request processed successfully",
  "data": {
    "blitzOrderId": 218100636600000022,
    "correlationOrderId": "order_1787046999088"
  }
}
```

### Step 3: Check Order Status

Once your order is placed, you can fetch its current status to see if it was executed, pending, or rejected by the broker.

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/orders' \
  -H 'Content-Type: application/json' \
  -H 'Accept: */*' \
  -H 'Authorization: Bearer {access_token}'
```

!!! tip "**Where to go next?**"
    If you want to build direct integrations, start with the **[API Reference](api/API_Structure.md)**. If you want a smoother development experience, head over to the **[Blitz SDK](sdk/blitzconnect/index.md)**.
