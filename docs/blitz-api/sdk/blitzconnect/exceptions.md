# Exceptions and errors

In addition to the `40x` and `50x` headers, error responses come with the name of the exception generated internally by the API server.

```
HTTP/1.1 500 Server error
Content-Type: application/json

{
    "status": "error",
    "message": "Error message",
    "error_type": "GeneralException"
}
```

## Error types

| Exception             | HTTP | Description                                                |
| --------------------- | ---- | ---------------------------------------------------------- |
| `AuthenticationError` | 401  | Login failed — invalid credentials or token expired        |
| `TokenException`      | 403  | Session expired or invalidated. Must re-login              |
| `RequestError`        | 400–500 | General request failure with status code and message     |
| `InputException`      | 400  | Missing required fields, bad parameter values              |
| `OrderException`      | 400  | Order-related failures (placement, fetch, etc.)            |
| `MarginException`     | 400  | Insufficient funds for order placement                     |
| `PositionException`   | 400  | Position-related errors                                    |
| `NetworkException`    | 502  | Unable to communicate with the OMS                         |
| `DataException`       | 500  | Internal system error parsing OMS response                 |
| `GeneralException`    | 500  | Unclassified error                                         |

## Common HTTP error codes

| Code | Meaning                                                    |
| ---- | ---------------------------------------------------------- |
| `400` | Missing or bad request parameters or values                |
| `401` | Authentication failed                                      |
| `403` | Session expired or invalidated. Must re-login              |
| `404` | Requested resource not found                               |
| `405` | Request method not allowed on the endpoint                 |
| `429` | Too many requests (rate limiting)                          |
| `500` | Unexpected server error                                    |
| `502` | Backend OMS is down                                        |
| `503` | Service unavailable                                        |
| `504` | Gateway timeout                                            |

## SDK error handling

=== "Python"

    SDK methods **raise** on failure rather than returning error dicts:

    | Exception | Raised when |
    | --- | --- |
    | `AuthenticationError` | Login fails — transport error, non-200, or `status != "success"` |
    | `RequestError` (`status_code` property) | Any non-(200, 201) API response, connection failure (`0`), or timeout (`0`). One automatic re-login + retry is attempted on `401` first |
    | `ValueError` | Invalid local input — unknown instrument symbol, malformed `EXCHANGE\|NAME` string |
    | `RuntimeError` | Instrument master download returns a non-200 status |

    All SDK exceptions derive from `BlitzSDKException` (`from blitzsdk.common.exceptions import ...`).

=== "C#"

    All failures throw `BlitzConnectException`:

    | Property | Meaning |
    | --- | --- |
    | `HttpStatusCode` | HTTP status code; `0` for transport-level failures (retries exhausted) and deserialization errors |
    | `RawResponseBody` | Raw server body attached when available |
    | `ApiError` | Best-effort parsed `BlitzApiResponse<object>` of the error body |

    WebSocket clients never throw for runtime issues — they surface problems via `OnError` events instead.
