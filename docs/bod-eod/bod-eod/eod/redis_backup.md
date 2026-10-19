# Redis Cache Backup and Deletion

## 1. Purpose

`redis_cache_backup_and_deletion.py` is a maintenance utility for backing up and deleting Redis keys that match a specified pattern.

The script performs Redis operations in the following sequence:

1. Connect to Redis
2. Back up matching Redis keys to a JSON file
3. Delete the matching Redis keys
4. Optionally delete a specified Redis configuration file

The backup step is a safety mechanism before Redis key deletion. If the backup fails, the script stops before performing key deletion.

> ⚠️ **Warning:** Redis key deletion and configuration-file deletion are destructive operations. Use the script only after confirming the key pattern and configuration-file path.

---

## 2. Scope of Operations

The script supports three operations:

| Operation                   | Description                                             |
| --------------------------- | ------------------------------------------------------- |
| Redis Backup                | Backs up Redis keys matching the specified pattern      |
| Redis Key Deletion          | Deletes keys matching the same pattern                  |
| Configuration File Deletion | Optionally deletes a specified Redis configuration file |

The operations are performed in this order:

```text
Connect to Redis
       ↓
Backup matching Redis keys
       ↓
Backup successful?
       ↓
      Yes
       ↓
Delete matching Redis keys
       ↓
Optional configuration-file deletion
       ↓
Complete
```

If the Redis backup fails, the script terminates before Redis key deletion.

---

## 3. Execution Context

The script is a standalone command-line utility.

It can be executed manually or invoked by another operational process.

The script itself does not contain scheduling logic. Any scheduling or BOD/EOD orchestration must be handled externally.

---

## 4. Redis Connection

The script connects to Redis using the following parameters:

| Parameter | Default                                  |
| --------- | ---------------------------------------- |
| Host      | Global Redis configuration / `localhost` |
| Port      | Global Redis configuration / `6379`      |
| Password  | `None`                                   |

The connection uses:

```text
decode_responses=False
```

This allows the script to receive Redis data as bytes and handle UTF-8 and binary values explicitly.

After creating the Redis client, the script calls `PING` to verify the connection.

Successful connection:

```text
Successfully connected to Redis at <host>:<port>
```

If the connection fails, the script exits with status `1`.

---

## 5. Command-Line Arguments

The script accepts the following arguments:

| Argument               | Required | Description                                 |
| ---------------------- | -------: | ------------------------------------------- |
| `--host`               |       No | Redis server host                           |
| `--port`               |       No | Redis server port                           |
| `--password`           |       No | Redis authentication password               |
| `--key_pattern`        |  **Yes** | Pattern of Redis keys to back up and delete |
| `--backup_file`        |       No | Backup filename                             |
| `--delete_config_path` |       No | Redis configuration file to delete          |

The key pattern is mandatory.

---

## 6. Key Pattern

The `--key_pattern` argument determines which Redis keys are processed.

Example:

```bash
--key_pattern "my_app:*"
```

Another example:

```bash
--key_pattern "PROCESSES*"
```

A wildcard pattern can also be used:

```bash
--key_pattern "*"
```

The same pattern is used for both:

* Backup
* Deletion

The backup operation discovers matching keys using Redis `SCAN`.

---

## 7. Backup Operation

### `backup_redis_keys()`

The backup function collects all Redis keys matching the supplied pattern and stores their values in JSON format.

The process is:

```text
SCAN matching keys
       ↓
Read Redis key type
       ↓
Read key value
       ↓
Convert bytes to UTF-8/Base64
       ↓
Build backup structure
       ↓
Write JSON file
```

The backup function returns:

| Return Value | Meaning                       |
| ------------ | ----------------------------- |
| `True`       | Backup completed successfully |
| `False`      | Backup failed                 |

---

## 8. Redis Key Discovery

The script uses:

```text
scan_iter(match=key_pattern)
```

rather than retrieving all keys with a single `KEYS` operation.

This allows the script to iterate through matching keys using Redis SCAN behavior.

The same approach is used during deletion to collect keys matching the pattern.

---

## 9. Supported Redis Data Types

The script explicitly handles the following Redis data types:

| Redis Type  | Backup Handling               |
| ----------- | ----------------------------- |
| `string`    | Stores the value directly     |
| `hash`      | Stores hash fields and values |
| `list`      | Stores all list items         |
| `set`       | Stores all set members        |
| `zset`      | Stores members and scores     |
| Other types | Skipped with a warning        |

Unsupported types are not added to the backup.

---

