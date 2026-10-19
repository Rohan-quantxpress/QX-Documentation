# Database to JSON

## 1. Introduction

`DatabaseToJson.py` is a command-line utility used to export data from a MySQL table into JSON format.

The utility can export an entire table into a single JSON file or generate a separate JSON file for every database row.

It also supports optional filtering, formatted JSON output, and dry-run execution.

---

## 2. Purpose

The utility can be used for:

* Strategy configuration backup
* Data migration
* Database-to-file conversion
* Creating JSON dumps
* Generating individual strategy configuration files
* Reviewing database data outside the database

---

## 3. Export Modes

Two export modes are supported.

### Single JSON File

All selected database rows are written into one JSON array.

```text
Database Table
      │
      ▼
DatabaseToJson.py
      │
      ▼
output.json
```

### One JSON File Per Row

Each database row is written to a separate JSON file.

```text
Strategies
   │
   ├── Strategy001.json
   ├── Strategy002.json
   ├── Strategy003.json
   └── ...
```

The filename is generated using the value of the selected ID column. The ID-column lookup is case-sensitive.

---

## 4. Command Syntax

```bash
python DatabaseToJson.py --table <table> --out <output>
```

---

## 5. Command-Line Parameters

| Parameter       |    Required | Description                                  |
| --------------- | ----------: | -------------------------------------------- |
| `--table`       |         Yes | MySQL table to export                        |
| `--out`         |         Yes | Output JSON file or directory                |
| `--where`       |          No | Optional filter condition                    |
| `--pretty`      |          No | Formats JSON with indentation                |
| `--one-per-row` |          No | Creates one JSON file per database row       |
| `--id-col`      | Conditional | Column used for per-row filenames            |
| `--dry-run`     |          No | Fetches/counts records without writing files |

These parameters are defined by the script's command-line parser.

---

## 6. Export Entire Table

Example:

```bash
python DatabaseToJson.py \
    --table Strategies \
    --out strategies.json \
    --pretty
```

This exports the selected table into:

```text
strategies.json
```

The resulting file contains the records as a JSON array.

---

## 7. Export One File Per Strategy

Example:

```bash
python DatabaseToJson.py \
    --table Strategies \
    --one-per-row \
    --id-col StrategyId \
    --out strategies \
    --pretty
```

The utility creates the output directory if it does not already exist.

Example output:

```text
strategies/
├── STR001.json
├── STR002.json
├── STR003.json
└── STR004.json
```

---

## 8. Filtering Records

The `--where` option can be used to export only records matching a condition.

Example:

```bash
python DatabaseToJson.py \
    --table Strategies \
    --where "IsDeleted=0" \
    --out active_strategies.json \
    --pretty
```

The script appends the supplied condition to the generated `SELECT` statement.

> Use valid SQL conditions when specifying `--where`.

---

## 9. Dry Run

The dry-run option can be used when you only want to check how many rows would be exported.

```bash
python DatabaseToJson.py \
    --table Strategies \
    --out strategies.json \
    --dry-run
```

The script connects to the database, fetches the records, displays the count, and exits without writing JSON files.

---

## 10. JSON Conversion

The utility attempts to identify JSON-like strings stored in database columns.

Values beginning and ending with `{}` or `[]` are parsed as JSON when possible. If parsing fails, the original value is retained.

This allows database fields containing JSON strings to be represented as JSON objects or arrays in the exported file.

---

## 11. `full_json` Handling

If a database row contains a `full_json` field and the field contains valid JSON, that JSON object is used as the exported representation of the row.

This allows the original JSON structure to be preserved when available.

---

## 12. Date and Decimal Handling

The utility converts database-specific values into JSON-compatible representations.

### Date/Time

Date and time values are converted using ISO formatting where possible.

### Decimal

Decimal values are converted to floating-point values where possible.

This processing is implemented in `_json_default()`.

---

## 13. Output Formatting

Without `--pretty`, JSON is written in compact form.

Example:

```bash
python DatabaseToJson.py \
    --table Strategies \
    --out strategies.json
```

With `--pretty`:

```bash
python DatabaseToJson.py \
    --table Strategies \
    --out strategies.json \
    --pretty
```

Pretty mode makes the generated JSON easier to read and review.

---

## 14. Complete Examples

### Export Strategies

```bash
python DatabaseToJson.py \
    --table Strategies \
    --out strategies.json \
    --pretty
```

### Export Signal Strategies

```bash
python DatabaseToJson.py \
    --table SignalStrategies \
    --out signal_strategies.json \
    --pretty
```

### Export Individual Strategies

```bash
python DatabaseToJson.py \
    --table Strategies \
    --one-per-row \
    --id-col StrategyId \
    --out strategies \
    --pretty
```

### Export Active Strategies

```bash
python DatabaseToJson.py \
    --table Strategies \
    --where "IsDeleted=0" \
    --out active_strategies.json \
    --pretty
```

---

## 15. Verification

After export, verify the generated file:

```bash
ls -lh strategies.json
```

Validate JSON syntax:

```bash
python -m json.tool strategies.json
```

For an individual file:

```bash
python -m json.tool strategies/STR001.json
```

---

## 16. Common Errors

### Missing `--table`

```text
error: the following arguments are required: --table
```

Provide the table name.

### Missing `--out`

Provide the output file or directory.

### `--one-per-row` Without `--id-col`

The script requires `--id-col` when using `--one-per-row`.

Example:

```bash
--one-per-row --id-col StrategyId
```

### Database Connection Error

Check:

* MySQL server status
* Host
* Port
* Username
* Password
* Database name
* Network connectivity

---

## 17. Operational Checklist

Before export:

* [ ] Database is accessible.
* [ ] Table exists.
* [ ] User has SELECT permission.
* [ ] Output location is writable.
* [ ] Correct table name is specified.

After export:

* [ ] Output file/directory exists.
* [ ] Expected record count is present.
* [ ] JSON syntax is valid.
* [ ] Exported strategy data is correct.
