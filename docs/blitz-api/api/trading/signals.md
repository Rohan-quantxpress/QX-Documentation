# **Signals API**

The **Signals API** acts as an instruction manual for how an automated trading system works in everyday language. It explains how a **Trading Strategy** (the computer program making the decisions) communicates with an **Execution Engine** (the program that actually places trades on the stock market) to send actionable trading instructions to the Blitz OMS/Strategy.

Using this API, you can broadcast **Start / Stop / Buy / Sell / Multi-instrument signals** between strategies or modules in real time.

This API is useful for monitoring, routing, and executing strategy-level instructions across different modules.

---

# Endpoints

| Type | Endpoint          | Description                                            |
| ---- | ----------------- | ------------------------------------------------------ |
| POST | [signals](#post-signal) | Send one or more signals to the strategy engine |

# Post a Signal {#post-signal}

The request body is a **JSON array** of signal objects — one element per signal. (A single signal is sent as a one-element array.)

```bash
curl -X POST \
  "https://{baseurl}/api_interactive/api/v1/signals" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}' \
  -d '[
    {
      "SourceStrategy": "AmberX1",
      "DestinationStrategy": "Calculus",
      "SourceSID": "SINGLE_Calculus",
      "InstanceRunningMode": "Started",
      "GlobalAction": "Signal",
      "Instruments": [
        {
          "ExchangeSegment": "NSEFO",
          "InstrumentName": "NIFTY10FEB2625550PE",
          "Action": "ENTERLONG",
          "Lot": "1",
          "TimeStamp": "17-09-2026 13:30:00",
          "InfoText": "Entry Signal"
        }
      ]
    }
  ]'
```

## Successful Response

```json
{
  "status": "success",
  "message": "request processed successfully.",
  "data": {
    "message": "Signal received successfully."
  }
}
```

---

# How the Trading Message is Structured

Whenever the strategy program wants to place an order, it sends a package of digital information called a **Signal Payload**. To keep everything tidy and organized, this package is always split into two components:

1. **The Signal Object (The Envelope):** This gives the high-level context, explaining who is sending the message, where it is going, and what program generated it.
2. **The Instrument Object (The Trade Details):** This details the explicit instructions for the asset being traded, specifying exactly what contract to trade, how much of it, and when.

Splitting the message this way allows a single transmission to handle overall strategy context alongside precise transaction metrics at the same time.

---

# Field Descriptions

## The "Signal" Object (The Envelope)

The Signal Object contains the metadata required to identify, track, and route a signal within the system. While it does not include any trade execution details, it provides the contextual information needed for processing and managing the signal.

| Field                | Type   | Description                                                                                            |
| -------------------- | ------ | ------------------------------------------------------------------------------------------------------ |
| `SourceStrategy`     | string | The name of the specific trading algorithm that generated the trade idea (e.g., `AmberX1`).            |
| `DestinationStrategy`| string | The name of the system or engine assigned to receive, read, and execute this package (e.g., `Calculus`).|
| `SourceSID`          | string | A completely unique ID tracking down the specific running computer or background process.              |
| `InstanceRunningMode`| string | Displays whether the sending program is currently active and running (typically `Started` or `Stopped`).|
| `GlobalAction`       | string | A broad classification tag that tells the router how to process the message (typically set to `Signal`).|
| `Instruments`        | array  | List of instrument-level actions.                                                                      |

## The "Instrument" Object (The Trade Details)

The Instrument Object contains the trade execution details for a specific instrument. It specifies the action to be performed, including the instrument identifier, order quantity, order type, and other parameters required to execute the trade.

| Field             | Type   | Description                                                                                             |
| ----------------- | ------ | ------------------------------------------------------------------------------------------------------- |
| `ExchangeSegment` | string | The specific stock exchange and market segment where the order should clear. Standard: `NSECM`, `NSEFO`, `BSECM`, `BSEFO`. |
| `InstrumentName`  | string | The precise contract symbol you want to target (e.g., `NIFTY10FEB2625550PE`).                          |
| `Action`          | string | The exact core trading instruction being sent (`ENTERLONG`, `BUY`, `ENTERSHORT`, `SELL`, `GOFLAT`, `SETPOSITION`). |
| `Lot`             | string | Controls your trade sizing by defining the exact number of contract lots you want to allocate.          |
| `TimeStamp`       | string | The exact time the signal was generated.                                                                |
| `InfoText`        | string | A blank note space for developers to embed context descriptions, logs, or annotations.                  |

---

# Action Execution Logic & Examples

Below is a breakdown of how to use each action type within your signals payload. **All actions share the same payload envelope** — only `Action`, `Lot` and `InfoText` change. The signal object below is one element of the request's **top-level array** (wrap it in `[ … ]` to send):

```json
{
    "SourceStrategy": "AmberX1",
    "DestinationStrategy": "Calculus",
    "SourceSID": "SINGLE_Calculus",
    "InstanceRunningMode": "Started",
    "GlobalAction": "Signal",
    "Instruments": [
        {
            "ExchangeSegment": "NSEFO",
            "InstrumentName": "NIFTY10FEB2625550PE",
            "Action": "<action>",   // see table below
            "Lot": "<lots>",
            "TimeStamp": "18-12-2025 09:21:00",
            "InfoText": "<free-form note>"
        }
    ]
}
```

## Per-action rules

### 1. Initial Long Entry (`ENTERLONG`)

Initiates a new buy-side position when no active long position currently exists for the specified instrument. If an active Long position already exists, send `BUY` instead.

**Example scenario**: Strategy `AmberX1` signals `Calculus` to open a brand-new Long trade of 10 lots on `NIFTY10FEB2625550PE`.

!!! warning "**System Rule**"
    Only valid when no active Long position exists for this asset.

### 2. Scaling into a Long Trade (`BUY`)

Adds additional quantity to an active long position, increasing overall exposure. Parsed only when an active Long trade state is open for the target asset.

!!! warning "**System Rule**"
    Only valid when a Long trade is already live; treated as a size update rather than a separate trade.

### 3. Initial Short Entry (`ENTERSHORT`)

Initiates a new sell-side position when no active short position exists for the specified instrument. If an active Short position already exists, send `SELL` instead.

!!! warning "**System Rule**"
    Only valid when no active Short position exists for this asset.

### 4. Scaling into a Short Trade (`SELL`)

Adds additional quantity to an active short position. Requires an active Short position context to be open.

!!! warning "**System Rule**"
    Only valid when a Short trade is already live.

### 5. Exiting/Flattening a Position (`GOFLAT`)

Reduces or closes an existing position:

- **Full exit**: if the lots in your message match or exceed your current position.
- **Partial exit (scale-out)**: if the lots in your message are fewer than your current position — the remainder stays live.

**Example**: `Lot: "10"` when holding 20 lots exits exactly 10.

### 6. Setting a Specific Position Size (`SETPOSITION`)

Directly sets the position to an exact number of lots, overriding any existing position (or opening one if none exists). Useful for strategies that need precise position-size control.

---

# Quick Rule Summary Matrix

| Command Hook  | Payload `Action` value | What it Does                               | Required Position Setup                        |
| ------------- | ---------------------- | ------------------------------------------ | ---------------------------------------------- |
| `ENTERLONG`   | `"ENTERLONG"`          | Opens a brand-new Buy trade loop           | You must have no active Long positions open    |
| `BUY`         | `"BUY"`                | Adds more lots to your current Buy trade   | An active Long trade must already be open      |
| `ENTERSHORT`  | `"ENTERSHORT"`         | Opens a brand-new Short trade loop         | You must have no active Short positions open   |
| `SELL`        | `"SELL"`               | Adds more lots to your current Short trade | An active Short trade must already be open     |
| `GOFLAT`      | `"GOFLAT"`             | Closes or shrinks your active trade size   | An active open trade context is required       |
| `SETPOSITION` | `"SETPOSITION"`        | Sets position to an exact number of lots   | Overrides existing position or creates new one |




