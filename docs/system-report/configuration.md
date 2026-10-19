# Configuration

## Configuration File

The default configuration file is:

```text
process_config.json
```

The configuration defines:

- Server information
- Monitored processes
- Process types
- Critical status
- Categories
- Container/process matching rules
- HTTP checks
- Process groups
- Group impact
- Troubleshooting FAQ

## Server Configuration

The configured server is:

```text
UAT QuantXpress
```

The configured report title is:

```text
QX Platform Process Health Report
```

## Process Configuration

Each monitored process contains configuration information used by the health-check engine.

The configuration determines:

- What process should be checked
- How the process should be checked
- Whether the process is critical
- Which category it belongs to
- How the process should be identified
- Whether an HTTP check should be performed

## Process Types

The supported process types are:

| Type | Description |
|---|---|
| `docker_engine` | Checks Docker CLI and daemon |
| `docker` | Checks Docker container |
| `exe` | Checks native EXE process |
| `python` | Checks Python process |

## Critical Processes

A process can be configured as:

```json
{
    "critical": true
}
```

Critical processes affect the overall report status when they are down.

Configured critical components include:

- Docker Engine
- Redis
- NATS
- MySQL
- Gateway
- AuthServer

## Docker Configuration

Docker containers can be identified using either:

```text
container_name
```

or:

```text
container_name_prefix
```

### Exact Container

Use `container_name` when the container name is fixed.

### Prefix Matching

Use `container_name_prefix` when the container name contains a changing version.

Example:

```text
qx.blitzgateway_1.0.07.23
```

Configured prefix:

```text
qx.blitzgateway
```

When multiple containers match the prefix, the most recently created matching container is selected.

## HTTP Configuration

A process can optionally contain:

```json
{
    "http_check": {
        "path": "/",
        "expected_status": [200]
    }
}
```

Optional values include:

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

## Group Configuration

Groups define logical dependencies between processes.

Each group contains:

- Group name
- Process members
- Impact description

A group is healthy when all configured members are running.

If one or more members are down, the group is considered impacted.

## FAQ Configuration

Troubleshooting entries are configured under:

```text
faq
```

An FAQ entry can reference:

```text
check_ids
```

or:

```text
check_group
```

The report uses these references to evaluate the current health of the relevant processes or groups.