## 10. String Values

For Redis strings, the script reads the value and attempts UTF-8 decoding.

Example logical structure:

```json
{
    "my_key": {
        "type": "string",
        "value": "example",
        "is_key_base64": false,
        "is_value_base64": false
    }
}
```

The actual value depends on the Redis data.

---

## 11. Hash Values

Redis hashes are stored as an object containing individual fields.

Example structure:

```json
{
    "user:1001": {
        "type": "hash",
        "value": {
            "name": {
                "value": "Rohan",
                "is_base64": false
            }
        },
        "is_key_base64": false,
        "is_value_base64": false
    }
}
```

The script processes hash fields and values individually.

### Important

The script tracks Base64 encoding for hash **values**, but it does not preserve a separate Base64 flag for hash field names.

---

## 12. List Values

Redis lists are backed up by reading all elements.

Each list item is stored with its value and Base64 indicator.

Example structure:

```json
{
    "queue": {
        "type": "list",
        "value": [
            {
                "value": "item1",
                "is_base64": false
            },
            {
                "value": "item2",
                "is_base64": false
            }
        ]
    }
}
```

The script reads the list using `LRANGE` from the first element through the last element.

---

## 13. Set Values

Redis sets are stored as a list of members.

Example structure:

```json
{
    "users": {
        "type": "set",
        "value": [
            {
                "value": "user1",
                "is_base64": false
            },
            {
                "value": "user2",
                "is_base64": false
            }
        ]
    }
}
```

The script reads set members using `SMEMBERS`.

---

## 14. Sorted Set Values

Redis sorted sets (`zset`) preserve both the member and its score.

Example:

```json
{
    "leaderboard": {
        "type": "zset",
        "value": [
            {
                "member": "user1",
                "score": 100.0,
                "is_base64": false
            }
        ]
    }
}
```

The script retrieves members together with their scores using `ZRANGE ... WITHSCORES`.

---

## 15. Binary Data Handling

Redis values are retrieved as bytes.

The script attempts to decode byte data as UTF-8.

If UTF-8 decoding fails, the data is encoded using Base64.

```text
Redis bytes
    ↓
UTF-8 decoding
    ↓
Successful?
   ↙     ↘
 Yes      No
 ↓         ↓
Text     Base64
```

This behavior is implemented by `decode_or_base64()`.

### Base64 Indicators

The backup structure can contain flags such as:

```text
is_key_base64
is_value_base64
```

For nested structures such as lists, sets, and sorted sets, the Base64 status is stored alongside individual values or members.

---

## 16. Backup Filename

If `--backup_file` is not supplied, the script automatically generates a filename using the current timestamp.

Format:

```text
redis_backup_YYYYMMDD_HHMMSS.json
```

Example:

```text
redis_backup_20260917_110530.json
```

This filename is generated before the Redis connection and backup process begins.

---

## 17. Backup Directory

The script always stores the backup under:

```text
backup/
```

If the directory does not exist, it is created automatically.

The supplied backup filename is placed inside this directory.

Example:

```text
backup/
└── redis_backup_20260917_110530.json
```

If a custom backup filename is supplied:

```bash
--backup_file my_redis_backup.json
```

the resulting path is:

```text
backup/
└── my_redis_backup.json
```

---

## 18. Backup File Format

The backup is stored as JSON.

The top-level structure uses Redis keys as JSON object keys.

A simplified example:

```json
{
    "my_app:key1": {
        "type": "string",
        "value": "example",
        "is_key_base64": false,
        "is_value_base64": false
    }
}
```

For each backed-up key, the script stores:

* Redis data type
* Redis value
* Key Base64 indicator
* Value Base64 indicator where applicable

The JSON file is written with indentation for readability.

---

## 19. Backup Completion

After successfully writing the JSON file, the script reports the number of backed-up keys.

Example:

```text
Backup completed. 25 keys backed up to 'backup/redis_backup_20260917_110530.json'
```

The backup function then returns:

```text
True
```

---

## 20. Backup Failure

The backup operation can fail because of:

* Redis errors
* File-writing errors
* Unexpected exceptions

Redis errors are reported as:

```text
Error during Redis backup: <error>
```

File-writing failures are reported separately.

If the backup fails, `False` is returned.

The main process then stops:

```text
Backup failed. Aborting further operations.
```

No Redis key deletion is performed after a failed backup.

---

## 21. Redis Key Deletion

### `delete_redis_keys()`

After a successful backup, the script scans Redis again for keys matching the same pattern.

The matching keys are collected and deleted using a Redis pipeline.

