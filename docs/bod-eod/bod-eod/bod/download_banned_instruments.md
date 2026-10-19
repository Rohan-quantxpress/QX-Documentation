# Download Banned Instruments

## Script: `download_banned_instruements.py`

### Purpose

`download_banned_instruements.py` is a BOD script used to **retrieve the NSE F&O security-ban information and make it available to the trading system**.

The script downloads the ban list from NSE, saves a local copy of the downloaded data, and inserts the banned instruments into the `BannedInstruments` MySQL table.

```text
NSE
 ↓
fo_secban.csv
 ↓
Download & Read Data
 ↓
Save Local CSV
 ↓
Insert into MySQL
 ↓
BannedInstruments
```

This allows the trading system to maintain an internal record of instruments identified in the NSE ban list.

---

### Data Source

The script downloads the following file from the NSE archive:

```text
fo_secban.csv
```

The source is the NSE archive endpoint configured in the script.

The downloaded file contains the information used by the script to identify banned symbols.

The relevant data consists of:

| Field    | Purpose                                      |
| -------- | -------------------------------------------- |
| `Index`  | Row/index information from the source file   |
| `Symbol` | Instrument symbol identified in the ban list |

---

### How the Script Works

The script follows a simple data-ingestion flow:

```text
Start
  ↓
Request NSE Ban List
  ↓
Download CSV
  ↓
Read CSV Data
  ↓
Save downloads/fo_secban.csv
  ↓
Connect to MySQL
  ↓
Insert Banned Instruments
  ↓
Complete
```

The downloaded CSV is first stored locally and then the instrument information is inserted into the database.

---

### Database Usage

The script inserts the banned instrument information into:

```text
BannedInstruments
```

The database record includes:

* Symbol
* Ban reason
* Banned status
* User-created status

The script marks the downloaded exchange instruments as exchange-banned records.

---

### Configuration

The MySQL connection details can be supplied through command-line parameters or obtained from the application's configuration.

The supported connection information includes:

* Host
* Port
* Username
* Password
* Database

This allows the same script to be executed against the configured database environment without changing the script itself.

---

### Role in BOD

This script is part of the **reference/risk-data preparation stage** of BOD.

```text
BOD
 ↓
Download NSE Ban List
 ↓
Update Internal Database
 ↓
Trading System Has Current Ban Information
```

Its position in the BOD workflow should be considered when other processes depend on the `BannedInstruments` table.

---

### Dependencies

The script requires:

* Network access to the NSE endpoint
* A reachable MySQL server
* The configured database
* The `BannedInstruments` table
* Appropriate database permissions

If the NSE download or database operation fails, the BOD process may not complete as expected.

---

### What to Verify

Before troubleshooting this process, check:

* NSE endpoint is reachable
* Internet/network connectivity is available
* MySQL is running
* Database connection details are correct
* `BannedInstruments` exists
* Required database permissions are available
* `downloads` directory is available
* The downloaded `fo_secban.csv` contains the expected data

---

### Key Takeaway

`download_banned_instruements.py` connects the **NSE exchange ban information** with the internal trading database.

```text
NSE Ban List
     ↓
Local CSV
     ↓
MySQL
     ↓
BannedInstruments
```

It is therefore an important BOD reference-data process for keeping the internal banned-instrument information synchronized with the downloaded NSE data.

---

# Script: `download_bse_contract_master.py`

## Purpose

`download_bse_contract_master.py` is a BOD **BSE reference-data preparation script**.

Its purpose is to download the required BSE contract/reference files, extract the required files, and prepare them with standardized filenames for use by downstream BOD processes.

The script focuses on **file preparation** rather than loading the data into the database.

```text
BSE
 ↓
Download Reference Files
 ↓
Extract Required Files
 ↓
Prepare Standard Filenames
 ↓
Downstream BOD Processes
```

---

## BSE Reference Files

The process works with BSE reference files required by downstream applications.

The configured sources include:

### SCRIP Master

The SCRIP master provides BSE instrument reference information.

The downloaded archive contains a dated SCRIP file.

```text
SCRIP_<ddMMyy>.TXT
```

The process prepares the required file for downstream use.

### Equity Details

The BSE equity details file contains equity reference information.

The downloaded file follows a date-based naming convention such as:

```text
EQD_CO<ddMMyy>.csv
```

The process prepares the required file for downstream consumers.

---

## How the Script Works

At a high level, the workflow is:

```text
Start
  ↓
Download BSE Reference Archives
  ↓
Validate / Extract Archives
  ↓
Locate Required Dated Files
  ↓
Prepare Standardized Files
  ↓
Downstream BOD Processes
```

The purpose of standardizing the output filenames is to prevent downstream processes from having to handle changing date-based filenames.

---

## Why Standardized Filenames Are Important

BSE reference files may contain dates in their filenames.

For example:

```text
SCRIP_160926.TXT
EQD_CO160926.csv
```

Downstream applications can be simplified when the required files are prepared using predictable names.

Conceptually:

```text
BSE Dated File
     ↓
BOD Preparation
     ↓
Standard File
     ↓
Downstream Process
```

This separates the external exchange naming convention from the naming expected by internal processes.

---

## Date Dependency

The process depends on the date associated with the BSE files.

The expected reference files may correspond to the previous day when the process is executed.

Therefore, the process is dependent on **BSE file publication timing and availability**.

Before troubleshooting a missing file, verify:

* The expected BSE file has been published
* The expected date is correct
* The downloaded archive is valid
* The required file exists inside the archive

This is particularly important around weekends and exchange holidays.

---

## Execution Context

The script can be executed:

* Automatically as part of BOD
* Manually when file preparation needs to be performed again

It is primarily responsible for external file download and preparation.

It does not represent the database-loading stage of the BOD workflow.

---

## Failure Scenarios

### BSE File Cannot Be Downloaded

If the BSE source cannot be reached or the download fails, the required reference files cannot be prepared.

Check:

* Network connectivity
* BSE availability
* Source URL
* Downloaded file

---

### Invalid Downloaded Archive

If a downloaded file is not a valid archive, extraction cannot be completed.

Check the downloaded archive before continuing with downstream processing.

---

### Required Dated File Is Missing

The archive may be available while the expected date-specific file is missing.

For example:

```text
Expected:
SCRIP_<expected-date>.TXT

Actual:
File not available
```

In this situation, downstream processes may not find the required standardized file.

Verify the publication date and contents of the downloaded archive.

---

## What to Verify

Before considering this process successful, verify:

* BSE files were downloaded
* Downloaded archives are valid
* Required dated files are present
* The expected output files were prepared
* Output files are available in the expected location
* Downstream BOD processes can access the files

---

## Key Takeaway

`download_bse_contract_master.py` acts as the **BSE reference-file preparation stage** of BOD.

```text
BSE
 ↓
Reference Files
 ↓
Download
 ↓
Extract
 ↓
Standardize
 ↓
Prepared BSE Files
 ↓
Downstream BOD Processes
```

Its main responsibility is to ensure that the BSE reference files required by subsequent processes are downloaded and prepared in a predictable format and location.

The process is dependent on **BSE file availability, publication timing, archive contents, and expected filename conventions**.
