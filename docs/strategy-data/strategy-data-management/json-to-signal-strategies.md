# JSON to Signal Strategies Database Import

## 1. Introduction

`JsonToDB_SignalStrategies.py` is used to import signal strategy configuration from a JSON file into the MySQL `SignalStrategies` table.

The script validates the input data, maps JSON fields to database columns, checks the database schema, detects existing signal strategies using `StrategyName`, and allows the operator to skip or overwrite existing records.

---

## 2. Target Database

The script uses:

```text
Database: Blitz
Table: SignalStrategies
Uniqueness Check: StrategyName
```

The configured primary/uniqueness field is `StrategyName`.

---

## 3. Supported Fields

The script supports the following JSON-to-database mappings:

| JSON Field           | Database Column      |
| -------------------- | -------------------- |
| `StrategyId`         | `StrategyId`         |
| `StrategyName`       | `StrategyName`       |
| `Description`        | `Description`        |
| `StrategyType`       | `StrategyType`       |
| `RiskLevel`          | `RiskLevel`          |
| `MinimumAmount`      | `MinimumAmount`      |
| `AssetClass`         | `AssetClass`         |
| `DD`                 | `DD`                 |
| `ROI`                | `ROI`                |
| `Subscribers`        | `Subscribers`        |
| `GenralParameters`   | `GenralParameters`   |
| `SpecificParameters` | `SpecificParameters` |
| `Tags`               | `Tags`               |
| `IsDeleted`          | `IsDeleted`          |
| `Fees`               | `Fees`               |
| `Author`             | `Author`             |
| `ROILast30`          | `ROILast30`          |
| `DDLast30`           | `DDLast30`           |
| `SharpeRatioLast30`  | `SharpeRatioLast30`  |
| `Icon`               | `Icon`               |
| `Category`           | `Category`           |

> The script currently uses the field name `GenralParameters` as defined in its mapping.

---

## 4. Required Fields

The following fields are required:

```text
StrategyId
StrategyName
Description
StrategyType
RiskLevel
MinimumAmount
AssetClass
DD
ROI
```

Records with missing or empty required fields are skipped.

---

## 5. Nested JSON Fields

The following fields are treated as nested JSON:

```text
GenralParameters
SpecificParameters
```

These values are serialized into JSON strings before being stored in the database.

---

## 6. Supported Input Format

The script accepts either:

### Single Object

```json
{
  "StrategyId": "SS001",
  "StrategyName": "ExampleSignalStrategy"
}
```

### Array of Objects

```json
[
  {
    "StrategyId": "SS001",
    "StrategyName": "ExampleSignalStrategy"
  },
  {
    "StrategyId": "SS002",
    "StrategyName": "AnotherSignalStrategy"
  }
]
```

The loader converts a single object into a one-element list for processing.

---

## 7. Command Syntax

The script expects exactly one command-line argument:

```bash
python JsonToDB_SignalStrategies.py <path_to_json_file>
```

Example:

```bash
python JsonToDB_SignalStrategies.py signal_strategies.json
```

The usage message is defined in the script as:

```text
Usage: python sync_signal_strategies.py <path_to_json_file>
```

---

## 8. Processing Workflow

```text
Signal Strategy JSON
          │
          ▼
     Load JSON
          │
          ▼
 JSON → DB Mapping
          │
          ▼
 Required Field Check
          │
          ▼
 Schema Validation
          │
          ▼
 Check StrategyName
          │
       ┌──┴──┐
       │     │
    Exists  New
       │     │
       ▼     ▼
   Skip /  Skip /
 Overwrite  Insert
       │     │
       └──┬──┘
          ▼
        Commit
```

---

## 9. Database Schema Check

When the script starts, it retrieves the columns available in the `SignalStrategies` table.

It compares the database columns with the columns defined in the JSON-to-database mapping.

The script reports:

### Missing Database Columns

Mapped columns that are not present in the database table.

### Additional Database Columns

Database columns that are not mapped from JSON.

These checks provide visibility into possible schema mismatches before processing records.

---

## 10. Record Validation

For each JSON record, the script:

1. Converts JSON data into a database record.
2. Checks required fields.
3. Reports missing database fields.
4. Verifies that `StrategyName` exists.
5. Checks whether the strategy already exists.

---

## 11. Existing Signal Strategy

The existing-record check uses:

```text
StrategyName
```

If the strategy exists, the operator is prompted:

```text
Strategy '<name>' already exists. [S]kip / [O]verwrite?
```

### Skip

```text
s
```

The existing strategy remains unchanged.

### Overwrite

```text
o
```

The existing record is updated.

---

## 12. New Signal Strategy

If the strategy does not already exist, the operator is prompted:

```text
Strategy '<name>' not found.
Insert as NEW strategy? [Y]es / [N]o:
```

### Yes

```text
y
```

The new strategy is inserted.

### No

```text
n
```

The record is skipped.

---

## 13. Database Commit

After all records have been processed, the script commits the changes:

```text
conn.commit()
```

It then displays a summary:

```text
Inserted
Updated
Skipped
```

---

## 14. Execution Example

Run:

```bash
python JsonToDB_SignalStrategies.py signal_strategies.json
```

Example processing:

```text
--- Processing record #1 ---

🔑 StrategyName: MomentumSignal

🆕 Strategy 'MomentumSignal' not found.
Insert as NEW strategy? [Y]es / [N]o:
```

Enter:

```text
y
```

to insert the strategy.

If the strategy already exists:

```text
⚠ Strategy 'MomentumSignal' already exists.
[S]kip / [O]verwrite?
```

Enter:

```text
o
```

to update it.

---

## 15. Database Verification

After the import, verify the records:

```sql
SELECT *
FROM SignalStrategies;
```

To check a specific strategy:

```sql
SELECT *
FROM SignalStrategies
WHERE StrategyName = 'MomentumSignal';
```

To check the total number of records:

```sql
SELECT COUNT(*)
FROM SignalStrategies;
```

---

## 16. Troubleshooting

### JSON File Error

Verify the file exists:

```bash
ls -lh signal_strategies.json
```

### Invalid JSON

Validate:

```bash
python -m json.tool signal_strategies.json
```

### Required Field Missing

The script reports the missing fields and skips that record.

Example:

```text
❌ Required fields missing/empty, skipping record.
```

### StrategyName Missing

The record cannot be processed without `StrategyName`.

### Schema Warning

If JSON mappings do not match the database schema, the script reports missing or additional database columns.

### Database Error

The script reports database errors using:

```text
Database error: <error>
```

---

## 17. Operational Checklist

### Before Import

* [ ] JSON file exists.
* [ ] JSON syntax is valid.
* [ ] Required fields are available.
* [ ] `SignalStrategies` table exists.
* [ ] Database is accessible.
* [ ] Database credentials are correct.
* [ ] Database schema matches the expected mapping.

### During Import

* [ ] Review schema warnings.
* [ ] Review required-field errors.
* [ ] Confirm existing strategies before overwrite.
* [ ] Confirm new strategies before insertion.

### After Import

* [ ] Commit completed successfully.
* [ ] Inserted count is correct.
* [ ] Updated count is correct.
* [ ] Skipped count is understood.
* [ ] Database records have been verified.
