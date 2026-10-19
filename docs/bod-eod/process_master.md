### 1.2.3 `process_master.py` — Central Process Manager

`process_master.py` is the **central controller of the Process Manager**.

It is responsible for loading the process configuration, scheduling BOD and EOD activities, executing configured scripts, monitoring application processes, updating process information in Redis, and handling process failures.

The Process Manager does not contain the business logic of individual BOD or EOD activities. Instead, it provides the **execution framework** that allows those activities to run automatically according to their configuration.

In simple terms:

```text
process.json
      │
      ▼
process_master.py
      │
      ├── BOD Scheduler
      │       └── BOD Scripts
      │
      ├── EOD Scheduler
      │       └── EOD Scripts
      │
      ├── Process Monitor
      │       ├── EXE Processes
      │       └── Docker Containers
      │
      ├── Redis
      │       └── Status & Monitoring Data
      │
      └── Email
              └── Failure / Process Alerts
```

---

## Purpose of `process_master.py`

The main purpose of `process_master.py` is to provide a **centralized mechanism for running and managing system processes**.

Instead of manually starting individual scripts or applications, the Process Manager uses the configuration defined in `process.json` and performs the required operations automatically.

The file is responsible for the following major areas:

* BOD process scheduling and execution
* EOD process scheduling and execution
* Sequential process execution
* Process start, stop, and restart operations
* Runtime process monitoring
* Redis status and monitoring updates
* Failure handling
* Email notifications
* Manual process control
* Scheduler initialization

---

## How `process_master.py` Works

The Process Manager works together with `process.json`.

`process.json` contains the **configuration**, while `process_master.py` contains the **execution logic**.

For example, the configuration defines information such as:

```text
Process Title
Process Type
Schedule Time
Application Path
Parameters
Active / Inactive Status
Start Flag
Stop Flag
Restart Flag
```

The Process Manager reads these settings and determines how each configured process should be handled.

Therefore:

```text
process.json
    = What should run and how it should run

process_master.py
    = Executes and manages what is configured
```

## The configuration file contains separate sections for BOD processes, EOD processes, and process monitoring.

## BOD and EOD Execution

`process_master.py` manages both **BOD** and **EOD** process flows.

### BOD

BOD processes are executed according to the configured BOD schedule and process configuration.

A typical flow is:

```text
BOD Schedule
     ↓
Load Active BOD Processes
     ↓
Set Process Status
     ↓
Execute Process
     ↓
Capture Result
     ↓
Update Redis
     ↓
Continue to Next Process
```

The configured BOD processes can include activities such as downloading files, copying files, uploading instrument information, generating checksums, and other operational tasks.

### EOD

EOD processes follow the same centralized execution mechanism but are configured separately.

A typical EOD flow is:

```text
EOD Schedule
     ↓
Load Active EOD Processes
     ↓
Execute Process 1
     ↓
Execute Process 2
     ↓
Execute Process 3
     ↓
...
     ↓
Complete EOD
```

## The configured EOD processes may perform activities such as database backup, database cleanup, Redis backup/cleanup, Bhavcopy processing, and other end-of-day operations.

## Sequential Execution

The Process Manager supports sequential execution of configured BOD/EOD processes.

This means the next process is executed only after the previous process has completed.

For example:

```text
Process 1
   ↓
Completed Successfully
   ↓
Process 2
   ↓
Completed Successfully
   ↓
Process 3
   ↓
Completed Successfully
```

If a process fails, the scheduled process group is stopped and the failure is handled by the Process Manager.

This behavior is important when one process depends on the successful completion of an earlier process.

For example:

```text
Download File
      ↓
Process File
      ↓
Upload Data
```

If the download fails, the dependent processes should not continue.

---

## Process Status Management

During execution, the Process Manager maintains the current state of a process.

A process can move through states such as:

```text
Pending
   ↓
Initialized
   ↓
Running
   ↓
Successful
```

If execution fails:

```text
Pending
   ↓
Initialized
   ↓
Running
   ↓
Failed
```

The Process Manager stores process execution information in Redis so that the current state and execution details can be consumed by other components of the system.

The process status includes information such as start time, running time, status, and execution logs.

---

## Process Monitoring

In addition to BOD and EOD execution, `process_master.py` continuously monitors configured application processes.

The monitoring configuration is maintained separately in `process.json`.

For example:

```text
Process Monitor
      │
      ├── Docker Containers
      ├── Windows EXE Processes
      └── Other Configured Processes
```

The Process Manager collects runtime information such as:

* Process ID (PID)
* CPU usage
* Memory usage
* Disk I/O
* Process start time
* Process status

## Monitoring information is periodically written to Redis. The monitoring loop runs continuously and samples process information at regular intervals.

## Process Start, Stop and Restart

`process_master.py` also manages the lifecycle of configured infrastructure processes.

Depending on the configured process type, it can manage:

### Windows EXE Processes

Windows executable processes are started using the Windows process mechanism and tracked by the Process Manager.

### Linux / macOS EXE Processes

On Linux/macOS, executable processes are managed using `tmux` sessions.

### Docker Containers

Docker processes are managed using Docker container operations such as:

```text
Start
Stop
Restart
```

The exact behavior is controlled by the process configuration and its flags.

---

## Process Monitoring and Automatic Recovery

The Process Manager can detect when a configured process is no longer running.

The configured `restart_flag` determines what action should be taken.

### Restart Enabled

