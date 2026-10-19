# NSE Span Downloader

## Script: `nse_span_downloader.py`

### 1. Purpose

`nse_span_downloader.py` is a **NSE SPAN file download and preparation script** used to obtain the SPAN file required by downstream risk and margin-related processes.

The script retrieves the available NSE F&O SPAN archive for a specified date, extracts the file, and prepares it locally using a consistent filename.

```
text id="z7q1xm"
NSE Archive
     ↓
Find Available SPAN File
     ↓
Download ZIP
     ↓
Extract ZIP
     ↓
Rename to nsccl.spn
     ↓
Downstream Processes
```

The script focuses on **acquiring and preparing the SPAN data**. It does not itself perform margin calculations.

---

## 2. Role in the BOD Workflow

SPAN data is reference information used by downstream processes involved in risk and margin-related operations.

The script therefore acts as a **data preparation step** before those processes consume the SPAN file.

```
text id="3u6x9d"
BOD
 ↓
Download SPAN File
 ↓
Prepare nsccl.spn
 ↓
Downstream Risk / Margin Processes
```

The exact position of this process in the BOD sequence should be based on the processes that consume `nsccl.spn`.

---

## 3. NSE Data Source

The script downloads SPAN files directly from the NSE archive.

### Archive Location

```
text id="q5r8vx"
https://nsearchives.nseindia.com/archives/nsccl/span/
```

The file is selected according to the requested date and the available SPAN update suffix.

---

## 4. SPAN File Naming

For a particular date, the script checks the following filenames:

```
text id="k6f2ws"
nsccl.YYYYMMDD.i5.zip
nsccl.YYYYMMDD.i4.zip
nsccl.YYYYMMDD.i3.zip
nsccl.YYYYMMDD.i2.zip
nsccl.YYYYMMDD.i1.zip
nsccl.YYYYMMDD.s.zip
```

The search is performed in this order:

```
text id="v3q8na"
i5
 ↓
i4
 ↓
i3
 ↓
i2
 ↓
i1
 ↓
s
```

The script stops when it finds the **first successfully downloadable file**.

This means it does not download all available SPAN versions.

---

## 5. SPAN Update Types

The suffix identifies the type/version of the SPAN file being searched for.

```
text id="j2x6cq"
i1 → i5
     ↓
Available SPAN updates

s
     ↓
Settlement SPAN file
```

The script gives priority to the higher-numbered `i` files before checking the settlement file.

The actual file available for a particular date depends on NSE publication and availability.

---

# 6. Date Handling

The script accepts an optional date through the `--date` argument.

The expected input format is:

```
text id="8zv5rb"
dd-MM-yyyy
```

Example:

```
text id="n7k4px"
15-01-2026
```

The script converts this date internally into the format required for the NSE archive filename:

```
text id="8h3w5a"
YYYYMMDD
```

For example:

```
text id="1c8r2n"
15-01-2026
     ↓
20260115
     ↓
nsccl.20260115.i5.zip
```

If no date is supplied, the script uses the **current system date**.

If an invalid date is provided, the script reports the invalid format and falls back to the current system date.

---

# 7. Download Location

Downloaded files are stored in the local:

```
text id="s2x6vg"
downloads/
```

The directory is automatically created if it does not already exist.

The ZIP file is temporarily stored there while it is being processed.

---

# 8. ZIP Extraction and File Preparation

The SPAN file is downloaded as a ZIP archive.

After a successful download, the script:

1. Saves the ZIP file.
2. Extracts its contents.
3. Removes the downloaded ZIP file.
4. Renames the extracted file to:

```
text id="w6q1mz"
nsccl.spn
```

The resulting flow is:

```
text id="0g8x4v"
nsccl.YYYYMMDD.iX.zip
          ↓
       Extract
          ↓
     nsccl.spn
```

Using a consistent filename makes it easier for downstream processes to locate the SPAN file without needing to know which dated archive was selected.

---

# 9. File Availability Logic

The script is designed to handle situations where a particular SPAN version is not available.

For example:

```
text id="9f4w2c"
i5 → Not Available
 ↓
i4 → Not Available
 ↓
i3 → Available
 ↓
Download i3
 ↓
Stop Searching
```

This allows the process to use the first available file according to its configured priority.

If none of the expected files are available, the script reports that no SPAN file could be downloaded for the requested date.

---

# 10. Runtime Dependencies

The process depends primarily on:

### NSE Archive

The NSE archive must be reachable and the expected SPAN file must be available.

Possible issues include:

* NSE archive unavailable
* File not yet published
* Requested date has no available file
* Expected filename is not present
* Network connectivity problems

### Local File System

The process requires:

* Permission to create the `downloads` directory
* Permission to write downloaded files
* Sufficient disk space
* Permission to extract files

---

# 11. Failure Handling

The script checks several types of download and file-system failures.

### File Not Found

If NSE returns `404` for a particular SPAN version, the script moves to the next suffix.

```
text id="c4n7xa"
i5 → 404
 ↓
i4 → 404
 ↓
i3 → Try
```

This continues until a file is successfully downloaded or all configured options have been checked.

### Connection Failure

If the connection to NSE fails, the script moves to the next available suffix.

### Timeout

A request timeout is treated as a failed attempt and the script continues checking the remaining suffixes.

### Invalid Response

The script checks the response content type and expects a ZIP-compatible response.

If the response does not appear to be a ZIP/octet-stream response, that URL is skipped.

### File-System Error

If the downloaded file cannot be saved, the process reports the error and continues with the next candidate.

---

# 12. What to Verify After Execution

After the process completes, verify:

* The requested SPAN date is correct
* An NSE SPAN archive was successfully downloaded
* The ZIP was extracted successfully
* `nsccl.spn` exists in the `downloads` directory
* The file can be accessed by downstream processes
* No download or extraction errors are present in the logs

Expected output:

```
text id="4v7m1c"
downloads/
└── nsccl.spn
```

---

# 13. Manual Execution

The script can also be executed manually for a specific date.

The date should be supplied in:

```
text id="6m2x9k"
dd-MM-yyyy
```

format.

This can be useful when:

* Re-running the process for a specific date
* Investigating a missing SPAN file
* Recreating the local SPAN file
* Recovering from a failed BOD execution

---

# 14. Operational Flow

The complete process can be summarized as:

```
text id="1y8q5z"
Start
  ↓
Determine Target Date
  ↓
Generate NSE SPAN Filename
  ↓
Try i5
  ↓
Try i4 if Required
  ↓
Try i3 if Required
  ↓
Try i2 if Required
  ↓
Try i1 if Required
  ↓
Try s if Required
  ↓
Download First Available File
  ↓
Extract ZIP
  ↓
Remove ZIP
  ↓
Rename Extracted File
  ↓
nsccl.spn
  ↓
Downstream Process
```

---

## Key Takeaway

`nse_span_downloader.py` is responsible for **obtaining and preparing the NSE F&O SPAN file required by downstream processes**.

Its main responsibilities are:

```
text id="7x3k9p"
Find Available NSE SPAN File
        ↓
Download
        ↓
Extract
        ↓
Standardize Filename
        ↓
Provide nsccl.spn
```

The most important operational dependencies are **NSE file availability, correct trading date, network connectivity, and local file-system access**.

The script does not perform margin calculations itself; it prepares the SPAN input file so that subsequent risk or margin-related processes can consume it.
