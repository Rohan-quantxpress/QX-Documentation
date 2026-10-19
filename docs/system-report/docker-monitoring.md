# Docker Monitoring

## Docker Engine

Docker Engine health is checked using:

```text
check_docker_engine()
```

The script first verifies that the Docker CLI exists on the system `PATH`.

It then executes:

```bash
docker info --format "{{.ServerVersion}}"
```

A successful command indicates that the Docker daemon is reachable.

## Docker Container

Container health is checked using:

```text
check_docker_container()
```

A container can be configured using:

```text
container_name
```

or:

```text
container_name_prefix
```

## Exact Name Matching

When `container_name` is configured, the specified container is checked directly.

## Prefix Matching

When `container_name_prefix` is configured, the script searches all Docker containers:

```bash
docker ps -a --format "{{.Names}}"
```

It selects names starting with the configured prefix.

## Versioned Containers

Prefix matching is useful when container names contain version information.

Example:

```text
qx.blitzgateway_1.0.07.23
```

Prefix:

```text
qx.blitzgateway
```

If multiple containers match, the most recently created matching container is selected.

## Container Information

The tool collects:

- Running state
- Container state
- Start time
- Uptime
- Restart count
- Image
- Image ID
- Resolved container name

If the container exists but is stopped:

```text
running = false
```

## Docker Published Ports

When an HTTP check does not specify a port, the script attempts to discover Docker published ports using:

```bash
docker port <container>
```

## Docker Resource Usage

The function:

```text
get_docker_stats_batch()
```

uses:

```bash
docker stats --no-stream
```

to collect:

- CPU percentage
- Memory used
- Memory limit
- Memory percentage

The command is executed as a batch operation for running containers.