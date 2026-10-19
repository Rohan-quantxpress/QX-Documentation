# Strategy Data Management

## 1. Introduction

The **Strategy Data Management** module provides utilities for transferring strategy configuration data between the **MySQL database** and **JSON files**.

The module is designed to simplify strategy data migration, backup, restoration, and synchronization activities by providing command-line utilities for both database export and JSON import operations.

The module supports two primary strategy data types:

* **Strategies**
* **Signal Strategies**

It also provides a generic database export utility that can export any MySQL table into JSON format.

---

## 2. Document Overview

This document explains the functionality and operational usage of the Strategy Data Management utilities.

It covers:

* Database-to-JSON export
* JSON-to-Strategies import
* JSON-to-SignalStrategies import
* Supported command-line parameters
* JSON input and output handling
* Database connectivity
* Record validation
* Insert and update behavior
* Duplicate strategy handling
* Dry-run operations
* JSON formatting and nested JSON handling
* Database verification
* Common errors and troubleshooting

The document is intended for developers, system administrators, deployment engineers, and users responsible for managing strategy configuration data.

---

## 3. Module Components

The module consists of three Python scripts.

| Script                         | Direction       | Purpose                                                        |
| ------------------------------ | --------------- | -------------------------------------------------------------- |
| `DatabaseToJson.py`            | Database → JSON | Exports MySQL table data into JSON                             |
| `JsonToDB_Strategies.py`       | JSON → Database | Imports strategy data into the `Strategies` table              |
| `JsonToDB_SignalStrategies.py` | JSON → Database | Imports signal strategy data into the `SignalStrategies` table |

---

## 4. Overall Data Flow

The utilities provide a two-way data flow between JSON files and the database.

<div class="data-flow">
<pre>
                    ┌──────────────────────┐
                    │       MySQL DB       │
                    │       Blitz DB       │
                    └──────────┬───────────┘
                               │
                             Export
                               ▼
                    ┌──────────────────────┐
                    │  DatabaseToJson.py   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      JSON Files      │
                    └──────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    │                      │
                    ▼                      ▼
          ┌──────────────────┐   ┌────────────────────────┐
          │ JsonToDB_        │   │ JsonToDB_              │
          │ Strategies.py    │   │ SignalStrategies.py    │
          └────────┬─────────┘   └───────────┬────────────┘
                   │                         │
                   ▼                         ▼
          ┌──────────────────┐   ┌────────────────────────┐
          │    Strategies    │   │    SignalStrategies    │
          │      Table       │   │         Table          │
          └──────────────────┘   └────────────────────────┘
</pre>
</div>

---

## 5. Database

The scripts use a MySQL database named `Blitz`.

The database connection configuration is defined inside the scripts.

For the Strategies import/export utilities, the configured database host is `192.168.1.15`, port `3306`, and database `Blitz`.
The Signal Strategies import script is configured separately and currently uses `127.0.0.1` as the database host.

> **Security Note:** Database credentials should be stored securely in production environments. Avoid committing passwords directly into source control.

---

## 6. Strategy Tables

### Strategies

The strategy import utility operates on:

```text
Strategies
```

The primary identifier used for existing-record detection is:

```text
StrategyId
```

### SignalStrategies

The signal strategy import utility operates on:

```text
SignalStrategies
```

The uniqueness check is performed using:

```text
StrategyName
```

---

## 7. Typical Use Cases

### Export

Database data can be exported when:

* A backup of strategy configuration is required.
* Strategy data needs to be migrated.
* Configuration needs to be transferred between environments.
* Individual strategy JSON files are required.
* Database records need to be reviewed in JSON format.

### Import

JSON import can be used when:

* Strategies need to be restored.
* New strategy configurations need to be inserted.
* Existing strategy configurations need to be updated.
* Strategy data needs to be migrated into another database.

---

## 8. Operational Workflow

A typical operational workflow is:

```text
1. Export database records
          ↓
2. Generate JSON dump
          ↓
3. Review / modify JSON
          ↓
4. Validate JSON
          ↓
5. Import JSON
          ↓
6. Validate database records
```

---

## 9. Prerequisites

Before using the utilities, ensure:

* Python 3 is installed.
* MySQL server is accessible.
* The `Blitz` database exists.
* Required database tables exist.
* Database credentials are configured.
* The `mysql.connector` Python package is installed.
* Input JSON files are valid.
* The executing user has appropriate database permissions.

Check Python:

```bash
python3 --version
```

---

## 10. Important Operational Considerations

The import scripts do not simply perform blind inserts.

They perform validation and check whether a strategy already exists before deciding whether to insert, update, or skip the record.

The Strategies importer also supports:

* `--force`
* `--dry-run`
* `--debug`

The Signal Strategies importer performs required-field and schema checks before processing records.

The Database-to-JSON utility supports:

* Full-table export
* One-file-per-row export
* Optional filtering
* Pretty JSON formatting
* Dry-run mode

---

## 11. Related Documents

* [Database to JSON](database-to-json.md)
* [JSON to Strategies](json-to-strategies.md)
* [JSON to Signal Strategies](json-to-signal-strategies.md)
