# Logout / Session

Terminate the authenticated session and revoke the access token.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    # Optionally pass a specific session id; otherwise the token's session is used
    response = client.logout()
    print(response["message"], response["revokedSessions"])
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var result = await client.LogoutAsync();
    Console.WriteLine($"{result.Message} ({result.RevokedSessions} revoked)");
    ```

---

## Endpoint

```
POST /session/logout
```

### Request

Optionally specify which session to revoke:

```json
{
  "sessionId": "string"
}
```

If omitted, the session id embedded in the bearer token is used.

```bash
curl -X POST "https://{baseurl}/api_interactive/api/v1/session/logout" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"sessionId": "string"}'
```

### Response

```json
{
  "message": "Logged out and token revoked.",
  "revokedSessions": 1
}
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `message` | string | Logout status message |
| `revokedSessions` | int | Number of sessions revoked |