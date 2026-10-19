## Script: `upload_cash_margin_db.py`

---

## 1. Purpose and Business Intent

`upload_cash_margin_db.py` is a **BOD RMS data update script** responsible for reading the daily cash margin file and updating the corresponding `CashMargin` values in the `RMSEntities` database table.

The script connects the daily cash margin information with existing RMS entities by converting each client code into the RMS entity format and updating the matching database record.

### Primary Responsibilities

* Locate the daily cash margin file
* Read cash margin information
* Support multiple input file formats
* Normalize client codes
* Generate the corresponding RMS entity code
* Update the `CashMargin` value in `RMSEntities`
* Report the number of RMS entities successfully updated

> **Important:** This script directly updates cash margin values used by RMS-related processing. Its execution should therefore be verified as part of the BOD RMS setup.

---

## 2. Input File

The script automatically determines the input file based on the current system date.

### File Naming Convention

```text
Bull8_DEPMYYYYMMDD.CSV
```

For example, if the current date is `17 September 2026`:

```text
Bull8_DEPM20260917.CSV
```

The file is expected to be located inside:

```text
downloads/
```

The effective input path is:

```text
<project-root>/downloads/Bull8_DEPMYYYYMMDD.CSV
```

The script does not currently provide a command-line option to specify a different input filename.

---

## 3. Supported Input Formats

The script supports two cash margin file formats.

### 3.1 Pipe-Separated Format

A file without a CSV header can contain records in the following format:

```text
CLIENT001|100000
CLIENT002|250000
CLIENT003|75000
```

The first value is treated as the client code and the second value as the cash margin.

The script converts the client code to uppercase and adds the RMS suffix.

For example:

```text
CLIENT001
```

becomes:

```text
CLIENT001|ALL|ALL|ALL
```

---

### 3.2 Header-Based CSV Format

The script can also process a CSV file containing column headers.

It identifies:

* The client column by searching for a column name containing `client`
* The cash margin column by searching for a column name containing `cash`

For example:

```csv
ClientCode,CashMargin
CLIENT001,100000
CLIENT002,250000
```

The actual column names do not have to be exactly `ClientCode` and `CashMargin`; the script searches the header names for the corresponding keywords.

---

## 4. RMS Entity Code Generation

The client code from the input file is normalized by:

1. Removing leading/trailing whitespace
2. Converting it to uppercase
3. Appending the fixed RMS suffix

The suffix used by the script is:

```text
|ALL|ALL|ALL
```

### Example

Input:

```text
client001
```

Generated RMS entity code:

```text
CLIENT001|ALL|ALL|ALL
```

This generated value is then used to locate the corresponding record in the `RMSEntities` table.

---

## 5. Database Update

The script connects to the configured MySQL database and updates the existing RMS entity records.

The target table is:

```text
RMSEntities
```

The update is based on:

```text
RMSEntityCode
```

and modifies:

```text
CashMargin
```

Conceptually:

```text
Input Client Code
       │
       ▼
Normalize Client Code
       │
       ▼
Add |ALL|ALL|ALL
       │
       ▼
Find RMSEntityCode
       │
       ▼
Update CashMargin
```

### Important

The script performs an **UPDATE only**.

It does not:

* Create RMS entities
* Insert missing RMS entities
* Delete RMS entities
* Create or modify database tables

If an `RMSEntityCode` from the input file does not already exist in `RMSEntities`, that record is not counted as updated.

---

## 6. Database Connection Arguments

The script supports the following command-line arguments:

| Argument     | Required | Default                          | Description         |
| ------------ | -------- | -------------------------------- | ------------------- |
| `--host`     | No       | From global config / `127.0.0.1` | MySQL database host |
| `--port`     | No       | From global config / `3306`      | MySQL database port |
| `--user`     | No       | From global config / `root`      | Database username   |
| `--password` | No       | From global config / `root`      | Database password   |
| `--database` | No       | From global config / `Blitz`     | Database name       |

The default values are obtained through:

```text
lib.global_config
```

This allows the script to use the application's configured database settings without requiring all connection parameters to be supplied manually.

---

## 7. Execution Context

