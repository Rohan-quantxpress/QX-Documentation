# Libraries and SDKs

Below is a list of client libraries for BlitzConnect written in various programming languages. These libraries allow you to interact with the APIs without having to make raw HTTP calls.

## Prerequisites

1. **Get API Credentials** — Create an app and obtain an API key from the [Blitz portal](../../userSetup/generateKey.md).
2. **Python** (for Python SDK) — Install Python 3.10+ from [python.org](https://www.python.org/). Ensure pip is up to date: `python -m pip install --upgrade pip`
3. **.NET 8** (for C# SDK) — Install the .NET 8 SDK from [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet/8.0).

| Language   | Repository                  | Examples                              |
| ---------- | --------------------------- | ------------------------------------- |
| Python     | `blitzsdk` package (local)  | `test_interactive_apis.py`, `test_interactive_ws.py` |
| C# (.NET 8) | `QX.BlitzConnect`          | `BlitzConnectTest/`                   |

=== "Python"

    ```bash
    # Install dependencies
    pip install -r requirements.txt
    ```

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")
    # Login happens automatically when the client is constructed
    orders = client.get_orders()
    ```

=== "C#"

    Add a project reference to your `.csproj`:

    ```xml
    <ItemGroup>
      <ProjectReference Include="..\QX.BlitzConnect\BlitzConnect.csproj" />
    </ItemGroup>
    ```

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Common;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();
    ```

## Version and API endpoint

The current stable version of the API is **1**. All requests go to the base URLs listed in the [API Reference overview](../blitzconnect-api/index.md#base-urls).

## Root API endpoints

| Service          | Endpoint                              |
| ---------------- | ------------------------------------- |
| Authentication   | `{AUTH_BASE_URL}` — see `config.py` `AUTH_BASE_URL` |
| Interactive      | `{API_BASE_URL}` — see `config.py` `API_BASE_URL`    |
| Interactive WS   | `{WS_URL}` — see `config.py` `WS_URL`    |
| Market Data      | `{MD_API_URL}` — see `config.py` `MD_API_URL`    |
| Instruments      | `{INSTRUMENT_URL}` — see `config.py` `INSTRUMENT_URL` |

## Built-in behavior

### Authentication & retries

| Behavior | Python (`blitzsdk`) | C# (`QX.BlitzConnect`) |
|---|---|---|
| Login | Automatic when a client object is constructed | Explicit `await client.LoginAsync()` (auto-login on first request if forgotten) |
| Request timeout | 15 s per request, 10 s for login | `RequestTimeoutSeconds` config property (default 30) |
| 401 handling | Re-logins once and retries the original request automatically | Clears token, re-logins and retries (not counted as a retry attempt) |
| Transport retries | None (connection errors raise immediately) | Up to 3 retries on network errors with 1 s / 2 s / 4 s backoff |
| TLS verification | **Disabled** (`verify=False`, InsecureRequestWarning suppressed) | Disabled (custom certificate callback accepts all certificates) |

!!! warning
    Both SDKs currently skip TLS certificate validation to support UAT environments. Enable proper certificate validation before production use.

### WebSocket reconnection

Both SDKs auto-reconnect after disconnects. The Python client retries every **5 seconds**; the C# clients back off exponentially from **1 s up to 30 s**. Subscriptions are replayed automatically after each reconnect (Python interactive also sends an app-level `"ping"` keep-alive every 30 s). See [WebSocket streaming](../blitzconnect-api/websocket.md) and [Market Data Streaming](../blitzconnect-api/market-data-streaming.md).

### Configuration

The Python SDK resolves settings from (in order): environment variable → JSON config file → empty default. Config file discovery: the `BLITZ_CONFIG_PATH` environment variable if it points to an existing file, else `<repo>/tests/test-config.json`.

| Setting | JSON key | Environment variable |
|---|---|---|
| Auth base URL | `AuthBaseUrl` | `BLITZ_AUTH_URL` |
| Interactive API URL | `InteractiveApiUrl` | `BLITZ_API_URL` |
| Interactive WS URL | `InteractiveWsUrl` | `BLITZ_WS_URL` |
| Market Data API URL | `MarketDataApiUrl` | `BLITZ_MD_API_URL` |
| Market Data WS URL | `MarketDataWsUrl` | `BLITZ_MD_WS_URL` |
| Instrument master URL | `InstrumentGzUrl` | `BLITZ_INSTRUMENT_URL` |
| App key | `AppKey` | `BLITZ_APP_KEY` |
| User ID | `UserId` | `BLITZ_USER_ID` |
| Client ID | `ClientId` | `BLITZ_CLIENT_ID` |

The C# SDK takes configuration via the `BlitzConfig` object (`AppKey`, `UserId`, `ClientId`, base-URL properties, `RequestTimeoutSeconds`) — no file or environment lookup.
