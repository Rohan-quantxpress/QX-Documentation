# Report Generation

## Overview

The Process Health Report Generator produces three possible output formats:

```text
Markdown
HTML
JSON
```

JSON output is optional.

## Markdown Report

The function:

```text
render_markdown()
```

generates the Markdown report.

The report contains:

1. Summary
2. At a Glance
3. HTTP / Port Health Checks
4. Group Impact Analysis
5. Detailed Process Status
6. Troubleshooting FAQ

## Summary

The summary contains:

- Total monitored processes
- Running processes
- Down processes
- Critical processes down
- Host CPU
- Host memory
- Disk usage

Overall status can be:

```text
HEALTHY
DEGRADED
CRITICAL
```

## At a Glance

The At-a-Glance table combines:

- Docker containers
- EXE processes
- Python processes

Information includes:

- Process
- Type
- Status
- CPU / Memory
- Version / Image
- Instances
- Path / Container
- HTTP Check

## Detailed Process Status

Processes are grouped by:

```text
Core / Foundation
Web UI / API
Market Data
Order Processing
BOD / EOD
Other
```

Details include:

- Process type
- Critical flag
- Status
- Uptime
- PID
- Path
- Instance count
- Docker state
- Container name
- Image
- HTTP result
- CPU
- Memory

## HTML Report

The function:

```text
render_html()
```

generates a standalone HTML report.

The HTML report contains:

- Overview
- At a Glance
- HTTP Checks
- Groups
- Detailed Status
- FAQ

The HTML file contains its own CSS and JavaScript and does not require external assets.

## HTML Navigation

The sidebar contains:

```text
Overview
At a Glance
HTTP Checks
Groups
Detailed Status
FAQ
```

The active section is updated while scrolling using:

```javascript
IntersectionObserver
```

## HTML Escaping

Dynamic values inserted into the HTML are escaped using:

```text
_esc()
```

The function handles special characters such as:

```text
&
<
>
"
```

## JSON Report

When `--json-out` is supplied, the script generates raw evaluated status.

Example:

```json
{
    "os": "...",
    "results": {},
    "groups": {}
}
```

The JSON contains evaluated process and group results.