# API Structure

This section covers the fundamentals of working with the Blitz API, including authentication, request formats, response structures, and common API conventions.

---

## Base URL

All API requests should be made to:

```http
https://{baseurl}
```

---

## Authentication

All endpoints require authentication using an access token.

### Request

```bash
curl -X 'POST' \
  'https://{baseurl}/api_gateway/v1/api/app_login' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
    "appKey": "CPXotUwWdlIpgJbYrguSnVyu05IZawswKaXKNQ",
    "userId": "Algo123"
  }'
```

### Response

```json
{
    "status": "success",
    "message": "request processed successfully.",
    "data": {
        "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcmVmZXJyZWRfdXNlcm5hbWUiOiJBbGdvMTIzIiwiYXBwTmFtZSI6IkFsZ29UZXN0IiwiaXBBZGRyZXNzIjoiMTcyLjE3LjAuMSIsImVudGl0eUlkIjoiODVhYmJlZWItMmU3Zi00NGE1LWI2NjgtODQ3MjBkMjljZWVkIiwidXNlcklkIjoiQWxnbzEyMyIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6IkFwcCIsImF1ZCI6WyJHYXRld2F5U2VydmVyIiwiQVBJU2VydmVyIl0sImV4cCI6MTc3OTg2Nzg0MCwiaXNzIjoiQXV0aFNlcnZlciJ9.zqaA2CHQpwzV-iZfq3h6mt1NaBm1AQdbUDw4kJuJgVI"
    }
}
```

Include the **access token** in the request header:

```http
Authorization: Bearer {access_token}
```

> **Warning — Security:** Never expose your access token in public repositories, frontend JavaScript code, or client-side applications. Always keep it secure and handle it strictly via your backend systems.

---

## HTTP Methods

The API relies on standard HTTP methods (verbs) to indicate the intention of the request:

| Method   | Description                                                              |
|----------|--------------------------------------------------------------------------|
| **GET**  | Retrieve data (e.g., fetch open orders, check current positions, retrieve trades). |
| **POST** | Create new resources or trigger actions (e.g., place a new order).       |
| **PUT**  | Update existing resources (e.g., modify a pending order).                |
| **DELETE**| Remove or cancel resources (e.g., cancel an open order).                |

---

## Request Structure

A typical API request involves setting the correct headers and, for `POST` and `PUT` methods, sending a JSON payload.

### Standard Request Headers

For every request, ensure you include the following headers:

- `Content-Type: application/json` — Informs the server that you are sending JSON data in the request body.
- `Accept: application/json` — Informs the server that you expect a JSON response.
- `Authorization: Bearer {access_token}` — Provides your active session token for authentication.

### Example Request

```bash
curl -X [API_METHOD] \
  'https://{baseurl}/api_interactive/api/v1/[ENDPOINT]' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -d '{"key": "value"}'
```

---

## Response Structure

All API responses, regardless of success or failure, are wrapped in a standard JSON envelope. This ensures predictability when parsing responses in your application.

### Success Envelope

A successful request will typically return an HTTP `200 OK` status, along with the following structure:

```json
{
  "status": "success",
  "message": "request processed successfully",
  "data": {
     // Endpoint-specific payload goes here
  }
}
```

- **`status`**: Indicates the result of the request. Commonly `"success"` or `"error"`.
- **`message`**: A human-readable string providing additional context about the response or describing an error if one occurred.
- **`data`**: The actual payload of the response. Depending on the endpoint, this can be an object `{}` or an array `[]`.