`upload_cash_margin_db.py` is a **standalone CLI utility** intended for cash margin data loading during the BOD process.

Its main activities are:

1. Establish a database connection
2. Locate today's cash margin file
3. Detect the file format
4. Read valid client/margin records
5. Generate RMS entity codes
6. Update matching database records
7. Report the number of successfully updated entities

---

## 8. Data Validation and Handling

The script performs basic validation while reading the input file.

### Missing Input File

If today's expected cash margin file is not available, the script logs an error and exits.

### Invalid Margin Value

If the cash margin cannot be converted to a numeric value, that record is skipped.

For example:

```text
CLIENT001|INVALID
```

will not be included in the database update.

### Whitespace and Case

Client codes are normalized before creating the RMS entity code:

```text
client001
```

and:

```text
 CLIENT001
```

are normalized to:

```text
CLIENT001|ALL|ALL|ALL
```

---

## 9. Transaction Handling

Database updates are executed inside a database transaction.

The script uses a transaction context for the update operation, so the database operation is committed when the processing completes successfully.

This helps ensure that the update operation is handled as a controlled database transaction rather than issuing independent commits for every row.

---

## 10. Processing Result

After processing the input records, the script logs the number of RMS entities whose `CashMargin` value was successfully updated.

Example:

```text
CashMargin updated for 125 RMS entities
```

This count represents records for which the database update affected an existing `RMSEntityCode`.

It does not necessarily represent the total number of rows present in the input file.

---

## 11. Dependencies

The script requires:

* Python 3
* MySQL database
* SQLAlchemy
* PyMySQL
* Project logging module:

  * `lib.logger`
* Project configuration module:

  * `lib.global_config`
* Daily cash margin file in the `downloads` directory

The database user must have permission to update the `RMSEntities` table.

---

## 12. Failure Scenarios

### Database Unavailable

If the configured database cannot be reached, the script cannot perform the cash margin update.

### Input File Missing

If the expected daily file:

```text
downloads/Bull8_DEPMYYYYMMDD.CSV
```

does not exist, processing stops.

### Invalid Records

Records with non-numeric cash margin values are skipped rather than being written to the database.

### Unknown RMS Entity

If the generated:

```text
<CLIENT>|ALL|ALL|ALL
```

does not exist in `RMSEntities`, no row is updated for that client.

### Incorrect File Structure

For header-based CSV files, the script expects to find column names containing:

```text
client
```

and:

```text
cash
```

If the required columns cannot be identified, processing may fail.

---

## 13. Verification

After execution, verify the log output:

```text
CashMargin updated for <number> RMS entities
```

For database-level verification, check the corresponding records in:

```text
RMSEntities
```

using the generated entity code.

Example:

```sql
SELECT RMSEntityCode, CashMargin
FROM RMSEntities
WHERE RMSEntityCode = 'CLIENT001|ALL|ALL|ALL';
```

This confirms that the expected cash margin value has been applied to the corresponding RMS entity.

---

## 14. Operational Checklist

Before running the script, verify:

* Today's `Bull8_DEPMYYYYMMDD.CSV` file is present.
* The file is located under the expected `downloads` directory.
* The database is running and reachable.
* Database credentials are correctly configured.
* The expected RMS entities already exist in `RMSEntities`.
* The input file contains valid client and cash margin values.

After execution:

* Check the script log.
* Check the number of updated RMS entities.
* Verify important client records in `RMSEntities`.
* Investigate any mismatch between expected and updated record counts.

---

## 15. Key Takeaway

`upload_cash_margin_db.py` is responsible for **loading the daily client cash margin information into the existing RMS entity records**.

Its operational flow is:

```text
Daily Cash Margin File
        │
        ▼
Bull8_DEPMYYYYMMDD.CSV
        │
        ▼
Detect File Format
        │
        ▼
Read Client + Cash Margin
        │
        ▼
Generate RMS Entity Code
CLIENT|ALL|ALL|ALL
        │
        ▼
Find Existing RMSEntities Record
        │
        ▼
Update CashMargin
        │
        ▼
Report Updated Count
```

The script therefore acts as a **BOD cash margin data-loading step** between the daily cash margin file and the RMS database.
