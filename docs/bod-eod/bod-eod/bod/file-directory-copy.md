## File and Directory Copy Script

### Script: `copy_files_src_dst.py`

`copy_files_src_dst.py` is a **file and directory management utility** used within the BOD workflow.

Its primary responsibility is to **copy required files or directories from a configured source location to one or more destination locations**.

The script can handle:

* Individual files
* Complete directories
* Multiple source items using wildcard patterns
* Multiple destination directories

This makes it useful when the same BOD input files need to be made available in different locations for downstream processes.

---

## Purpose in the BOD Workflow

The script is generally used as an **early file-preparation step** in the BOD process.

```text
Source Files
     ↓
copy_files_src_dst.py
     ↓
Required Destination Locations
     ↓
Downstream BOD Processes
```

The script does not perform the actual business processing of the files. Its responsibility is to **make the required files or directories available at the configured destinations**.

---

## How the Script Works

The script receives two main inputs:

```text
Source
  +
Destination(s)
```

It then:

1. Checks whether the source exists.
2. Creates the destination directory when required.
3. Determines whether the source is a file or directory.
4. Copies the source to the configured destination.
5. Supports copying to multiple destinations.
6. Reports the result through the Process Manager logs.

For example:

```text
Source
C:\BOD\Downloads
        ↓
        ├── Destination 1
        ├── Destination 2
        └── Destination 3
```

This allows one source location to be distributed to multiple locations required by the BOD workflow.

---

## File Copy

When the source is a single file, the script copies that file to the destination while retaining its original filename.

```text
Source File
    ↓
Destination Directory
    ↓
Same Filename
```

---

## Directory Copy

When the source is a directory, the script copies the directory and its contents to the destination.

If the destination directory already exists, its contents can be merged/overwritten as part of the copy operation.

Therefore, the configured destination should always be verified before running the process.

---

## Multiple Files Using Wildcards

The script also supports wildcard-based source paths.

For example:

```text
C:\BOD\Downloads\*
```

This allows multiple matching files or directories to be processed without specifying every item individually.

The matching items are then copied to the configured destination locations.

```text
Source Pattern
      ↓
Find Matching Items
      ↓
Copy Each Item
      ↓
Destination 1
      ↓
Destination 2
```

If no items match the configured pattern, the process reports an error and exits with a failure status.

---

## Multiple Destinations

The script supports one or more destination locations.

This is useful when the same files are required by different applications or processing stages.

Example:

```text
Source
  │
  ├──→ Application A
  │
  ├──→ Application B
  │
  └──→ Application C
```

All configured destinations must be checked to ensure that they point to the correct locations.

---

## Error Handling

The script reports common operational problems through the Process Manager logger.

Examples include:

* Source path does not exist
* Source is neither a valid file nor directory
* Destination cannot be accessed
* Permission is denied
* Source and destination refer to the same file
* No files match the configured wildcard
* Unexpected copy errors

If any required copy operation fails, the script reports the failure so that the BOD workflow can identify the problem.

---

## Configuration in `process.json`

The script is registered in `process.json` with its source and destination information provided as process parameters.

Conceptually:

```text
process.json
     ↓
Source Location
     +
Destination Location(s)
     ↓
copy_files_src_dst.py
     ↓
Copy Operation
```

When configuring this process, pay particular attention to:

* Source path
* Destination path
* Wildcard pattern, if used
* Destination count
* Execution order within BOD

Incorrect paths are one of the most common causes of failure for this type of process.

---

## BOD Role

The script is typically positioned where files need to be prepared before another BOD process consumes them.

For example:

```text
Download / Receive Files
        ↓
copy_files_src_dst.py
        ↓
Prepare Required Locations
        ↓
Process Market / Reference Data
        ↓
Load Data
```

The exact position depends on the BOD configuration and the dependencies of the other processes.

---

## What to Verify Before Running

Before adding or modifying this process, verify:

* Source location exists
* Required files are available
* Wildcard pattern is correct, if applicable
* Destination locations are correct
* Destination directories can be created or accessed
* The process is registered correctly in `process.json`
* The process has the correct execution order

---

## Key Takeaway

`copy_files_src_dst.py` is a **file preparation and distribution utility** within the BOD workflow.

Its role is simple:

```text
Find Required Files
       ↓
Copy Files / Directories
       ↓
Make Them Available at Required Locations
       ↓
Allow Downstream Processes to Continue
```

It acts as a supporting process for BOD by ensuring that required files are copied to the locations expected by subsequent processes.
