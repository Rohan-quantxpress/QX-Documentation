# BlitzConnect API Reference

REST and WebSocket APIs for algorithmic trading on the Blitz platform — place and manage orders,
stream live market data, manage portfolios, and more.

> This section covers the **BlitzConnect API** using SDK usage examples.
> For raw REST API docs, see [REST API Reference](../../api/API_Structure.md).

---

<div class="grid cards" markdown>

-   __Orders & Trading__

    Place, modify, cancel orders in real time. Manage positions, view trade history, and receive signals.

    - [Orders](orders.md)
    - [Positions](positions.md)
    - [Trades](trades.md)
    - [Signals](signals.md)
    - [Statistics](statistics.md)

-   __Market Data__

    Real-time LTP, market quotes, option chains, instrument master, and historical candle data.

    - [LTP](ltp.md)
    - [Market Quote](market-quote.md)
    - [Option Chain](option-chain.md)
    - [Instrument Detail](instrument-detail.md)
    - [Instruments](instruments.md)
    - [Historical](historical.md)
    - [Market Data Streaming](market-data-streaming.md)

-   __Realtime & Streaming__

    WebSocket streaming for order updates, portfolio events, and statistics.

    - [Interactive WebSocket](websocket.md)
    - [Market Data Streaming](market-data-streaming.md)

</div>

---

## Base URLs

| Service | URL |
|---|---|
| Auth | `https://{baseurl}/api_gateway/v1` |
| Interactive API | `https://{baseurl}/api_interactive/api/v1` |
| Market Data | `https://{baseurl}/md-api` |
| Instruments | `https://{baseurl}/v1/api` |

WebSocket endpoints follow the same pattern: `ws://<host>/api_interactive/ws?access_token=<jwt>` (interactive) and `ws://<host>/md-streaming/ws?key=<jwt>` (market data).

---

## Getting started

1. **Get credentials** — `app_key` + `user_id` from the Blitz portal.
2. **Authenticate** — Call the login endpoint to get an `access_token`.
3. **Use the SDK** — [BlitzConnect SDK](../blitzconnect/index.md) provides Python and C# libraries.
