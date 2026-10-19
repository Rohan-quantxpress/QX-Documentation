# Process Groups

## Overview

Process groups represent logical sections of the QX Platform.

Groups allow the health report to describe not only whether individual processes are running, but also the impact of a process failure on a larger application component.

## Group Evaluation

The function:

```text
evaluate_groups()
```

evaluates every configured group.

A group is:

```text
Healthy
```

when all members are running.

A group becomes:

```text
Impacted
```

when one or more members are down.

## Group Result

A group result contains:

- Group display name
- Impact if down
- Members
- Down members
- Healthy status

## Core / Foundation

Members:

```text
Docker Engine
Redis
NATS
MySQL
Gateway
AuthServer
```

Impact:

```text
The application will not work at all.
```

## Web UI / API

Members:

```text
Trader Web
Admin Web
ApiApplication
ApiAdminApplication
```

Impact:

```text
Trader UI and/or Admin UI may become unreachable or return errors.
```

## Market Data

Members:

```text
Market Feed Service
MDAPI
MD Streaming
```

Impact:

```text
Live prices and market watch may stop updating.
```

## NNF Market Data Adapters

Members:

```text
Blitz NNF
Blitz NNFCM
```

Impact:

```text
NNF/NNFCM sourced prices may stop updating.
```

## Order Processing

Members:

```text
BlitzServer
RMS Validator
```

Impact:

```text
Orders may not be accepted, validated, or routed.
```

## BOD / EOD

Member:

```text
Process Master
```

Impact:

```text
BOD/EOD batch processes may not run or complete.
```