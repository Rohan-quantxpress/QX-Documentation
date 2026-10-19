# Process Manager — User & Developer Guide

## Introduction

The **Process Manager** is an automation and process-monitoring component used to manage the execution and availability of application processes across the system.

It automates **BOD (Beginning of Day)** and **EOD (End of Day)** activities by executing configured processes according to their defined schedules and execution order. It also continuously monitors configured application processes and can start, stop, or restart them based on their configuration.

The Process Manager supports different types of processes, including:

* Python scripts
* Windows executable (`.exe`) processes
* Docker containers

Process execution and configuration are controlled through the `Processes.json` configuration file. The configuration defines BOD processes, EOD processes, their execution schedules, parameters, active/inactive status, and process-monitoring settings.

## Purpose of This Documentation

This documentation explains how the Process Manager works and provides the information required to configure, operate, monitor, and troubleshoot automated processes.

It is intended for:

* New users working with the Process Manager
* Developers adding or modifying BOD/EOD processes
* Developers maintaining process configurations
* Support and operations teams monitoring running processes
* Users troubleshooting failed or stopped processes

## What This Documentation Covers

This guide covers the complete Process Manager workflow, including:

### BOD Process Management

* Understanding the BOD execution flow
* Adding a new BOD process
* Configuring a BOD process
* Defining execution time and order
* Passing parameters to BOD scripts
* Enabling or disabling a BOD process
* Verifying BOD execution
* Handling BOD process failures

### EOD Process Management

* Understanding the EOD execution flow
* Adding a new EOD process
* Configuring EOD processes
* Defining execution schedules
* Passing parameters to EOD scripts
* Enabling or disabling EOD processes
* Verifying EOD execution
* Troubleshooting EOD failures

### Process Monitoring

The Process Manager continuously monitors configured processes and records information such as:

* Process status
* Process ID (PID)
* CPU usage
* Memory usage
* Disk I/O
* Process start time
* Docker container status

Monitoring information is stored in Redis for use by other system components.

### Process Start, Stop & Restart

The Process Manager can manage application processes according to their configuration.

Depending on the process type, it can:

* Start processes
* Stop processes
* Restart processes
* Detect processes that are no longer running
* Automatically restart configured processes
* Send an alert when a process is unavailable

Windows executable processes are managed using Windows process handling, while Linux/macOS executable processes use `tmux`. Docker processes are managed through Docker commands.

## Configuration-Based Operation

The Process Manager uses `Processes.json` as its primary configuration file.

The configuration contains three major areas:

1. **BOD Processes**
2. **EOD Processes**
3. **Process Monitor**

Each process can define properties such as:

* Process number
* Process title
* Process type
* Schedule time
* Application path
* Execution mode
* Active/inactive status
* Parameters
* Start flag
* Stop flag
* Restart flag

For example, the configuration defines BOD processes with properties such as `no`, `title`, `type`, `schedule_time`, `application_path`, `running_type`, `is_active`, and `parameter`.

## Automated Execution Flow

The overall Process Manager workflow can be understood as:

```text
Process Manager
      │
      ├── BOD Scheduler
      │      └── Execute BOD Processes
      │
      ├── EOD Scheduler
      │      └── Execute EOD Processes
      │
      └── Process Monitor
             ├── Check Process Status
             ├── Collect Resource Statistics
             ├── Update Redis
             ├── Restart Process (if configured)
             └── Send Alert Email
```

During scheduled execution, processes are executed according to their configuration. If a scheduled process fails, the Process Manager records the failure and can send a failure notification.

## Who Should Read This Documentation?

### New Users

Start with the **Project Structure and Configuration** section to understand where Process Manager files and configuration are located.

### Developers

Read the **Adding a New BOD/EOD Process** section before adding a new script to the automated workflow.

### Operations & Support

Refer to the **Process Monitoring** and **Troubleshooting** sections when a process is stopped, fails, or requires a restart.

## Before Making Changes

Before adding or modifying a process, understand the relationship between:

```text
Processes.json
      ↓
Process Manager
      ↓
Scheduler
      ↓
Python / EXE / Docker Process
      ↓
Redis Status & Monitoring
      ↓
UI / Alerts
```

A process should not be added only by placing its script in a folder. The process must also be correctly configured in `Processes.json` so that the Process Manager knows **what to execute, when to execute it, which parameters to pass, and how the process should be handled**.

## Where to Start

If you are new to the Process Manager, follow the documentation in this order:

1. **Project Structure**
2. **Process Manager Configuration**
3. **BOD Process**
4. **EOD Process**
5. **Process Monitoring**
6. **Adding a New Process**
7. **Execution & Verification**
8. **Troubleshooting**

This ensures that you understand the Process Manager architecture and configuration before making changes to the automated process flow.
