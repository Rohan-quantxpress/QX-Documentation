## `process.json` — Process Registration and Configuration

`process.json` is the **central configuration file** used by the Process Manager to define which processes belong to the BOD/EOD workflow and how they should be executed.

Instead of keeping process information and execution workflows hardcoded inside `process_master.py`, the Process Manager reads the configuration from this file and builds its execution plan dynamically.

### What `process.json` Controls

The file defines:

* Which processes are registered with the Process Manager
* Whether a process is active or inactive
* The execution order of BOD and EOD processes
* The location of each process
* Parameters required by a process
* BOD and EOD scheduling
* Processes that need continuous monitoring
* Process start, stop, and restart behavior

### Why `process.json` Is Important

A process must be registered in `process.json` to become part of the Process Manager workflow.

If a process is not registered:

* It will not be included in the BOD/EOD execution flow
* It will not be managed by the Process Manager
* It will not receive Process Manager status handling
* It will not be available through the corresponding process-management workflow

The Process Manager therefore follows this basic relationship:

```text
process.json
     ↓
Defines what should run
     ↓
process_master.py
     ↓
Executes and manages the configured processes
```

---

## Configuration Structure

At a high level, the configuration contains four major areas:

```text
process.json
│
├── Global Configuration
│   ├── Base Path
│   ├── Email Configuration
│   └── Redis Configuration
│
├── BOD Configuration
│   ├── Schedule
│   └── Process List
│
├── EOD Configuration
│   ├── Schedule
│   └── Process List
│
└── Process Monitoring
    └── Processes to Monitor
```

Each section has a specific responsibility within the Process Manager.

---

## 1. Global Configuration

The global configuration contains settings that are shared by the Process Manager and its processes.

### Base Path

The `base_path` provides the common location from which application and script paths can be resolved.

This allows process definitions to use a common base location instead of repeatedly specifying complete paths.

For example:

```text
${base_path}/script.py
```

The Process Manager resolves `${base_path}` using the configured value when the process is executed.

### Why Base Path Is Used

Using a common base path provides:

* Centralized path management
* Easier deployment configuration
* Less repetition in process definitions
* Easier movement of the application between environments

---

### Email Configuration

The email configuration provides the SMTP details used by the Process Manager for notifications.

Email notifications are used when important process or system events occur, such as:

* Process failures
* Monitoring-related alerts
* Critical operational issues

The email configuration should be maintained securely, particularly when authentication credentials are required.

---

### Redis Configuration

Redis provides the runtime communication and state-management layer used by the Process Manager.

The configuration contains the required Redis connection information.

Redis is used by the Process Manager for activities such as:

* Process status updates
* Runtime process information
* Monitoring information
* Communication between Process Manager components

This allows the Process Manager and UI to work with current process information without relying only on static configuration.

---

# 2. BOD Configuration

The BOD section defines the processes that are executed as part of the **Beginning of Day** workflow.

It contains:

* BOD schedule
* Registered BOD processes
* Execution order
* Process configuration
* Process parameters
* Active/inactive status

### BOD Schedule

The BOD schedule defines when the Process Manager starts the BOD workflow.

At the configured time, the Process Manager loads the registered BOD processes and begins execution according to their configured order.

```text
BOD Schedule
      ↓
Load BOD Processes
      ↓
Arrange Execution Order
      ↓
Execute Processes Sequentially
```

---

## BOD Process Registration

Each process registered under `bod_processes` represents one executable step in the BOD workflow.

The configuration provides the information required by Process Manager to identify and execute that process.

A typical process definition contains information such as:

```text
Process Number
Process Title
Process Type
Application Path
Running Type
Active Status
Parameters
```

### Execution Order

The process number determines the order in which BOD processes are executed.

For example:

```text
Process 1
   ↓
Process 2
   ↓
Process 3
   ↓
Process 4
```

This is important because BOD processes may depend on the successful completion of previous processes.

### Active / Inactive Processes

A process can be disabled through its configuration without removing its definition.

```text
Active
  ↓
Included in execution

Inactive
  ↓
Skipped by Process Manager
```

This is useful when a process needs to be temporarily disabled while keeping its configuration available for future use.

### Process Parameters

A process can have parameters defined in the configuration.

When the Process Manager executes the process, these parameters are passed to the application as required.

This allows the same Process Manager framework to execute different processes with different runtime configurations.

---

## BOD Execution Behavior

BOD processes are executed according to their configured order.

The general execution flow is:

```text
BOD Scheduler
      ↓
Start BOD Workflow
      ↓
Process 1
      ↓
Process 2
      ↓
Process 3
      ↓
...
      ↓
BOD Completed
```

If a required process fails, the workflow does not simply continue as if everything succeeded. The failure is handled by Process Manager and the appropriate failure notification/status handling is performed.

This helps prevent dependent processes from running when an earlier required step has failed.

---

# 3. EOD Configuration

The EOD section defines the processes executed as part of the **End of Day** workflow.

Similar to BOD, it contains:

