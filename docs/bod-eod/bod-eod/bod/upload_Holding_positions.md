## Script: `upolad_holding_position_db.py`

---

## 1. Purpose and Business Intent

`upolad_holding_position_db.py` is a **BOD holdings ingestion and database update script** responsible for loading client holding information and enriching it with instrument master data before storing it in the `Holdings` table.

The current execution flow processes the **holding file**. Position-file processing is present in the script but is currently disabled in `main()`.

### Primary Responsibilities

* Load the daily client holding file
* Support CSV, Excel, and pipe-separated holding formats
* Normalize holding data
* Load the instrument master
* Match holdings with instruments using ISIN
* Search NSE instruments first and BSE instruments when an ISIN is not found in NSE
* Remove duplicate and invalid holding records
* Create the `Holdings` table if it does not exist
* Replace existing holdings with the latest valid holding data
* Store instrument and exchange information along with the holding

> **Important:** The script performs a **full overwrite of the `Holdings` table**. Existing holding records are deleted before the new records are inserted.

---

## 2. Current Execution Scope

Although the script contains logic for both holdings and positions, only the holdings flow is currently active.

### Currently Active

```text
Holding File
     ↓
Instrument Master
     ↓
ISIN Matching
     ↓
Data Cleaning
     ↓
Holdings Table
```

### Currently Disabled

The position-processing section is commented out in `main()`.

Therefore, the current execution does **not** load the position file into `InstrumentStatistics`.

---

## 3. Input Data Sources

### 3.1 Holding File

The default daily holding file is:

```text
downloads/Bull8_HOLDINGYYYYMMDD.CSV
```

For example:

```text
downloads/Bull8_HOLDING20260917.CSV
```

The filename is generated using the current system date.

The script supports:

* `.CSV`
* `.XLS`
* `.XLSX`
* Pipe-separated text data without a header

### Pipe-Separated Format

Example:

```text
CLIENT001|INE123456789|100|...|125.50
CLIENT002|INE987654321|250|...|210.25
```

The script extracts the required holding fields from the pipe-separated record.

### Header-Based Format

For CSV files, the script expects the relevant fields to correspond to:

| Source Column | Internal Field  |
| ------------- | --------------- |
| `ACCOUNT ID`  | `UserID`        |
| `ISIN`        | `isin`          |
| `QUANTITY`    | `quantity`      |
| `BUYAVGPRICE` | `buy_avg_price` |

---

## 4. Instrument Master

The holding data is enriched using the instrument master CSV.

The default instrument master location is:

```text
/home/Blitz/BOD/QX.ProcessManager/InstrumentsRegisterInRedis/Instruments/instruments.csv
```

A custom instrument file can also be supplied through the command line.

The instrument master provides information such as:

* ISIN
* Instrument Name
* Ticker
* Instrument ID
* Exchange

The script normalizes the instrument master before using it for matching.

---

## 5. Instrument Matching

Holdings are matched with instruments using the **ISIN** value.

The matching process gives priority to NSE instruments.

### Matching Flow

```text
Holding ISIN
     │
     ▼
Search NSE Instruments
     │
     ├── Found ──► Use NSE Instrument
     │
     └── Not Found
              │
              ▼
       Search BSE Instruments
              │
              ├── Found ──► Use BSE Instrument
              │
              └── Not Found ──► Skip Holding
```

This allows an ISIN that is not available in the NSE segment to be searched in the BSE segment.

Holdings whose ISIN cannot be matched to an instrument are logged and excluded from the database update.

---

## 6. Data Cleaning

Before the data is inserted into the database, the script performs several cleanup operations.

### Duplicate Holdings

Duplicate records based on:

```text
UserID + ISIN
```

are removed.

Only one record for the same user and ISIN is retained.

### Invalid Instrument Names

Records where the instrument name contains `#` are removed.

### Instrument ID

Instrument IDs are normalized to remove unwanted decimal formatting such as:

```text
12345.0
```

which becomes:

```text
12345
```

### ISIN

ISIN values are trimmed of leading and trailing whitespace before matching and storage.

---

## 7. Exchange Segment Mapping

The exchange information from the instrument master is converted into the corresponding exchange segment value before being stored in `Holdings`.

