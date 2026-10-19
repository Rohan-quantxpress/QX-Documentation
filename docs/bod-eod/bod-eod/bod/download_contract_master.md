# Download Contract Master

## Script: `download_contract_master.py`

### 1. Purpose

`download_contract_master.py` is a **reference-data download and preparation script** used during the BOD workflow.

Its responsibility is to obtain the NSE contract and security master files from the **authenticated NSE Extranet service**, prepare the downloaded files locally, and store the raw compressed data in Redis.

These files provide reference information required by downstream processes that work with NSE instruments and contracts.

The overall flow is:

```text
NSE Extranet
     ↓
Authenticated Login
     ↓
Download Reference Files
     ↓
security.gz
contract.gz
     ↓
Extract Files
     ↓
Local Reference Files
     ↓
Redis Raw Data
     ↓
Downstream BOD Processes
```

---

## 2. Why This Script Is Important

Contract and security master files provide reference information used by other components of the trading system.

Downstream processes may depend on this information to work with NSE instruments, contracts, and their associated reference data.

Therefore, this script forms an important part of the **BOD reference-data preparation stage**.

It should be completed successfully before processes that depend on these files or their corresponding Redis data are executed.

---

## 3. NSE Data Source

The script obtains the files from the **NSE Extranet API**.

Unlike publicly accessible NSE market-data endpoints, this service requires authentication using NSE member credentials.

### API Services Used

The script uses the following NSE Extranet services:

```text
Base URL
https://www.connect2nse.com/extranet-api
```

```text
Login
/login/2.0

File Download
/common/file/download/2.0
```

The login service is used to obtain an authentication token, which is then used for downloading the required files.

---

## 4. Files Downloaded

The script downloads two reference files:

| File          | Segment | Location  | Purpose                         |
| ------------- | ------- | --------- | ------------------------------- |
| `security.gz` | `CM`    | `/ntneat` | NSE security master information |
| `contract.gz` | `CM`    | `/ntneat` | NSE contract master information |

The files are downloaded individually.

The execution flow is:

```text
Login
  ↓
Download security.gz
  ↓
Process security.gz
  ↓
Download contract.gz
  ↓
Process contract.gz
  ↓
BOD Reference Data Ready
```

If a required download fails after the available retry attempts, the script stops instead of continuing with the next file.

---

## 5. Authentication

Authentication is required before downloading the files.

The script receives the following NSE credentials as command-line parameters:

* `member_code`
* `login_id`
* `encrypted_password`

After successful authentication, NSE returns an authentication token.

The token is then used for subsequent file-download requests.

```text
NSE Credentials
      ↓
NSE Login
      ↓
Authentication Token
      ↓
Authorized File Download
```

### Security Consideration

NSE authentication credentials should be handled securely.

They should not be hardcoded into the script or committed to source control.

---

## 6. File Download and Retry Behavior

The script downloads each file from the configured NSE Extranet location.

If a download fails, the script retries the operation.

The current implementation allows up to **three download attempts** for each file.

```text
Download Attempt
      ↓
   Success?
   /      \
 Yes       No
 ↓          ↓
Continue   Retry
            ↓
        Up to 3 Attempts
```

If the file cannot be downloaded successfully after the available attempts, the process reports the failure.

---

## 7. File Preparation

The downloaded files are compressed using the `.gz` format.

The script extracts the compressed file into a usable text file.

For example:

```text
security.gz
    ↓
security.txt
```

and:

```text
contract.gz
    ↓
contract.txt
```

The extracted files are stored in the configured download location used by the process.

This allows downstream processes to consume the reference data without having to handle the compressed source files themselves.

---

## 8. Redis Storage

In addition to extracting the downloaded files locally, the script stores the **raw compressed file content in Redis**.

The Redis connection currently uses:

```text
Host: localhost
Port: 6379
DB: 0
```

The Redis key is generated from the downloaded filename.

For example:

```text
security.gz
    ↓
SECURITY_GZ
```

