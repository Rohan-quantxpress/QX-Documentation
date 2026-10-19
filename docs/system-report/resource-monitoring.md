# Resource Monitoring

## Overview

The Process Health Report Generator collects resource information at both process/container level and host level.

## Docker Resources

The function:

```text
get_docker_stats_batch()
```

runs:

```bash
docker stats --no-stream
```

It collects:

- CPU percentage
- Memory used
- Memory limit
- Memory percentage

## Native Process Resources

For EXE and Python processes, `psutil` is used to collect:

```text
CPU %
Memory RSS
```

CPU measurement uses a short sampling window of approximately:

```text
0.3 seconds
```

## Multiple Process Instances

If multiple instances of the same process are running, the tool records:

- CPU per instance
- Memory per instance

It also calculates totals for the parent process result.

## Host Resources

The function:

```text
get_system_resources()
```

collects:

- CPU utilization
- Memory usage
- Disk usage

## Disk Paths

On Windows:

```text
C:
```

On Linux:

```text
/
```

## Host Resource Output

The report contains:

- Total
- Used
- Free
- Percentage used