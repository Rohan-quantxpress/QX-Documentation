
### 1️⃣ Purpose and Business Intent

`process_monitor.py` is a **continuous runtime monitoring and alerting script**
responsible for tracking the health and resource usage of **critical system
processes and Docker containers** defined in a configuration file.

Its primary objectives are to:
- Monitor whether critical processes are **running or stopped**
- Capture **CPU, memory, disk I/O, PID, and start time**
- Persist time-series process metrics into **Redis**
- Trigger **email alerts** when a monitored process is not running

This script is designed to run **continuously in the background**
and acts as an **operational safety net** for BOD/EOD and long-running services.

> 🔒 If this script is not running,  
> **process failures may go undetected**, leading to silent system outages.

---

### 2️⃣ What Processes Are Monitored (Source of Truth)

The list of processes to monitor is defined in:

**Processes.json**


This file must contain a key:

```json
"process_monitor": [
  {
    "title": "Readable Name",
    "process": "actual_process_or_container_name"
  }
]

```

### Rules Enforced by the Script

The monitoring script applies **strict configuration validation rules** to ensure reliability and consistency.

### Mandatory Fields
- **title** → Required  
- **process** → Required  

If **either field is missing or invalid**, the script will:
- Log the configuration error
- **Terminate immediately**
- Skip further monitoring cycles

### Configuration Authority
- `Processes.json` acts as the **single source of truth**
- Only processes explicitly defined in `Processes.json` are monitored
- No hardcoded or implicit processes are allowed

This design ensures:
- Predictable behavior
- Controlled monitoring scope
- No accidental or unauthorized process tracking

---

## 3️⃣ Execution Context

The script operates as a **background monitoring thread**.

### Execution Characteristics
- Implemented using `threading.Thread`
- Runs in a **continuous loop**
- Sleeps for **5 seconds** between each monitoring cycle
- Designed for long-running execution

### Lifecycle Control
The monitoring thread is controlled using two methods:

- `start_monitoring()` → Starts the background monitoring loop
- `stop_monitoring()` → Gracefully stops the monitoring thread

### Intended Usage
The script is designed to run:
- Alongside the **Process Manager**
- As a **long-running service component**
- Without blocking the main application flow

---

## 4️⃣ What Is Actually Monitored

The script monitors **two execution environments**.  
This section describes the first one.

---

## 4.1 OS-Level Processes

The script monitors operating system–level processes using the **psutil** library.

### Metrics Collected
For each matched process, the following information is collected:

- **Process ID (PID)**
- **CPU usage (%)**
- **Memory usage (%)**
- **Disk I/O** (read + write bytes)
- **Process start time**
- **Process status** (running, sleeping, etc.)

### Process Matching Logic
- Uses **exact process name matching**
- Only processes whose names exactly match the `process` field in `Processes.json` are tracked
- Partial or fuzzy matching is **not supported**

This strict matching ensures:
- Accurate monitoring
- No false positives
- Clear correlation between configuration and runtime behavior

 


### 🔚 Summary

`process_monitor.py` provides continuous visibility into the health and
resource usage of critical system processes and Docker containers.
By persisting metrics in Redis and sending alerts on failures, it helps
prevent silent outages and enables proactive operational response.

Because the script depends on operating system access, Docker availability,
Redis connectivity, and SMTP infrastructure, it must run continuously and
be monitored itself to ensure ongoing effectiveness.
