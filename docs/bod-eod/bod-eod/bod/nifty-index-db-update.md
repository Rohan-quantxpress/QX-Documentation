# Nifty Index Constituents Download and DB Update

## Script: `Nifty Index Constituents Download and DB Update`

---

## 1. Purpose

This script downloads the latest constituent lists for configured Nifty indices, prepares and validates the data, and replaces the contents of the MySQL `NiftyIndices` table with the latest processed data.

The currently configured indices are:

* Nifty Bank
* Nifty 50
* Nifty 100
* Nifty 200

### Overall Flow

```text
Nifty Index Files
       ↓
Download from Nifty
       ↓
Use Backup if Download Fails
       ↓
Read and Clean CSV
       ↓
Remove Duplicates
       ↓
Combine All Indices
       ↓
TRUNCATE NiftyIndices
       ↓
Insert Fresh Data
```

---

## 2. Configured Index Files

The script processes four configured CSV files:

| File                    | Index Name  |
| ----------------------- | ----------- |
| `ind_niftybanklist.csv` | `niftybank` |
| `ind_nifty50list.csv`   | `nifty50`   |
| `ind_nifty100list.csv`  | `nifty100`  |
| `ind_nifty200list.csv`  | `nifty200`  |

The index name is derived automatically from the filename.

For example:

```text
ind_nifty50list.csv
        ↓
nifty50
```

The resulting index name is stored in the `Indices` field.

---

## 3. Download Location

Downloaded files are stored in:

```text
/home/Blitz/BOD/QX.ProcessManager/utils/downloads
```

The directory is automatically created if it does not already exist.

For example:

```text
/home/Blitz/BOD/QX.ProcessManager/utils/downloads/ind_nifty50list.csv
```

---

## 4. Backup Mechanism

The script has a backup directory:

```text
/home/Blitz/BOD/QX.ProcessManager/utils/backup
```

For every index file, the script first attempts to download the latest version from the configured Nifty URL.

If the download fails, it checks the backup directory for a file with the same filename.

### Fallback Flow

```text
Download Latest File
        │
        ├── Success
        │     ↓
        │   Use Downloaded File
        │
        └── Failure
              ↓
        Check Backup Directory
              │
              ├── Backup Exists
              │      ↓
              │   Use Backup File
              │
              └── Backup Missing
                     ↓
                Processing Stops
```

This allows the process to continue using an existing backup file when the external download is temporarily unavailable.

---

## 5. CSV Data Processing

Each downloaded or backup CSV is read and prepared before being inserted into the database.

The processing includes:

1. Read the CSV file.
2. Normalize the required column names.
3. Add the corresponding index name.
4. Keep only the required fields.
5. Remove rows without an ISIN.
6. Remove unnecessary whitespace.
7. Remove duplicate `(ISIN, Indices)` combinations.

### Required Data

The processed dataset contains:

| Column           | Purpose                                   |
| ---------------- | ----------------------------------------- |
| `CompanyName`    | Company name                              |
| `Industry`       | Industry classification                   |
| `InstrumentName` | Security symbol/instrument name           |
| `Series`         | Security series                           |
| `ISIN`           | Security identification number            |
| `Indices`        | Nifty index to which the security belongs |

---

## 6. Column Normalization

The source Nifty files use column names that are normalized before database processing.

| Source Column  | Processed Column |
| -------------- | ---------------- |
| `Company Name` | `CompanyName`    |
| `Symbol`       | `InstrumentName` |
| `ISIN Code`    | `ISIN`           |

Other source columns that are not required by the process are excluded from the final dataset.

---

## 7. Data Cleaning

Before database insertion, the script performs basic data cleanup.

### Missing ISIN

Rows without an ISIN are removed.

### ISIN Formatting

Leading and trailing whitespace is removed from ISIN values.

### Instrument Name

Leading and trailing whitespace is removed from instrument names.

### Duplicate Records

Duplicate records are removed using:

```text
ISIN + Indices
```

This means the same security can belong to multiple indices, but the same security cannot be stored twice for the same index.

---

## 8. Combining Index Data

Each index file is processed independently.

The resulting datasets are then combined into a single dataset.

```text
Nifty Bank
     │
Nifty 50
     │
Nifty 100
     │
Nifty 200
     │
     ▼
Combined Dataset
     │
     ▼
Final Duplicate Check
```

A final duplicate check is performed using:

```text
ISIN + Indices
```

before the database update.

---

## 9. Database

The target MySQL table is:

```text
NiftyIndices
```

The script uses SQLAlchemy with the MySQL/PyMySQL connection.

Database connection details can be provided through command-line arguments or obtained from the application's global configuration.

---

## 10. Database Update Strategy

The script uses a **full replacement strategy**.

It does not incrementally update existing Nifty constituent records.

