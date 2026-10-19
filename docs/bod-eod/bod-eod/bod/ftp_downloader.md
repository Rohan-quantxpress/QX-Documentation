## Script: `ftp_downloader.py`

### 1. Purpose

`ftp_downloader.py` is a **generic file-download utility** used to retrieve a file from a remote FTP server and save it to a specified local location.

The script is designed to be reusable across BOD/EOD processes where required files are provided through an FTP server.

It does not contain exchange-specific or business-specific processing logic. Its responsibility is limited to **retrieving the required file and making it available locally**.

```
text id="5y0m2x"
Remote FTP Server
       ↓
FTP Authentication
       ↓
Locate Remote File
       ↓
Download File
       ↓
Local File System
       ↓
Downstream Process
```

---

## 2. Role in BOD / EOD

When a BOD or EOD process depends on a file hosted on an FTP server, this utility can be used to bring that file into the local environment.

The typical workflow is:

```
text id="6f8b9c"
FTP Server
    ↓
ftp_downloader.py
    ↓
Local File
    ↓
File Processing Script
    ↓
BOD / EOD Processing
```

The downloaded file can then be consumed by another process registered in the Process Manager.

---

## 3. How the Script Works

The script follows a simple download flow:

1. Receives the FTP connection details.
2. Connects to the FTP server.
3. Authenticates using the supplied username and password.
4. Opens the destination file locally.
5. Retrieves the specified remote file.
6. Saves the downloaded content to the local path.
7. Closes the FTP connection.

```
text id="kq0z4h"
Start
  ↓
Read FTP Details
  ↓
Connect to FTP Server
  ↓
Login
  ↓
Open Local Destination
  ↓
Download Remote File
  ↓
Save File
  ↓
Close FTP Connection
  ↓
Complete
```

---

## 4. Remote File

The remote file is provided through the `--remote-file` parameter.

The path must identify the file that should be retrieved from the FTP server.

For example:

```
text id="l0s4x7"
/incoming/data/file.csv
```

The remote file must exist and the configured FTP user must have permission to retrieve it.

If the remote file cannot be accessed, the download will fail.

---

## 5. Local File

The destination is provided through the `--local-file` parameter.

This defines exactly where the downloaded file will be stored.

For example:

```
text id="xq4k8p"
C:\BOD\downloads\file.csv
```

If the parent directory does not exist, the script attempts to create it before starting the download.

This makes the utility suitable for automated workflows where the required local directory may not already exist.

---

## 6. FTP Authentication

The script requires standard FTP username/password authentication.

The required connection information is:

* FTP host
* FTP username
* FTP password
* Remote file path
* Local file path

The authentication flow is:

```
text id="0x3n8r"
FTP Host
   ↓
Username + Password
   ↓
FTP Login
   ↓
Authenticated Session
   ↓
File Download
```

The credentials should be supplied securely and should not be hardcoded into the source code or committed to the project repository.

---

## 7. Runtime Parameters

The script requires all five command-line parameters.

| Argument        | Required | Purpose                                |
| --------------- | -------- | -------------------------------------- |
| `--host`        | Yes      | FTP server hostname or IP address      |
| `--user`        | Yes      | FTP username                           |
| `--password`    | Yes      | FTP password                           |
| `--remote-file` | Yes      | Remote file to download                |
| `--local-file`  | Yes      | Local location where the file is saved |

Example structure:

```
text id="2q0m7w"
ftp_downloader.py
    ↓
--host
--user
--password
--remote-file
--local-file
```

If a required parameter is missing, the command-line argument parser stops execution.

---

## 8. File Handling

The downloaded file is written in **binary mode**.

This allows the utility to download different types of files without treating their contents as text.

The script itself does not parse, transform, validate, or modify the downloaded file.

Its responsibility ends after the file has been retrieved and written to the configured local location.

---

## 9. Error Handling

The script handles common FTP and local file-system errors.

### FTP Connection Failure

If the FTP server cannot be reached, the connection fails and the download does not proceed.

Possible causes include:

* FTP server unavailable
* Network connectivity issue
* Incorrect hostname/IP
* FTP service unavailable

### Authentication Failure

If the username or password is incorrect, the FTP login fails and the file is not downloaded.

### Remote File Failure

If the requested remote file does not exist or cannot be retrieved, the FTP server returns an error.

### Local File Failure

If the destination cannot be created or written, the download cannot be completed.

Possible causes include:

* Invalid local path
* Insufficient permissions
* Insufficient disk space
* Directory creation failure

---

## 10. Important Operational Limitation

The current implementation does **not** provide:

* Automatic retry
* Download resume
* Checksum validation
* File-content validation
* Duplicate-file validation

Therefore, if a connection is interrupted during transfer, the resulting local file should be checked before it is consumed by a downstream process.

For important BOD/EOD files, operational verification is recommended after download.

---

## 11. What to Verify

When this script is used as part of a BOD/EOD workflow, verify:

### FTP Side

* FTP server is reachable
* Credentials are valid
* User has access to the required file
* Remote file exists
* Remote path is correct

### Local Side

* Destination directory is writable
* Sufficient disk space is available
* Downloaded file exists
* Downloaded filename is correct
* File size is reasonable for the expected file

### Downstream Side

* The next process can access the downloaded file
* The file is located where the downstream process expects it
* No additional file movement or transformation is required

---

## 12. Operational Flow

The complete role of the utility can be summarized as:

```
text id="w9f5c3"
Remote FTP File
      ↓
FTP Connection
      ↓
Authentication
      ↓
File Retrieval
      ↓
Local File
      ↓
Verification
      ↓
Downstream BOD/EOD Process
```

The utility provides the **file-transfer layer**, while the downstream script is responsible for processing the downloaded file.

---

## Key Takeaway

`ftp_downloader.py` is a **generic FTP file-transfer utility**.

Its job is straightforward:

```
text id="3h8m5q"
Connect
  ↓
Authenticate
  ↓
Download
  ↓
Save Locally
  ↓
Close Connection
```

It does not perform business-specific processing or database operations. It simply makes a remote FTP file available in the local file system so that other BOD/EOD processes can use it.

Its successful operation depends primarily on **FTP availability, valid credentials, remote-file availability, network stability, and local file-system access**.