### Deletion Flow

```text
SCAN matching keys
       ↓
Collect matching keys
       ↓
Create Redis pipeline
       ↓
Add DELETE commands
       ↓
Execute pipeline
       ↓
Calculate deleted count
```

---

## 22. Batch Deletion

The script creates a Redis pipeline and adds a `DELETE` command for each matching key.

The pipeline is then executed:

```text
Matching Keys
     ↓
Redis Pipeline
     ↓
DELETE key 1
DELETE key 2
DELETE key 3
...
     ↓
Execute
```

The returned command results are summed to calculate the number of deleted keys.

---

## 23. No Matching Keys

If no keys match the specified pattern, the script reports:

```text
No keys found matching pattern '<pattern>' for deletion.
```

The deletion function returns:

```text
0
```

---

## 24. Deletion Errors

Redis errors during deletion are caught and reported.

Example:

```text
Error during Redis key deletion: <error>
```

The function returns:

```text
0
```

Unexpected exceptions are handled similarly.

### Important

A deletion result of `0` does not terminate the entire script.

The main function prints a warning and continues to the optional configuration-file deletion step.

---

## 25. Redis Configuration File Deletion

The script optionally supports deleting a Redis configuration file.

This is controlled using:

```bash
--delete_config_path
```

Example:

```bash
--delete_config_path /path/to/redis.conf
```

This operation is independent of Redis key deletion.

---

## 26. Configuration File Safety

Configuration-file deletion is a destructive filesystem operation.

The script displays a warning:

```text
WARNING: Deleting a Redis configuration file can lead to data loss or service disruption.
```

However, the current implementation does **not** ask the operator for interactive confirmation.

If `--delete_config_path` is supplied, the script proceeds directly to the deletion function.

> ⚠️ Always verify the complete configuration-file path before executing the script with `--delete_config_path`.

---

## 27. Configuration File Validation

Before deletion, the script checks whether a path was provided.

If no path is supplied:

```text
No Redis configuration file path provided for deletion. Skipping.
```

If the specified file does not exist:

```text
Error: Configuration file '<path>' does not exist.
```

In both cases, configuration-file deletion is not performed.

---

## 28. Configuration File Permissions

The script uses:

```text
os.remove()
```

to delete the configuration file.

The executing user must have sufficient filesystem permissions.

On production systems, elevated privileges may be required.

If deletion fails because of filesystem permissions, the script reports the error and indicates that elevated privileges may be required.

---

## 29. Complete Execution Sequence

The complete process is:

```text
Start
  ↓
Parse CLI arguments
  ↓
Generate backup filename if required
  ↓
Connect to Redis
  ↓
Connection successful?
  ├── No → Exit
  └── Yes
        ↓
Backup matching Redis keys
        ↓
Backup successful?
  ├── No → Abort
  └── Yes
        ↓
Delete matching Redis keys
        ↓
Check --delete_config_path
        ↓
Path supplied?
  ├── No → Skip
  └── Yes
        ↓
Attempt configuration-file deletion
        ↓
Print completion message
        ↓
Exit
```

The actual main execution sequence is implemented in `main()`.

---

## 30. Example: Backup and Delete Redis Keys

```bash
python redis_cache_backup_and_deletion.py \
  --host localhost \
  --port 6379 \
  --key_pattern "my_app:*"
```

If `--backup_file` is not supplied, the script generates a timestamped JSON filename.

Example result:

```text
backup/
└── redis_backup_20260917_110530.json
```

The matching keys are then deleted.

---

## 31. Example: Redis Password

```bash
python redis_cache_backup_and_deletion.py \
  --host localhost \
  --port 6379 \
  --password "<redis_password>" \
  --key_pattern "my_app:*"
```

The password is passed to the Redis client during connection.

---

## 32. Example: Custom Backup Filename

```bash
python redis_cache_backup_and_deletion.py \
  --key_pattern "my_app:*" \
  --backup_file my_app_redis_backup.json
```

The resulting file is stored as:

```text
backup/
└── my_app_redis_backup.json
```

---

## 33. Example: Delete All Matching Keys

To process all Redis keys:

```bash
python redis_cache_backup_and_deletion.py \
  --key_pattern "*"
```

> ⚠️ **Extreme caution:** `*` can match a very large number of Redis keys. Verify the pattern before execution.

---

## 34. Example: Delete Redis Configuration File

```bash
python redis_cache_backup_and_deletion.py \
  --key_pattern "my_app:*" \
  --delete_config_path "/path/to/redis.conf"
```