The database operation is:

```text
Existing NiftyIndices Data
          ↓
TRUNCATE TABLE NiftyIndices
          ↓
Insert Newly Processed Data
          ↓
Fresh Nifty Constituents
```

The table structure is retained, while all existing rows are removed before the latest data is inserted.

> **Important:** Because the table is truncated before insertion, the process should be executed only when the required source files or valid backups are available.

---

## 11. Database Insert

The final dataset is inserted into `NiftyIndices` using pandas `to_sql()`.

Records are inserted in batches of:

```text
1000 rows
```

The insertion also uses MySQL `INSERT IGNORE` as an additional duplicate-protection mechanism.

The primary duplicate prevention is already performed during DataFrame processing using:

```text
ISIN + Indices
```

---

## 12. Database Arguments

The script supports the following command-line arguments:

| Argument     | Required | Default                     | Description    |
| ------------ | -------- | --------------------------- | -------------- |
| `--host`     | No       | `127.0.0.1` / global config | MySQL host     |
| `--port`     | No       | `3306` / global config      | MySQL port     |
| `--user`     | No       | `root` / global config      | MySQL username |
| `--password` | No       | `root` / global config      | MySQL password |
| `--database` | No       | `Blitz` / global config     | Database name  |

The script obtains default values through:

```text
lib.global_config.get()
```

---

## 13. External Dependencies

The script depends on the availability of the configured Nifty index URLs.

It also requires:

* Network access to download the files
* Valid backup files when downloads are unavailable
* Readable CSV data
* MySQL database connectivity
* Appropriate database permissions
* Python packages required by the script, including:

  * `requests`
  * `pandas`
  * `SQLAlchemy`
  * `PyMySQL`

---

## 14. Failure Handling

### Download Failure

If an index file cannot be downloaded, the corresponding backup file is checked.

### Backup Missing

If both the download and backup file are unavailable, processing for that file fails and the overall process cannot complete successfully.

### Invalid or Unexpected CSV

If the downloaded file does not contain the expected columns, processing can fail during CSV preparation.

### Database Failure

If the database operation fails, the transaction does not complete successfully.

The truncate and insert operations are performed within the same SQLAlchemy transaction context.

---

## 15. Logging

The script logs important operational events, including:

* Download attempts
* Download failures
* Backup-file usage
* Number of rows prepared per index
* Total rows prepared
* Table truncation
* Insert operation
* Successfully processed indices

Example:

```text
nifty50: 50 rows ready
nifty100: 100 rows ready
nifty200: 200 rows ready
Total rows to insert: <count>
Truncating table NiftyIndices
Inserting <count> new rows
```

These logs can be used to verify the BOD execution.

---

## 16. Execution

The script can be executed using the default database configuration:

```bash
python <script_name>.py
```

To provide database details explicitly:

```bash
python <script_name>.py \
    --host 127.0.0.1 \
    --port 3306 \
    --user root \
    --password root \
    --database Blitz
```

---

## 17. Operational Verification

After execution, verify the final log for successful insertion.

The script reports:

```text
Successfully inserted <count> rows
```

It also reports the indices processed:

```text
Indices updated: niftybank, nifty50, nifty100, nifty200
```

### Database Verification

Check the total number of records:

```sql
SELECT COUNT(*) FROM NiftyIndices;
```

Check the available indices:

```sql
SELECT DISTINCT Indices
FROM NiftyIndices;
```

Expected index values are:

```text
niftybank
nifty50
nifty100
nifty200
```

---

## 18. BOD Operational Checklist

Before execution:

* Verify network connectivity to the Nifty data source.
* Verify the backup directory contains valid fallback files.
* Verify MySQL connectivity.
* Verify the required database configuration.
* Ensure there is no unexpected issue with the existing `NiftyIndices` data before a full replacement.

After execution:

* Check the script logs.
* Verify all four configured indices were processed.
* Check the total inserted row count.
* Verify the `NiftyIndices` table contains fresh data.
* Investigate any unexpected missing index or unusually low row count.

---

## 19. Key Takeaway

This script maintains the database representation of the configured Nifty index constituents.

Its complete operational flow is:

```text
Nifty Index Sources
        ↓
Download Latest CSV
        ↓
Fallback to Backup if Required
        ↓
Read CSV
        ↓
Normalize Columns
        ↓
Clean Data
        ↓
Remove Duplicate ISIN + Index
        ↓
Combine All Index Data
        ↓
TRUNCATE NiftyIndices
        ↓
Bulk Insert Fresh Data
        ↓
NiftyIndices Updated
```

The most important operational characteristic is the **full table replacement**. The existing `NiftyIndices` records are removed and replaced with the latest successfully processed constituent data for Nifty Bank, Nifty 50, Nifty 100, and Nifty 200.