| Exchange | Exchange Segment |
| -------- | ---------------: |
| NSE      |              `1` |
| BSE      |             `11` |

If the exchange cannot be identified as NSE or BSE, the exchange segment remains empty.

---

## 8. Holdings Database Table

The script creates the `Holdings` table if it does not already exist.

The current schema is:

```sql
Holdings (
    UserID VARCHAR(50) NOT NULL,
    ISIN VARCHAR(20) NOT NULL,
    InstrumentName VARCHAR(100),
    Ticker VARCHAR(50),
    InstrumentId VARCHAR(50),
    ExchangeSegment VARCHAR(10),
    Quantity DOUBLE,
    BuyAvgPrice DOUBLE,
    PRIMARY KEY (UserID, ISIN)
)
```

### Key Fields

| Field             | Purpose                         |
| ----------------- | ------------------------------- |
| `UserID`          | Identifies the client/user      |
| `ISIN`            | Identifies the security         |
| `InstrumentName`  | Instrument name from the master |
| `Ticker`          | Instrument ticker               |
| `InstrumentId`    | Internal instrument identifier  |
| `ExchangeSegment` | NSE/BSE segment identifier      |
| `Quantity`        | Client holding quantity         |
| `BuyAvgPrice`     | Average purchase price          |

The primary key is:

```text
(UserID, ISIN)
```

---

## 9. Database Update Strategy

The script does **not** perform an upsert.

Instead, it uses a **full overwrite strategy**.

The process is:

```text
Create Holdings table if required
          ↓
Delete existing Holdings records
          ↓
Prepare cleaned holding records
          ↓
Bulk insert new records
          ↓
Holdings table contains latest input data
```

The existing records are deleted using:

```sql
DELETE FROM Holdings
```

The new records are then inserted in bulk.

> **Important:** Because existing records are deleted before insertion, this script should be executed only when the latest holding file is available and validated.

---

## 10. Execution Flow

The active BOD holding flow is:

1. Load the instrument master.
2. Validate that the instrument master was loaded successfully.
3. Load today's holding file.
4. Detect the holding file format.
5. Normalize holding columns.
6. Separate NSE and BSE instruments.
7. Match holding ISINs with NSE instruments.
8. Search BSE for ISINs not found in NSE.
9. Remove invalid and duplicate records.
10. Create the `Holdings` table if required.
11. Delete existing holdings.
12. Insert the cleaned holding records.
13. Log the processing result.

### Overall Flow

```text
Daily Holding File
        │
        ▼
Load Holding Data
        │
        ▼
Load Instrument Master
        │
        ▼
Match ISIN
   ┌────┴────┐
   ▼         ▼
  NSE       BSE
   └────┬────┘
        ▼
Clean & Deduplicate
        │
        ▼
Delete Existing Holdings
        │
        ▼
Insert Latest Holdings
        │
        ▼
BOD Holdings Updated
```

---

## 11. Command-Line Arguments

The script supports the following arguments:

| Argument            | Required | Default                             | Description            |
| ------------------- | -------- | ----------------------------------- | ---------------------- |
| `--holding_file`    | No       | Today's `Bull8_HOLDINGYYYYMMDD.CSV` | Holding file path      |
| `--position_file`   | No       | Today's `Bull8_POSYYYYMMDD.CSV`     | Position file path     |
| `--instrument_file` | No       | Configured instrument master path   | Instrument master file |
| `--host`            | No       | Global config / `127.0.0.1`         | Database host          |
| `--port`            | No       | Global config / `3306`              | Database port          |
| `--user`            | No       | Global config / `root`              | Database username      |
| `--password`        | No       | Global config / `root`              | Database password      |
| `--database`        | No       | Global config / `Blitz`             | Database name          |

### Important Implementation Note

Although `--holding_file` is defined as a command-line argument, the current `load_holding_file()` implementation constructs the holding filename directly from today's date:

```text
downloads/Bull8_HOLDINGYYYYMMDD.CSV
```

Therefore, the current active holding-processing flow does not actually use a custom `--holding_file` value.

Similarly, `--position_file` is currently unused because position processing is disabled.

---

## 12. File Dependencies

### Holding File

The expected daily holding file must be available before execution:

