# 1.4 Registering the New Script in `process.json`

After creating a new BOD script, the next step is to **register the script in `process.json`**.

The Process Manager uses this configuration to identify which processes should be executed as part of the BOD workflow.

A script placed in the project folder is **not automatically included in BOD**. It must be registered in the configuration with the correct process information and path.

---

## Why Registration Is Important

The registration connects the new script to the Process Manager.

The overall flow is:

```text
New Script
    ↓
Register in process.json
    ↓
Process Manager reads configuration
    ↓
Process appears in the process flow
    ↓
BOD executes the process
```

If the registration is missing or incorrect, the Process Manager cannot execute the new process correctly.

---

## Understanding Process Paths

The most important part when registering a new process is providing the **correct application path**.

The path tells Process Manager where the executable script is located.

For scripts stored in the project's `utils` folder, the configured `base_path` can be used instead of writing the complete directory path for every process.

Example:

```json
"application_path": "${base_path}\\download_nse_most_active_scripts.py"
```

This tells Process Manager to locate the script relative to the configured base path.

### Before Registering a Process

Confirm:

* The script exists in the expected folder
* The filename is correct
* The folder name is correct
* The configured path points to the actual script
* The script can be executed independently

A small path mistake can prevent the process from starting.

---

# Input File Locations

Some BOD processes depend on files downloaded or received before the process starts.

These file locations are normally provided through the process parameters.

For example:

```json
"parameter": {
    "source": "C:\\Users\\...\\downloads\\input\\*"
}
```

The configured location tells the script where to find its input files.

### Before Running BOD

Verify that:

* The input directory exists
* Required files are present
* The filename pattern is correct
* The process has access to the directory
* The expected files are available before the process starts

If the required input files are missing, the process may fail even though the script itself is working correctly.

---

# Output and Destination Locations

Some processes generate files or copy files to another location.

In such cases, the destination location is also provided through the process configuration.

Example:

```json
"destination": [
    "C:\\Users\\...\\BOD Project\\"
]
```

Before running the process, verify that the destination:

* Exists, or can be created by the script
* Is the intended output location
* Has sufficient access permissions
* Does not contain unrelated files that could be affected

---

# Path Rules

When registering a new process, follow the path format already used by the project.

### Recommended Checks

* Use the correct absolute location where required by the existing configuration
* Use the correct Windows path format
* Use escaped backslashes in JSON paths
* Do not introduce unnecessary spaces
* Make sure the filename matches exactly
* Make sure the referenced folder exists
* Keep input and output locations consistent

Example:

```text
C:\Users\...\QX.ProcessManager\utils
```

When written inside JSON, Windows backslashes need to be escaped:

```text
C:\\Users\\...\\QX.ProcessManager\\utils
```

---

# Registering the Process

Once the script and its required folders are ready, add the process definition to the appropriate BOD process list.

The registration should contain the information required by Process Manager, including:

* Process number
* Process title
* Process type
* Application path
* Running type
* Active status
* Required parameters

The process number should be selected according to the required BOD execution order.

For example:

```text
Existing Process 1
       ↓
Existing Process 2
       ↓
New Process
       ↓
Existing Process 4
```

The order should be selected based on the dependency of the new process on other BOD processes.

---

# Verifying the Registration

After saving `process.json`, verify the configuration before running BOD.

Check the following:

| Check              | Expected Result                     |
| ------------------ | ----------------------------------- |
| Process definition | Present in BOD process list         |
| Process title      | Correct                             |
| Process number     | Correct execution position          |
| Application path   | Points to the actual script         |
| Process type       | Configured as BOD                   |
| Active status      | Enabled when the process should run |
| Parameters         | Correct input/output locations      |
| Input files        | Available before execution          |

---

# Restart Process Manager

After changing `process.json`, restart the Process Manager so that the updated configuration is loaded.

```text
Update process.json
       ↓
Stop Process Manager
       ↓
Start process_master.py
       ↓
Configuration Loaded
```

Do not assume that an already-running Process Manager instance has automatically picked up configuration changes.

---

# Verify from the UI

After restarting Process Manager:

1. Open the Admin UI.
2. Navigate to the relevant BOD/process monitoring section.
3. Locate the newly registered process.
4. Verify the process name.
5. Verify its execution order.
6. Confirm that the process is active.

![AdminUI](https://raw.githubusercontent.com/priyanshu-quantxpress/MD-files/main/images/AdminUI.png)

> Admin UI showing the BOD process list.

If the new process does not appear as expected, check the configuration and restart Process Manager again after correcting it.

---

# Run and Verify BOD

Once the process is visible in the UI, run the BOD workflow.

```text
BOD Triggered
      ↓
New Process Reached
      ↓
Script Started
      ↓
Input Files Processed
      ↓
Output Generated
      ↓
Process Completed
```

Verify that:

* The process starts at the expected position
* The required input files are found
* The script completes successfully
* The expected output is generated
* The process status is updated correctly in the UI

![RunBOD\_EOD](https://raw.githubusercontent.com/priyanshu-quantxpress/MD-files/main/images/RunBOD_EOD.png)

> UI showing BOD/EOD execution and process status.

---

# If the Process Fails

If the newly registered process fails, check the issue in the following order:

```text
1. Check process status
        ↓
2. Check application path
        ↓
3. Check input files
        ↓
4. Check parameters
        ↓
5. Check output/destination path
        ↓
6. Review process logs
        ↓
7. Correct the configuration or script
        ↓
8. Restart Process Manager
        ↓
9. Run BOD again
```

### Common Registration Problems

| Problem                   | What to Check                                    |
| ------------------------- | ------------------------------------------------ |
| Process does not appear   | Process registration and Process Manager restart |
| Process does not start    | Application path                                 |
| Input file not found      | Source/input path and file availability          |
| Wrong process order       | Process number                                   |
| Process is skipped        | Active status                                    |
| Output not generated      | Destination path and script behavior             |
| Process fails immediately | Path, parameters, permissions, and logs          |

---

# Complete Registration Flow

The complete process for adding a new BOD script is:

```text
Create New Script
       ↓
Place Script in Correct Folder
       ↓
Configure Input / Output Locations
       ↓
Register Script in process.json
       ↓
Set Execution Order
       ↓
Set Process as Active
       ↓
Save Configuration
       ↓
Restart Process Manager
       ↓
Verify Process in UI
       ↓
Run BOD
       ↓
Verify Execution
       ↓
Verify Output
```

## Key Point

Adding a script to the project is only the **first step**.

For the Process Manager to execute it as part of BOD, the script must be correctly **registered and configured in `process.json`**.

The most important things to verify are:

**Script → Path → Parameters → Order → Active Status → UI → BOD Execution**

Following this sequence makes it easier to identify configuration problems before they affect the BOD workflow.
