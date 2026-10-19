# Architecture and Execution Flow

## High-Level Workflow

The Process Health Report Generator follows the workflow below:

```mermaid
flowchart TD
    A[Start] --> B[Read process_config.json]
    B --> C[Detect Operating System]
    C --> D[Evaluate Configured Processes]

    D --> E{Process Type}

    E -->|docker_engine| F[Check Docker Engine]
    E -->|docker| G[Check Docker Container]
    E -->|exe| H[Check Native EXE]
    E -->|python| I[Check Python Process]

    F --> J[HTTP Check]
    G --> J
    H --> J
    I --> J

    J --> K[Collect CPU / Memory]
    K --> L[Evaluate Process Groups]
    L --> M[Collect Host Resources]

    M --> N[Generate Markdown]
    N --> O[Generate HTML]

    O --> P{JSON Output Requested?}

    P -->|Yes| Q[Generate JSON]
    P -->|No| R[End]

    Q --> R
```

## Execution Stages

### 1. Load Configuration

The script reads:

```text
process_config.json
```

The configuration contains the processes, groups, HTTP checks, and FAQ definitions.

### 2. Detect Operating System

The script detects whether it is running on:

- Windows
- Linux

This is required because native process names can differ between platforms.

### 3. Evaluate Processes

Each configured process is evaluated according to its configured type.

```text
docker_engine → Docker Engine check
docker         → Docker container check
exe            → Native process check
python         → Python process check
```

### 4. HTTP Health Check

If an `http_check` is configured, the script attempts to determine the service port and performs an HTTP request.

### 5. Resource Collection

The script collects:

- CPU
- Memory
- Docker resource usage
- Host resource usage

### 6. Group Evaluation

Processes are grouped into logical application components.

The health of each group is calculated from the status of its members.

### 7. Report Generation

The tool generates:

```text
Markdown report
HTML report
Optional JSON status
```

## Complete Data Flow

```text
process_config.json
        |
        v
Read Configuration
        |
        v
Detect Operating System
        |
        v
Evaluate Processes
        |
        +--> Docker Engine
        |
        +--> Docker Container
        |
        +--> EXE Process
        |
        +--> Python Process
        |
        v
HTTP Health Check
        |
        v
CPU / Memory Collection
        |
        v
Evaluate Groups
        |
        v
Collect Host Resources
        |
        v
Generate Markdown
        |
        v
Generate HTML
        |
        v
Optional JSON
```