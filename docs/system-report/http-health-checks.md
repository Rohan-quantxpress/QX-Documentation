# HTTP and Port Health Checks

## Purpose

A process being present does not necessarily mean that its application is responding.

HTTP health checks verify whether the configured application endpoint is reachable.

## Configuration

Example:

```json
{
    "http_check": {
        "path": "/",
        "expected_status": [200]
    }
}
```

Optional properties:

```text
port
host
scheme
timeout
```

Defaults:

```text
host    = 127.0.0.1
scheme  = http
timeout = 3 seconds
```

## Port Discovery

### Docker

Docker ports are discovered using:

```bash
docker port <container>
```

### EXE / Python

For native processes, `psutil` is used to identify listening sockets belonging to the process.

If a port cannot be identified, the HTTP check is skipped.

The report indicates that the following should be configured:

```text
http_check.port
```

## HTTP Probe

The HTTP request has the following structure:

```text
GET <scheme>://<host>:<port><path>
```

Example:

```text
GET http://127.0.0.1:8080/health
```

## HTTP Response Handling

Any HTTP response proves that the port is reachable.

For example:

```text
200
404
500
```

all indicate that the server responded.

The following are treated as connection failures:

- Connection refused
- Connection failure
- Timeout

## Expected Status

The configured expected status list determines whether the HTTP check passes.

Example:

```json
"expected_status": [200]
```

Only the configured expected response status is considered successful.

## HTTP Check Workflow

```text
Resolve Port
     |
     v
Run HTTP Request
     |
     v
Receive Response
     |
     v
Compare Status
     |
     v
Store HTTP Health Result
```

If the process is not running, the HTTP check is skipped.