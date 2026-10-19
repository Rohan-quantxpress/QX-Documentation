# Introduction

## Overview

The **Process Health Report Generator** is a Python-based health monitoring and reporting utility used to verify the operational status of processes, Docker containers, application endpoints, and server resources.

The tool reads a server-specific `process_config.json`, evaluates all configured processes and containers, performs optional HTTP health checks, collects CPU and memory information, evaluates process groups, and generates health reports.

The tool supports:

- Docker Engine monitoring
- Docker container monitoring
- EXE process monitoring
- Python process monitoring
- HTTP and port health checks
- CPU and memory monitoring
- Host CPU, memory, and disk monitoring
- Process group impact analysis
- Troubleshooting FAQ
- Markdown report generation
- Standalone HTML report generation
- Optional JSON status output

## Purpose

The main purpose of this tool is to provide a single health report showing whether the required QX Platform components are running correctly.

The tool automatically checks:

- Docker Engine
- Docker containers
- Native EXE processes
- Python processes
- HTTP endpoints
- CPU usage
- Memory usage
- Host resources
- Process groups

It also identifies down processes and provides configured troubleshooting information.

## Main Components

The monitoring solution consists primarily of:

```text
generate_report.py
process_config.json
```

### generate_report.py

The Python script responsible for:

- Reading configuration
- Detecting the operating system
- Checking processes
- Checking Docker
- Performing HTTP checks
- Collecting resource information
- Evaluating groups
- Generating reports

### process_config.json

The configuration file responsible for defining:

- Server information
- Processes
- Process types
- Critical processes
- Categories
- Docker matching rules
- HTTP checks
- Process groups
- Impact descriptions
- Troubleshooting FAQ

## Read-Only Behavior

The Process Health Report Generator is a **read-only health monitoring and reporting tool**.

It checks:

- Process state
- Container state
- HTTP availability
- CPU usage
- Memory usage
- Host resources
- Group impact

It does not:

- Start processes
- Stop processes
- Restart processes
- Modify monitored processes
- Modify monitored containers