```text
downloads/Bull8_HOLDINGYYYYMMDD.CSV
```

If it is missing, the script logs an error and stops the holding-processing flow.

### Instrument Master

The instrument master must exist at the configured location.

If the instrument master is missing, the script exits before processing holdings.

### Database

The configured MySQL database must be reachable and accessible using the supplied credentials.

---

## 13. Handling Missing Instruments

Not every holding is guaranteed to match an instrument master entry.

When an ISIN cannot be found in either NSE or BSE instrument data, the holding is excluded from the final dataset.

The script logs the missing ISINs so that operations teams can investigate the discrepancy.

Example:

```text
Skipping 2 holdings for missing ISINs
Missing ISIN: INE123456789
Missing ISIN: INE987654321
```

This prevents unmatched holdings from being inserted without the required instrument information.

---

## 14. Processing Statistics

During execution, the script logs information such as:

* Number of instruments loaded
* Number of holding records loaded
* Number of NSE instruments
* Number of BSE instruments
* Number of valid holding rows
* Number of invalid holding rows
* Number of records inserted

Example:

```text
Instrument master loaded with 50000 records
Holding data loaded with 1250 records
NSE instruments: 35000
BSE instruments: 15000
Valid holding rows: 1235
Invalid holding rows: 15
Inserted 1235 records into Holdings
Holdings table fully overwritten successfully
```

These logs can be used as part of BOD verification.

---

## 15. Failure Scenarios

### Instrument Master Missing

If the instrument master file does not exist, the script exits with an error.

### Holding File Missing

If today's holding file is unavailable, no holding data is processed.

### Unsupported File Format

The holding loader supports CSV and Excel files. Unsupported formats cause the script to terminate.

### Missing ISIN Match

Holdings that cannot be matched against NSE or BSE instruments are skipped.

### Database Failure

If the database connection or insertion operation fails, the transaction can fail and the error is logged.

### Empty Valid Dataset

If no valid holdings remain after matching and filtering, the script does not update the `Holdings` table.

---

## 16. Operational Verification

After the script completes successfully, verify the log for:

```text
Holding parser completed successfully
```

Also verify:

* The expected holding file was loaded.
* The instrument master was loaded.
* Valid holding count is reasonable.
* Missing ISIN count is investigated.
* The number of inserted records is as expected.
* The `Holdings` table contains the latest records.

Example database check:

```sql
SELECT COUNT(*) FROM Holdings;
```

To inspect client holdings:

```sql
SELECT
    UserID,
    ISIN,
    InstrumentName,
    Ticker,
    InstrumentId,
    ExchangeSegment,
    Quantity,
    BuyAvgPrice
FROM Holdings
LIMIT 20;
```

---

## 17. Important Operational Consideration

This script uses a **replace-all approach** rather than incremental updates.

The sequence is:

```text
Existing Holdings
       ↓
DELETE FROM Holdings
       ↓
Insert Latest Holdings
```

Therefore, the latest holding file should be available and validated before running the script.

A successful script execution means that the `Holdings` table was populated from the latest valid holding data processed by the script; it does not mean that every row from the source file was accepted, because duplicate, invalid, or unmatched records can be excluded.

---

## 18. Position Processing — Current Status

The script contains a separate position-processing implementation that can load position data into the `InstrumentStatistics` table.

However, the call to this processing flow is currently commented out in `main()`.

Therefore:

> **The current production execution of this script should be considered a Holdings loader only.**

The position-related code should not be documented as an active BOD step until it is enabled in `main()`.

---

## 19. Key Takeaway

`upolad_holding_position_db.py` currently performs **BOD holdings ingestion and replacement**.

Its active responsibility is:

```text
Daily Holding File
       ↓
Load & Normalize
       ↓
Instrument Master
       ↓
NSE ISIN Match
       ↓
BSE ISIN Match for Missing NSE Records
       ↓
Remove Invalid / Duplicate Records
       ↓
Delete Existing Holdings
       ↓
Insert Latest Valid Holdings
       ↓
Holdings Table Updated
```

The most important operational characteristic is that the script **fully replaces the existing `Holdings` table data** rather than performing an incremental upsert.

The position-processing functionality exists in the source code but is currently disabled and therefore is not part of the active execution flow.