```text
Process Running
      ↓
Process Stops
      ↓
Monitor Detects Process Down
      ↓
Restart Process
      ↓
Process Running Again
```

### Restart Disabled

```text
Process Running
      ↓
Process Stops
      ↓
Monitor Detects Process Down
      ↓
Email Alert
      ↓
Operator Investigates
```

The process configuration defines `start_flag`, `stop_flag`, and `restart_flag` for monitored processes.

---

## Redis Integration

Redis is an important part of the Process Manager architecture.

It is used to store:

* Process execution status
* Process execution logs
* Process start time
* Process running time
* Process monitoring information
* Runtime process statistics

The Process Manager creates Redis keys for individual process status and monitoring information.
This allows the Process Manager to communicate process information to other parts of the system, including the UI.

---

## Failure Handling

The Process Manager handles failures during scheduled process execution.

When a configured BOD or EOD process fails:

```text
Process Starts
     ↓
Execution Fails
     ↓
Failure Recorded
     ↓
Process Status = Failed
     ↓
Failure Details Stored
     ↓
Failure Email Sent
     ↓
Scheduled Group Stops
```

The failure notification contains information about the process and the error details, allowing the operator to investigate the failed process.

---

## Email Alerts

Email notifications are used for operational alerts.

The Process Manager can send notifications when:

* A scheduled process fails
* A monitored process is not running

Email configuration is provided through the application configuration, including SMTP server, SMTP port, sender details, and recipient configuration.

---

## Manual Process Control

The Process Manager also provides mechanisms for manually controlling processes.

This allows an operator to perform actions outside the normal scheduled execution flow when required.

Manual operations can include:

* Running a process manually
* Starting configured processes
* Stopping configured processes
* Managing the process monitor

This is useful during development, testing, maintenance, and troubleshooting.

---

## Scheduler

The Process Manager uses a scheduler to execute configured processes at their defined times.

The scheduler is responsible for triggering the BOD and EOD execution flows according to the configured schedule.

Conceptually:

```text
Configured Schedule
       ↓
Scheduler
       ↓
BOD / EOD Process Group
       ↓
Sequential Execution
       ↓
Status & Logs
```

The schedule time and process configuration are maintained in `process.json`.

---

## Main Runtime Flow

When `process_master.py` starts, it initializes the required components and then enters its normal runtime operation.

The overall startup flow can be understood as:

```text
Start Process Manager
        ↓
Load Configuration
        ↓
Initialize Required Services
        ↓
Initialize Redis
        ↓
Validate Environment
        ↓
Load BOD / EOD Configuration
        ↓
Initialize Process Monitoring
        ↓
Configure Scheduler
        ↓
Start Scheduled Execution
        ↓
Monitor Processes
        ↓
Handle Alerts / Recovery
```

After initialization, the Process Manager remains active and performs scheduled execution and continuous monitoring.

---

## Why `process_master.py` Is Important

`process_master.py` is the **central execution layer** of the Process Manager.

Without it, the configured BOD and EOD scripts would need to be executed manually or through separate scheduling mechanisms.

It provides a single place for:

```text
Scheduling
    +
Execution
    +
Monitoring
    +
Process Management
    +
Redis Updates
    +
Failure Handling
    +
Alerts
```

This centralized approach makes the execution of operational processes consistent and easier to monitor.

---

## What Developers Should Change

In normal development, developers should **avoid modifying `process_master.py` when adding a new BOD or EOD process**.

For a new BOD/EOD activity, the normal approach is:

```text
Create / Update Script
        ↓
Place Script in utils/
        ↓
Add Process Configuration
        ↓
Set Schedule
        ↓
Set Parameters
        ↓
Enable Process
        ↓
Process Manager Executes It
```

The Process Manager itself should only be modified when the **common execution or management behavior** needs to change.

For example, changing how all processes are monitored or how all scheduled jobs are handled is a Process Manager-level change.

---

## What Developers Should Not Do

Do not add individual BOD or EOD business logic directly into `process_master.py`.

For example, avoid adding:

```text
Download NSE File Logic
Database Cleanup Logic
Bhavcopy Processing Logic
Checksum Logic
Instrument Master Logic
```

directly into the Process Manager.

These activities should remain as separate scripts under `utils/` and should be registered through the configuration.

This keeps the Process Manager generic and allows individual processes to be developed and maintained independently.

---

## Relationship With Other Project Components

The overall responsibility of the Process Manager can be summarized as follows:

| Component           | Responsibility                                                      |
| ------------------- | ------------------------------------------------------------------- |
| `process.json`      | Defines what processes should run and how they should be configured |
| `process_master.py` | Controls scheduling, execution, monitoring, and process management  |
| `utils/`            | Contains individual BOD and EOD process scripts                     |
| `lib/config.py`     | Provides configuration-related functionality                        |
| `lib/email.py`      | Provides email functionality                                        |
| `lib/logger.py`     | Provides logging functionality                                      |
| Redis               | Stores runtime process status, logs, and monitoring information     |
| UI                  | Displays process execution and monitoring information               |

---

## Key Takeaway

`process_master.py` should be treated as the **engine of the Process Manager**, not as a location for individual process logic.

The basic principle is:

```text
Configuration → process.json

Execution & Management → process_master.py

Business / Process Logic → utils/

Common Utilities → lib/

Runtime Status → Redis

Operator Visibility → UI / Alerts
```

When adding a new BOD or EOD process, the primary changes should normally be made to the **process script and its configuration**, rather than modifying the Process Manager itself.