and:

```text
contract.gz
    ↓
CONTRACT_GZ
```

The existing Redis key is cleared before the new file content is stored.

This provides the system with a Redis copy of the raw downloaded data.

---

## 9. BOD Role

This script belongs to the **NSE reference-data preparation stage** of BOD.

A simplified BOD dependency can be represented as:

```text
BOD Starts
    ↓
Authenticate with NSE
    ↓
Download Security Master
    ↓
Prepare Security Data
    ↓
Download Contract Master
    ↓
Prepare Contract Data
    ↓
Reference Data Available
    ↓
Dependent BOD Processes
```

The exact position of this process in BOD should be determined by the processes that consume the generated files or Redis data.

---

## 10. Runtime Inputs

The script requires the following command-line arguments:

| Argument               | Required | Purpose                       |
| ---------------------- | -------- | ----------------------------- |
| `--member_code`        | Yes      | NSE member code               |
| `--login_id`           | Yes      | NSE login ID                  |
| `--encrypted_password` | Yes      | NSE encrypted password        |
| `--download_folder`    | No       | Download-folder configuration |

The first three parameters are required for authentication.

If the required authentication parameters are not supplied, the script cannot perform the NSE login and the process terminates.

---

## 11. Dependencies

The script depends on several external components.

### NSE Extranet

The NSE Extranet service must be accessible from the machine running the script.

The script requires:

* Network connectivity
* Valid NSE credentials
* Successful NSE authentication
* Access to the required contract-master files

### Local File System

The script requires permission to:

* Create the download directory
* Write downloaded files
* Write extracted files

### Redis

Redis must be available for storing the downloaded compressed data.

The configured Redis instance must be reachable from the machine running the script.

---

## 12. Common Failure Scenarios

### Authentication Failure

If NSE authentication fails:

```text
Login
  ↓
Authentication Failed
  ↓
Process Stops
```

Check:

* Member code
* Login ID
* Encrypted password
* NSE Extranet availability

---

### File Download Failure

If a file cannot be downloaded, the script retries the download.

If all attempts fail, the process reports the failure.

Check:

* Network connectivity
* NSE Extranet availability
* Authentication status
* Requested file availability

---

### File Extraction Failure

If the downloaded compressed file cannot be processed, the extracted output file may not be available.

Check the downloaded file and the process logs.

---

### Redis Failure

If Redis is unavailable, the Redis storage operation cannot be completed.

Check:

* Redis service status
* Redis host
* Redis port
* Redis connectivity

---

## 13. What to Verify After Execution

After the process completes, verify:

* NSE authentication was successful
* `security.gz` was downloaded
* `contract.gz` was downloaded
* Extracted files are present
* Expected files are available in the download directory
* Redis contains the expected raw-data keys
* No download or extraction errors are present in the logs

Expected Redis keys include:

```text
SECURITY_GZ
CONTRACT_GZ
```

---

## 14. Operational Flow

The complete operation can be summarized as:

```text
Start
  ↓
Read NSE Credentials
  ↓
Login to NSE Extranet
  ↓
Receive Authentication Token
  ↓
Download security.gz
  ↓
Extract Security File
  ↓
Store Raw Security Data in Redis
  ↓
Download contract.gz
  ↓
Extract Contract File
  ↓
Store Raw Contract Data in Redis
  ↓
Complete
```

---

## Key Takeaway

`download_contract_master.py` is responsible for **acquiring and preparing NSE contract and security reference data for the BOD workflow**.

It connects the external NSE Extranet with the internal file system and Redis layer:

```text
NSE Extranet
      ↓
security.gz / contract.gz
      ↓
Local Extracted Files
      +
Redis Raw Data
      ↓
Downstream BOD Processes
```

The most important operational dependencies are **NSE authentication, NSE file availability, local file-system access, and Redis availability**.

A successful execution ensures that the required NSE reference files have been downloaded and prepared for the processes that depend on them.
