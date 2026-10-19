# JSON to Strategies Database Import

## 1. Introduction

`JsonToDB_Strategies.py` is used to import strategy configuration from a JSON file into the MySQL `Strategies` table.

The utility converts JSON fields into the corresponding database columns, validates required fields, calculates a SHA-256 checksum, detects existing strategies, and then allows the operator to either skip or overwrite existing records.

The script also supports dry-run and debug modes.

---

## 2. Target Database

The script targets:

```text
Database: Blitz
Table: Strategies
Primary Key: StrategyId
```

---

## 3. Supported JSON Fields

The script maps JSON fields to database columns.

| JSON Field        | Database Column   |
| ----------------- | ----------------- |
| `strategyid`      | `StrategyId`      |
| `strategyname`    | `StrategyName`    |
| `name`            | `StrategyName`    |
| `assemblyname`    | `AssemblyName`    |
| `version`         | `Version`         |
| `owner`           | `Owner`           |
| `category`        | `Category`        |
| `displayname`     | `DisplayName`     |
| `description`     | `Description`     |
| `strategytype`    | `StrategyType`    |
| `checksum`        | `Checksum`        |
| `ivobjects`       | `IVObjects`       |
| `inputparameters` | `InputParameters` |
| `outputvariables` | `OutputVariables` |
| `commands`        | `Commands`        |
| `controls`        | `Controls`        |
| `globalcommands`  | `GlobalCommands`  |
| `exchangedetails` | `ExchangeDetails` |

---

## 4. Required Fields

The following fields are required:

```text
StrategyId
StrategyName
AssemblyName
Version
Owner
Category
Description
StrategyType
Checksum
```

A record containing missing or empty required fields is skipped unless `--force` is specified.

---

## 5. JSON Input Formats

The utility accepts:

### JSON Array

```json
[
  {
    "strategyid": "STR001",
    "strategyname": "ExampleStrategy"
  },
  {
    "strategyid": "STR002",
    "strategyname": "AnotherStrategy"
  }
]
```

### Single JSON Object

```json
{
  "strategyid": "STR001",
  "strategyname": "ExampleStrategy"
}
```

The loader also supports an object containing a single list value.

---

## 6. Command Syntax

```bash
python JsonToDB_Strategies.py --file <json-file>
```

Example:

```bash
python JsonToDB_Strategies.py --file strategies.json
```

The `--file` argument is mandatory.

---

## 7. Command-Line Options

| Option         | Description                                                            |
| -------------- | ---------------------------------------------------------------------- |
| `--file`, `-f` | JSON input file                                                        |
| `--force`      | Allows processing records even when required fields are missing        |
| `--dry-run`    | Processes the data without executing database insert/update statements |
| `--debug`      | Prints the generated database record                                   |

---

## 8. Field Normalization

The script normalizes nested JSON structures before storing them.

The following fields are converted to PascalCase recursively:

```text
IVObjects
InputParameters
OutputVariables
Commands
Controls
GlobalCommands
```

`ExchangeDetails` is handled separately and preserves its camelCase structure.

---

## 9. Checksum

Before creating the database record, the script calculates a SHA-256 checksum.

The checksum is generated from the JSON record after excluding the existing checksum field.

The JSON is serialized with sorted keys before calculating the SHA-256 hash.

The generated value is stored in:

```text
Checksum
```

---

## 10. Existing Strategy Handling

The script checks whether a strategy already exists using:

```text
StrategyId
```

If an existing strategy is found, the operator is prompted:

```text
Existing found → [S]kip / [O]verwrite:
```

### Skip

Entering:

```text
s
```

skips the existing record.

### Overwrite

Entering:

```text
o
```

updates the existing database record.

---

## 11. New Strategy Handling

If the strategy does not exist, the script asks:

```text
New found → [S]kip / [O]insert:
```

Entering:

```text
o
```

inserts the record.

Entering:

```text
s
```

skips the record.

---

## 12. Missing Database Columns

Before importing new records, the script reads the database table metadata from `INFORMATION_SCHEMA.COLUMNS`.

This allows the script to identify:

* Column defaults
* Nullable columns
* Data types

Missing database columns can then be automatically populated using defaults, nullable values, or type-specific fallback values.

---

## 13. Dry Run

Use:

```bash
python JsonToDB_Strategies.py \
    --file strategies.json \
    --dry-run
```

In dry-run mode, database INSERT and UPDATE execution is skipped while the records are still processed.

---

## 14. Debug Mode

Use:

```bash
python JsonToDB_Strategies.py \
    --file strategies.json \
    --debug
```

Debug mode prints the generated database record before validation and database processing.

This is useful when checking JSON-to-database field mapping.

---

## 15. Import Workflow

```text
JSON File
   │
   ▼
Load JSON Records
   │
   ▼
Build DB Record
   │
   ├── Calculate Checksum
   │
   ├── Map Fields
   │
   └── Normalize Nested JSON
   │
   ▼
Validate Required Fields
   │
   ▼
Check StrategyId
   │
   ├── Existing
   │      ├── Skip
   │      └── Update
   │
   └── New
          ├── Skip
          └── Insert
   │
   ▼
Commit
```

---

## 16. Execution Result

At the end of execution, the script displays a summary containing:

```text
Inserted
Updated
Skipped
```

Example:

```text
✅ Done | Inserted=5, Updated=2, Skipped=1
```

---

## 17. Recommended Import Procedure

### Step 1 – Validate JSON

```bash
python -m json.tool strategies.json
```

### Step 2 – Perform Debug Run

```bash
python JsonToDB_Strategies.py \
    --file strategies.json \
    --debug
```

### Step 3 – Run Import

```bash
python JsonToDB_Strategies.py \
    --file strategies.json
```

### Step 4 – Respond to Prompts

For existing strategies:

```text
s = Skip
o = Overwrite
```

For new strategies:

```text
s = Skip
o = Insert
```

### Step 5 – Verify Database

```sql
SELECT *
FROM Strategies;
```

---

## 18. Troubleshooting

### JSON File Not Found

Check:

```bash
ls -lh strategies.json
```

Use the correct absolute or relative path.

### Invalid JSON

Run:

```bash
python -m json.tool strategies.json
```

### Missing Required Fields

The script reports the missing fields and skips the record unless `--force` is supplied.

### Database Connection Error

Verify:

```text
Host
Port
Username
Password
Database
```

### Unexpected Database Values

Use debug mode:

```bash
python JsonToDB_Strategies.py \
    --file strategies.json \
    --debug
```

---

## 19. Operational Checklist

* [ ] JSON file is available.
* [ ] JSON syntax is valid.
* [ ] Required strategy fields are present.
* [ ] Database is accessible.
* [ ] `Strategies` table exists.
* [ ] Correct JSON file is selected.
* [ ] Existing strategy behavior is reviewed.
* [ ] Import summary is checked.
* [ ] Database records are verified.
