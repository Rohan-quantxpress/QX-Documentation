## `store_adapter_info.py`

## 1. Purpose and Business Intent

`store_adapter_info.py` is a **configuration ingestion utility** that reads adapter configuration files, converts them from INI format into JSON, and stores the resulting configuration in Redis.

The script is responsible for making adapter configuration available in a centralized and quickly accessible format for components that require adapter metadata during runtime.

### Primary Responsibilities

* Locate adapter configuration files matching `*AdapterInfo.ini`
* Read and convert INI configuration into structured JSON
* Verify that the required `AdapterInfo` section exists
* Verify that the `adapterid` field exists
* Generate a Redis key using the adapter ID
* Store the complete JSON configuration in Redis

The script does **not** perform database operations or modify the original INI files.

---

## 2. Adapter Configuration Files

The script scans the configured directory for files ending with:

```
text
AdapterInfo.ini
```

For example:

```
text
NSEAdapterInfo.ini
BSEAdapterInfo.ini
TestAdapterInfo.ini
```

Each file is expected to contain an `AdapterInfo` section with an `adapterid`.

### Required Configuration

```
ini
[AdapterInfo]
adapterid = ABC123
name = SampleAdapter
```

The `adapterid` is important because it is used to identify the adapter when storing its configuration in Redis.

---

## 3. INI to JSON Conversion

The script uses Python's `configparser` to read the INI file.

Each INI section becomes a JSON object, while each configuration key becomes a property within that object.

### Example

**INI:**

```
ini
[AdapterInfo]
adapterid = ABC123
name = SampleAdapter

[Connection]
host = localhost
port = 5000
```

**Stored JSON:**

```
json
{
    "AdapterInfo": {
        "adapterid": "ABC123",
        "name": "SampleAdapter"
    },
    "Connection": {
        "host": "localhost",
        "port": "5000"
    }
}
```

The configuration values are read as strings by `configparser`. The script does not perform additional type conversion.

---

## 4. Redis Storage

After conversion, the complete JSON configuration is stored in Redis.

The Redis key is generated from the `adapterid` found inside the `AdapterInfo` section.

### Redis Key Format

```
text
ADAPTERINFO:<adapterid>
```

For example:

```
text
ADAPTERINFO:ABC123
```

The key is stored in uppercase.

### Stored Value

The Redis value contains the complete adapter configuration as a JSON string.

Conceptually:

```
text
Key:
ADAPTERINFO:ABC123

Value:
{
    "AdapterInfo": {
        "adapterid": "ABC123",
        "name": "SampleAdapter"
    },
    "Connection": {
        "host": "localhost",
        "port": "5000"
    }
}
```

This allows downstream applications to retrieve an adapter's complete configuration using its adapter ID.

---

## 5. Required Adapter Information

Before storing a configuration, the script checks for two required items:

### `AdapterInfo` Section

The INI file must contain:

```
ini
[AdapterInfo]
```

If the section is missing, the file is not stored in Redis.

### `adapterid`

The `AdapterInfo` section must contain:

```
ini
adapterid = <adapter-id>
```

If `adapterid` is missing, the file is not stored.

The `adapterid` is then used to generate the Redis key.

> **Important:** The script does not perform extensive validation of the remaining adapter configuration fields. Its explicit structural checks are for the `AdapterInfo` section and `adapterid`.

---

## 6. Execution Context

`store_adapter_info.py` is a **standalone command-line utility**.

It can be executed manually or invoked from an operational workflow where adapter configuration needs to be loaded into Redis.

### Runtime Flow

```
text
AdapterInfo.ini Files
        │
        ▼
Scan Configured Directory
        │
        ▼
Read INI Configuration
        │
        ▼
Convert INI → JSON
        │
        ▼
Check AdapterInfo
        │
        ▼
Check adapterid
        │
        ▼
Generate Redis Key
ADAPTERINFO:<adapterid>
        │
        ▼
Store JSON in Redis
```

The script processes all matching adapter files found in the configured directory.

---

## 7. Command-Line Arguments

The script supports the following arguments:

