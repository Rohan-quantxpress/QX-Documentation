# Process Monitoring

## Command Execution

The `_run()` function executes system commands using:

```python
subprocess.run()
```

The command timeout is:

```text
15 seconds
```

The function returns:

- Success status
- Standard output
- Standard error

It handles:

- Command not found
- Command timeout
- Other execution errors

## Docker Engine Check

The function:

```text
check_docker_engine()
```

checks whether Docker is available and whether the Docker daemon is reachable.

It executes:

```bash
docker info --format "{{.ServerVersion}}"
```

Possible failures include:

- Docker CLI missing
- Docker daemon unavailable
- Docker service not started
- Permission problem

## Native Process Check

The function:

```text
check_native_process()
```

handles:

```text
exe
python
```

processes using `psutil`.

## EXE Process Matching

For:

```text
type = exe
```

the process name is obtained from:

```text
windows_process_name
```

on Windows and:

```text
linux_process_name
```

on Linux.

Process names are matched case-insensitively.

## Python Process Matching

For:

```text
type = python
```

the script searches for:

```text
match_cmdline
```

inside the running process command line.

Examples:

```text
process_master.py
tpoms
```

## Process Information

For running native processes, the tool records:

- PID
- Process path
- Instance count
- Uptime
- Individual instance paths

If multiple matching instances are found, all instances are retained.

## Process Path Resolution

The `_resolve_process_path()` function determines the process path.

For EXE processes:

```python
psutil.Process(pid).exe()
```

is used.

For Python processes, the command line is searched for the configured match value or a `.py` argument.

## Uptime

Process and container uptime is converted to:

```text
Xd Yh Zm
```

Example:

```text
2d 4h 18m
```

The functions responsible are:

```text
_humanize_since()
_humanize_since_epoch()
```

## Main Evaluation Function

The main orchestration function is:

```text
evaluate_all()
```

It evaluates every configured process and calls the appropriate checker based on process type.

```text
docker_engine → check_docker_engine()
docker         → check_docker_container()
exe            → check_native_process()
python         → check_native_process()
```

After checking the process, the function adds:

- Display name
- Category
- Critical flag
- Description
- Type

It then performs HTTP checks and resource collection.