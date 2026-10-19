# **Logout / Session API**

The **Logout / Session API** allows you to terminate the authenticated session and revoke the access token.

Using this API, you can securely sign out from the Blitz Interactive API, invalidating the current token so it can no longer be used.

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

| **Type** | **Endpoint** | **Description**                 |
| -------- | ------------ | ------------------------------- |
| POST     | `/session/logout` | Terminate the current session and revoke the token |

---

# Logout {#logout}

Terminates the session identified by the `sessionId` in the request body (falls back to the `sessionId` / `jti` claim in the token) and revokes the presented bearer token.

After a successful logout, the access token can no longer be used to call any interactive API.

## Request

Optionally specify which session to revoke:

```json
{
  "sessionId": "string"
}
```

If omitted, the session id embedded in the bearer token is used.

## cURL Example

```bash
curl -X 'POST' \
  'https://{baseurl}/api_interactive/api/v1/session/logout' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -d '{
    "sessionId": "string"
  }'
```

## Response

### Success Response (200 OK)

<details>
<summary>Click to view Logout JSON Body</summary>

```json
{
    "Message":  "Logged out and token revoked.",
    "RevokedSessions":  1
}
```
</details>

## Response Field Descriptions

| Field             | Type    | Description                                                   |
| ----------------- | ------- | ------------------------------------------------------------- |
| `message`         | string  | Human-readable logout status message                          |
| `revokedSessions` | integer | Number of sessions revoked (0 if no active session was found) |
