# Authentication API

The **Authentication API** is used to securely authenticate users and generate an access token required for accessing protected APIs within the QuantXpress platform.

Before accessing any secured APIs such as Orders, Trades, Positions, Strategies, or WebSocket services, users must first authenticate using a valid `userId` and `appKey`.

Once authentication is successful, the API returns a JWT access token. This token must be included in the `Authorization` header for all subsequent secured API requests.

| Type | Endpoint                        | Description                                     |
|------|----------------------------------|-------------------------------------------------|
| POST | `api_gateway/v1/api/app_login`   | Authenticate user and generate access token     |

---

## Authentication

Authenticate a user using valid credentials and generate an access token.

### Endpoint

```http
POST /api_gateway/v1/api/app_login
```

### Authentication Flow

```text
Client Application
      │
      │ 1. Send userId + appKey
      ▼
Authentication API
      │
      │ 2. Validate credentials
      ▼
Generate JWT Access Token
      │
      │ 3. Return access token
      ▼
Client Stores Token
      │
      │ 4. Use token in secured APIs
      ▼
Protected APIs Accessible
```

### Request Headers

```http
Content-Type: application/json
Accept: */*
```

### cURL Example

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

### Request Body

```json
{
  "appKey": "CPXotUwWdlIpgJbYrguSnVyu05IZawswKaXKNQ",
  "userId": "Algo123"
}
```

### Request Field Descriptions

| Field    | Type   | Description                                          |
|----------|--------|------------------------------------------------------|
| `userId` | string | Unique user identifier used for authentication       |
| `appKey` | string | Application key assigned to the user                 |

---

## Response

### Success Response

<details>
<summary>Click to view Login JSON Body</summary>

```json
{
    "status":  "success",
    "message":  "request processed successfully.",
    "data":  {
                 "accessToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcmVmZXJyZWRfdXNlcm5hbWUiOiJTQU0wMDEiLCJhcHBOYW1lIjoiVGVzdEFwcCIsImlwQWRkcmVzcyI6IjE3Mi4xNy4wLjEiLCJlbnRpdHlJZCI6ImZlYjdmMDVkLWRmMGItNGQwMC05M2ZjLTNlNTliN2VkZjRiZiIsInVzZXJJZCI6IlNBTTAwMSIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6IkFwcCIsImF1ZCI6WyJHYXRld2F5U2VydmVyIiwiQVBJU2VydmVyIl0sImV4cCI6MTc5MDEwNjM3MiwiaXNzIjoiQXV0aFNlcnZlciJ9.gMT1WshqYUth5PUeuaAsvHAam98mKUBSBIvCcYngMEE"
             }
}
```
</details>

### Response Field Descriptions

| Field         | Type   | Description                                              |
|---------------|--------|----------------------------------------------------------|
| `status`      | string | Status of the API request                                |
| `message`     | string | Informational response message                           |
| `data`        | object | Contains authentication response data                    |
| `accessToken` | string | JWT access token used for authenticated API requests     |

---

## Access Token

After successful authentication, the API returns a JWT (JSON Web Token). It contains authenticated session details and is required for accessing secured APIs. See the [Success Response](#success-response) above for a full example.

---

## Using the Access Token

Include the access token in the `Authorization` header of all protected API requests.

```http
Authorization: Bearer {access_token}
```

### Example Authenticated Request

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/trades' \
  -H 'Authorization: Bearer {access_token}' \
  -H 'accept: */*'
```

---

## Authentication Lifecycle

```text
Application Starts
      │
      ▼
Call app_login API
      │
      ▼
Receive Access Token
      │
      ▼
Store Token Securely
      │
      ▼
Use Token in APIs
      │
      ▼
Token Expires
      │
      ▼
Re-authenticate
```

---

## Error Response

### Invalid Credentials

```json
{
  "status": "error",
  "message": "invalid userId or appKey",
  "data": null
}
```

### Common HTTP Status Codes

| Status Code | Description                       |
|-------------|-----------------------------------|
| `200`       | Authentication successful         |
| `400`       | Invalid request payload           |
| `401`       | Unauthorized or invalid credentials|
| `403`       | Access denied                     |
| `500`       | Internal server error             |

---

## Best Practices & Security

**Token handling**

- Store the access token securely in memory, cache, or encrypted storage — never in logs, source code, frontend applications, screenshots, or public repositories.
- Authenticate once during application startup and reuse the token across requests until it expires.
- Re-authenticate after token expiration; expired or invalid tokens fail with `401 Unauthorized`.
- Use centralized token management if multiple services or threads share APIs.

**Credentials & transport**

- Never expose `appKey` or `accessToken` publicly; do not hardcode credentials in production source code.
- Always use HTTPS endpoints in production environments to avoid credential leakage.
- Avoid generating unnecessary login sessions repeatedly in short intervals.