The script will:

1. Connect to Redis
2. Back up matching keys
3. Delete matching keys
4. Attempt to delete the specified configuration file

The configuration file deletion is attempted only after the backup stage succeeds.

---

## 35. Operational Safety Checklist

Before executing the script:

* [ ] Confirm the Redis host.
* [ ] Confirm the Redis port.
* [ ] Confirm the Redis credentials if authentication is enabled.
* [ ] Carefully verify `--key_pattern`.
* [ ] Confirm the backup directory is writable.
* [ ] Verify that the backup file is created successfully.
* [ ] Do not use `*` unless deleting all matching Redis keys is explicitly intended.
* [ ] If using `--delete_config_path`, verify the exact filesystem path.
* [ ] Confirm that configuration-file deletion is required.
* [ ] Ensure the operator has the required filesystem permissions.

---

## 36. Important Behavior

### Backup Before Deletion

Redis key deletion is not performed when the backup operation fails.

```text
Backup Failed
     ↓
Abort
     ↓
No Key Deletion
```

### Same Pattern for Backup and Deletion

The `--key_pattern` determines both the keys that are backed up and the keys that are subsequently deleted.

### Configuration Deletion Is Separate

The optional `--delete_config_path` operation is a filesystem operation and is separate from Redis key deletion.

### No Restore Function

This script creates a JSON backup but does not contain functionality to restore the backup into Redis.

---

## 37. Runtime Dependencies

### Python Modules

The script uses:

```text
redis
argparse
json
os
sys
datetime
base64
```

The `redis` Python package is required because the script creates and uses a Redis client.

The remaining modules used by the script are part of the Python standard library.

---

## 38. Project Configuration Dependency

The script imports:

```python
from lib.global_config import get
```

and uses the global configuration to obtain Redis defaults:

```text
redis.host
redis.port
```

The fallback values are:

```text
Host: localhost
Port: 6379
```

The Redis password is not loaded from `global_config` by this script; the CLI argument defaults to `None`.

---

## 39. Function Reference

| Function                     | Responsibility                                 |
| ---------------------------- | ---------------------------------------------- |
| `parse_arguments()`          | Parses command-line arguments                  |
| `connect_to_redis()`         | Creates and validates the Redis connection     |
| `decode_or_base64()`         | Converts Redis bytes to UTF-8 or Base64        |
| `backup_redis_keys()`        | Backs up matching Redis keys to JSON           |
| `delete_redis_keys()`        | Deletes matching Redis keys                    |
| `delete_redis_config_file()` | Deletes the specified Redis configuration file |
| `main()`                     | Coordinates the complete operation             |

---

## 40. Error Handling Summary

| Failure                   | Behavior                       |
| ------------------------- | ------------------------------ |
| Redis connection failure  | Script exits                   |
| Redis backup failure      | Key deletion is aborted        |
| Backup file write failure | Backup returns failure         |
| Unsupported Redis type    | Key is skipped                 |
| No matching deletion keys | Reports zero deleted           |
| Redis deletion error      | Reports error and returns zero |
| Config file missing       | Config deletion skipped        |
| Config permission error   | Reports failure                |
| Unexpected exception      | Reports error                  |

---

## 41. Backup and Deletion Result

A successful run produces:

```text
Redis connection
      ↓
JSON backup
      ↓
Matching Redis keys deleted
      ↓
Optional configuration-file deletion
      ↓
Redis operations completed.
```

The final message is:

```text
Redis operations completed.
```

This message indicates that the main workflow reached its end; the operator should review preceding output to confirm whether Redis keys and any requested configuration file were actually deleted successfully.

---

## 42. Summary

`redis_cache_backup_and_deletion.py` is a Redis maintenance utility that combines backup and controlled deletion operations.

Its main characteristics are:

* Requires a Redis key pattern.
* Connects to Redis using configurable host and port.
* Uses `SCAN` to discover matching keys.
* Supports string, hash, list, set, and sorted-set values.
* Handles binary data using UTF-8 decoding and Base64 encoding.
* Stores backups as JSON files.
* Automatically creates the `backup/` directory.
* Generates timestamped backup filenames when none are supplied.
* Deletes matching Redis keys using a Redis pipeline.
* Stops key deletion when the backup fails.
* Optionally deletes a specified Redis configuration file.
* Does not provide Redis restore functionality.
* Does not contain its own scheduling mechanism.

Because the script performs destructive Redis and filesystem operations, the key pattern and optional configuration-file path should be verified carefully before execution.
