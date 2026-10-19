# Troubleshooting

## Overview

The troubleshooting FAQ is configuration-driven.

FAQ entries are defined under:

```text
faq
```

in:

```text
process_config.json
```

An FAQ entry can reference:

```text
check_ids
```

or:

```text
check_group
```

## Whole Application Unreachable

Check the Core / Foundation group:

```text
Docker Engine
Redis
NATS
MySQL
Gateway
AuthServer
```

## 502 Bad Gateway

Check the Web UI / API group:

```text
Trader Web
Admin Web
ApiApplication
ApiAdminApplication
```

## Charts Not Loading

Check:

```text
MDSCore
```

## No Live Prices

Check the Market Data group:

```text
Market Feed Service
MDAPI
MD Streaming
```

For NNF-specific market data issues, check:

```text
Blitz NNF
Blitz NNFCM
```

## Orders Not Going Through

Check:

```text
BlitzServer
RMS Validator
```

## BOD / EOD Not Running

Check:

```text
Process Master
```

## Troubleshooting Flow

```mermaid
flowchart TD

    A[Application Issue] --> B{Application Completely Unreachable?}

    B -->|Yes| C[Check Core / Foundation]
    C --> C1[Docker Engine]
    C1 --> C2[Redis]
    C2 --> C3[NATS]
    C3 --> C4[MySQL]
    C4 --> C5[Gateway]
    C5 --> C6[AuthServer]

    B -->|No| D{502 Bad Gateway?}
    D -->|Yes| E[Check Web UI / API]

    D -->|No| F{Charts Not Loading?}
    F -->|Yes| G[Check MDSCore]

    F -->|No| H{No Live Prices?}
    H -->|Yes| I[Check Market Data]

    H -->|No| J{Orders Not Going Through?}
    J -->|Yes| K[Check Order Processing]

    J -->|No| L{BOD/EOD Not Running?}
    L -->|Yes| M[Check Process Master]
```