* EOD schedule
* Registered EOD processes
* Execution order
* Process paths
* Parameters
* Active/inactive status

### EOD Schedule

The configured EOD schedule determines when the Process Manager starts the EOD workflow.

```text
EOD Schedule
      ↓
Load EOD Processes
      ↓
Execute in Configured Order
      ↓
EOD Completed
```

---

## EOD Workflow

EOD processes are generally used for end-of-day operational activities such as:

* Data backup
* Data cleanup
* Runtime/cache cleanup
* Log management
* End-of-day data processing
* Daily data ingestion

The exact activities depend on the processes registered in the configuration.

### Why EOD Order Matters

The order of EOD processes can be important because one process may depend on another.

For example:

```text
Backup
   ↓
Cleanup
   ↓
Final Processing
```

A cleanup operation should not remove required data before the required backup or preservation step has completed.

Therefore, when adding or modifying an EOD process, its position in the execution sequence must be considered carefully.

---

# 4. Process Monitoring Configuration

The monitoring section defines applications or services that the Process Manager should continuously monitor.

This extends Process Manager beyond simple scheduled execution.

```text
Process Manager
      │
      ├── BOD Scheduler
      ├── EOD Scheduler
      └── Process Monitor
```

### What Is Monitored

Depending on the configured process type, Process Manager can monitor information such as:

* Running status
* Process ID
* CPU usage
* Memory usage
* Disk I/O
* Docker container status

The monitoring information is made available through the Process Manager's runtime monitoring mechanism.

---

## Monitoring Controls

Monitoring definitions can specify whether a process is allowed to be:

* Automatically started
* Stopped
* Restarted

This gives the Process Manager control over continuously running applications in addition to scheduled BOD/EOD jobs.

### Monitoring Behavior

The monitoring system periodically checks configured processes.

```text
Check Process
      ↓
Is Process Running?
      │
   ┌──┴──┐
   │     │
  Yes    No
   │     │
Update   Handle
Status   Failure/Recovery
   │
   ↓
Redis
   ↓
UI / Monitoring
```

When a monitored process is unavailable, the configured recovery and alert behavior is applied.

---

# 5. How Process Manager Uses `process.json`

The complete relationship between configuration and execution can be understood as:

```text
                 process.json
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
         BOD          EOD      Monitoring
          │           │           │
          ↓           ↓           ↓
      Scheduler    Scheduler   Monitor
          │           │           │
          └───────────┼───────────┘
                      ↓
               process_master.py
                      ↓
             Execute / Manage
                      ↓
                    Redis
                      ↓
                  Process UI
```

`process.json` therefore acts as the **configuration layer**, while `process_master.py` acts as the **execution and management layer**.

---

# 6. Adding a New Process

When a new BOD or EOD script is created, creating the script alone is not enough.

The process must also be registered in `process.json`.

The general workflow is:

```text
Create Process Script
        ↓
Place Script in Project Structure
        ↓
Add Process Definition
        ↓
Set Application Path
        ↓
Set Execution Order
        ↓
Set Parameters if Required
        ↓
Set is_active
        ↓
Save Configuration
        ↓
Process Manager Loads Configuration
        ↓
Process Becomes Part of Workflow
```

Before adding a new process, verify:

* The application path is correct
* The process order is correct
* Required parameters are present
* `is_active` is correctly configured
* The process type is correct
* The process can run successfully independently

---

# 7. Configuration Change Impact

Changes made to `process.json` can directly affect the Process Manager workflow.

For example:

| Change                     | Effect                                                 |
| -------------------------- | ------------------------------------------------------ |
| Add a process              | Process becomes available to the configured workflow   |
| Remove a process           | Process is no longer part of that workflow             |
| Change process number      | Execution order changes                                |
| Set `is_active` to `false` | Process is skipped                                     |
| Change application path    | Process Manager runs the process from the new location |
| Change parameters          | Runtime arguments passed to the process change         |
| Change BOD/EOD schedule    | Workflow start time changes                            |
| Change monitoring settings | Monitoring/start/stop/restart behavior changes         |

Because of this, configuration changes should be reviewed before deployment.

---

# 8. Important Operational Rule

`process.json` should be treated as the **source of truth for process registration and workflow configuration**.

When adding a new process:

```text
New Script
   +
process.json Registration
   =
Managed Process
```

A script that exists in the project but is not registered in the configuration is not automatically included in the Process Manager workflow.

Similarly, changing the configuration can change how an existing process is executed or monitored.

---

## Key Takeaway

`process.json` defines **what the Process Manager should manage and how those processes are organized**.

`process_master.py` defines **how those configured processes are executed, monitored, and managed at runtime**.

```text
process.json
     │
     │ Defines
     ↓
Processes + Order + Schedule + Parameters + Monitoring
     │
     ↓
process_master.py
     │
     │ Executes & Manages
     ↓
BOD / EOD / Applications / Monitoring
```

For day-to-day operations, developers should generally **add or modify process configuration in `process.json` rather than adding process-specific workflow logic to `process_master.py`**.