| Argument       | Required | Default           | Description                                   |
| -------------- | -------- | ----------------- | --------------------------------------------- |
| `--redis_host` | No       | `localhost`       | Redis server hostname or IP                   |
| `--redis_port` | No       | `6379`            | Redis server port                             |
| `--redis_db`   | No       | `0`               | Redis database number                         |
| `--directory`  | No       | `adapter_configs` | Directory containing `*AdapterInfo.ini` files |

Redis defaults are obtained through the project's global configuration using `lib.global_config.get()`.

### Example

```
bash
python store_adapter_info.py
```

Using a custom configuration directory:

```
bash
python store_adapter_info.py --directory /path/to/adapter_configs
```

Using a specific Redis server:

```
bash
python store_adapter_info.py \
    --redis_host 192.168.1.10 \
    --redis_port 6379 \
    --redis_db 0
```

---

## 8. Redis Connection

Before processing files, the script establishes a connection to Redis and performs a `PING` test.

Processing starts only after the Redis connection is successfully verified.

If Redis cannot be reached, the script:

1. Displays the connection error
2. Does not process adapter files
3. Exits with status code `1`

This prevents the script from attempting to process configurations when the destination Redis service is unavailable.

---

## 9. File Processing

The configured directory is scanned using the following filename condition:

```
text
*AdapterInfo.ini
```

For every matching file:

1. The INI file is read.
2. Its sections and values are converted into JSON.
3. The `AdapterInfo` section is checked.
4. The `adapterid` is checked.
5. A Redis key is generated.
6. The JSON configuration is stored in Redis.
7. The file is counted as successfully processed.

Files that cannot be converted or do not contain the required adapter information are skipped.

---

## 10. Error Handling

The script handles common operational failures during processing.

### Directory Not Found

If the configured directory does not exist, the script reports the error and does not process files.

### Invalid INI Configuration

If the INI file cannot be parsed, the file is skipped and an error is reported.

### Missing `AdapterInfo`

If the required section is not present, the configuration is not stored.

### Missing `adapterid`

If the adapter ID is missing, the configuration is not stored.

### Redis Connection Failure

If Redis is unavailable during startup, the script exits with status `1`.

### Redis Storage Failure

If storing a configuration in Redis fails, the error is reported and the affected file is not counted as successfully stored.

---

## 11. Processing Result

At the end of execution, the script prints the total number of adapter configuration files successfully stored in Redis.

Example:

```
text
Finished processing. Total files processed and stored in Redis: 5
```

This provides a simple operational indication of how many configurations were successfully loaded.

---

## 12. Dependencies

The script requires:

* Python 3
* `redis`
* Python standard library:

  * `os`
  * `configparser`
  * `json`
  * `sys`
  * `argparse`
* Project configuration module:

  * `lib.global_config`

Redis must also be running and reachable using the configured host, port, and database.

---

## 13. Verification

After successful execution, verify that the expected adapter keys exist in Redis.

For example:

```
bash
redis-cli
```

Then:

```
text
KEYS ADAPTERINFO:*
```

To inspect a specific adapter:

```
text
GET ADAPTERINFO:ABC123
```

The returned value should contain the JSON representation of the corresponding `AdapterInfo.ini` file.

---

## 14. Operational Considerations

When using this script as part of system initialization or BOD-related workflows, verify:

* The adapter configuration directory exists.
* Required `*AdapterInfo.ini` files are present.
* Each configuration contains an `AdapterInfo` section.
* Each configuration contains a valid `adapterid`.
* Redis is running and accessible.
* The expected `ADAPTERINFO:<adapterid>` keys are present after execution.

The script does not remove existing Redis keys before storing configurations. Therefore, successfully processed adapters are written using their generated adapter-specific keys.

---

## 15. Key Takeaway

`store_adapter_info.py` provides a simple bridge between **file-based adapter configuration and Redis-based runtime configuration**.

Its overall responsibility is:

```
text
Adapter INI Files
       ↓
INI → JSON
       ↓
Validate AdapterInfo / adapterid
       ↓
Generate ADAPTERINFO:<adapterid>
       ↓
Store Configuration in Redis
```

The script does not modify databases or perform adapter-specific business processing. Its main purpose is to prepare adapter configuration in Redis so that other components can access the configuration centrally at runtime.
