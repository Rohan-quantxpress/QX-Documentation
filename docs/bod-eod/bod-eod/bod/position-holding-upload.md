# Upload Futures and Options Positions

## Script: `upload_fno_position.py`

---

## 1. Purpose

`upload_fno_position.py` is a **BOD position and holding upload script** responsible for reading the daily position and holding files, matching them with the instrument master and registered users, calculating the required position values, and uploading the resulting records into the MySQL `InstrumentStatistics` table.

The script processes:

* Position data
* Holding data
* Instrument master data
* User/entity information from `AspNetUsers`

Both position and holding information are ultimately stored in `InstrumentStatistics`.

---

## 2. Overall Execution Flow

```text
Start
  ↓
Read Configuration
  ↓
Build Daily File Paths
  ↓
Validate Position File
  ↓
Validate Holding File
  ↓
Validate Instrument Master
  ↓
Load Instrument Master
  ↓
Connect to MySQL
  ↓
Load Users from AspNetUsers
  ↓
Process Position File
  ↓
Match User + Instrument
  ↓
Calculate Position Values
  ↓
Insert / Update InstrumentStatistics
  ↓
Process Holding File
  ↓
Match User + Instrument
  ↓
Calculate Holding Values
  ↓
Insert / Update InstrumentStatistics
  ↓
Close Database Connection
```

The script stops before processing if any required input file is missing or if the database connection cannot be established.

---

# Configuration

## 3. Default File Locations

Position and holding files are expected in:

```text
/home/Blitz/BOD/QX.ProcessManager/downloads
```

The default instrument master is:

```text
./InstrumentMaster/Instruments/instruments.csv
```

These defaults can be overridden using command-line arguments.

---

## 4. Daily File Names

The script generates the default filenames using the current system date in:

```text
YYYYMMDD
```

### Position File

```text
Bull8_POSYYYYMMDD.CSV
```

Example:

```text
Bull8_POS20260917.CSV
```

### Holding File

```text
Bull8_HOLDINGYYYYMMDD.CSV
```

Example:

```text
Bull8_HOLDING20260917.CSV
```

Custom position and holding files can be supplied using:

```text
--position-file
--holding-file
```

---

# Instrument and User Mapping

## 5. Instrument Master

The script loads the configured instrument master CSV using pandas.

The instrument master is used to obtain information such as:

* `InstrumentId`
* `InstrumentName`
* `ExchangeSegment`
* `LotSize`
* `ISIN`

If the instrument file cannot be loaded, the script exits.

---

## 6. User Mapping

The script loads registered users from:

```text
AspNetUsers
```

The database query retrieves:

```sql
SELECT Id, ClientId
FROM AspNetUsers
```

The `Id` field is treated as the:

```text
EntityId
```

This creates the following mapping:

```text
ClientId → EntityId
```

The mapping is used to associate position and holding records with the corresponding application entity.

If no users are found, processing stops.

---

# Position Processing

## 7. Position File

The position file is pipe-separated and follows the expected position structure.

The expected fields are:

| Field                   |
| ----------------------- |
| `BRANCHID`              |
| `DEALERID`              |
| `CLIENTID`              |
| `EXCHANGESEG`           |
| `SERIES`                |
| `SYMBOL`                |
| `EXPIRYDATE`            |
| `STRIKEPRICE`           |
| `OPTIONTYPE`            |
| `BUYQTY`                |
| `BUYAMT`                |
| `SELLQTY`               |
| `SELLAMT`               |
| `PRODUCTTYPE`           |
| `Actual Buy Avg Price`  |
| `Actual Sell Avg Price` |

The script checks whether the file already contains the expected header. If not, it assigns the expected header structure to the data.

---

## 8. Position Data Preparation

Before processing, numeric fields such as:

* `STRIKEPRICE`
* `BUYQTY`
* `SELLQTY`
* `BUYAMT`
* `SELLAMT`
* `Actual Buy Avg Price`
* `Actual Sell Avg Price`

are converted to appropriate numeric values.

The expiry date is interpreted using:

```text
%d-%b-%y
```

---

## 9. Position Instrument Name

The script creates an `InstrumentName` from the position information.

For futures, the format is:

```text
SYMBOL + EXPIRY + FUT
```

Example:

```text
NIFTY25SEP26FUT
```

For options, the format is:

```text
SYMBOL + EXPIRY + STRIKEPRICE + OPTIONTYPE
```

Example:

```text
NIFTY25SEP2625000CE
```

The script supports the following series patterns:

### Futures

```text
FUT
IF
SF
```

### Options

```text
OPT
IO
SO
```

---

## 10. Position Exchange Mapping

The input exchange segment is converted to its internal numeric identifier.

| Exchange           |   ID |
| ------------------ | ---: |
| `nse_cm` / `NSECM` |  `1` |
| `nse_fo` / `NSEFO` |  `2` |
| `nse_cd` / `NSECD` |  `3` |
| `bse_cm` / `BSECM` | `11` |
| `bse_fo` / `BSEFO` | `12` |
| `bse_cd` / `BSECD` | `13` |
| `mcx_fo` / `MCXFO` | `14` |

Unknown exchange values are mapped to:

```text
0
```

---

## 11. Position User Matching

The position `CLIENTID` is matched against `ClientId` from `AspNetUsers`.

```text
Position CLIENTID
       ↓
AspNetUsers.ClientId
       ↓
EntityId
```

A position without a matching user entity is excluded from the upload.

---

## 12. Position Instrument Matching

Position records are matched against the instrument master using:

```text
InstrumentName
+
ExchangeSegment
```

The matching provides:

* `InstrumentId`
* `InstrumentName`
* `ExchangeSegment`
* `LotSize`

A position for which the instrument cannot be found is excluded from the upload.

---

## 13. Skipped Position Records

A position record is skipped when:

* The corresponding `EntityId` cannot be found.
* The corresponding `InstrumentId` cannot be found.

Skipped records are written to:

```text
upload_fno_position.log
```

The skip reason identifies whether the user entity or instrument was missing.

---

# Position Calculations

## 14. Position Values

The script derives the following values:

```text
LongPosition  = BUYQTY
ShortPosition = SELLQTY
NetPosition   = BUYQTY - SELLQTY
```

Open quantities are initialized to:

```text
OpenBuyQuantity  = 0
OpenSellQuantity = 0
```

---

## 15. Net Position in Lots

The net position is converted into lots using the instrument's `LotSize`:

```text
NetPositionInLot = NetPosition / LotSize
```

The resulting value is stored as an integer.

---

## 16. Position Trade Values

The following values are populated from the position file:

```text
AvgBuy         = Actual Buy Avg Price
AvgSell        = Actual Sell Avg Price
TradeBuyValue  = BUYAMT
TradeSellValue = SELLAMT
```

The net trade value is calculated as:

```text
TradeNetValue = TradeSellValue - TradeBuyValue
```

---

## 17. Position Average and Exposure

The script calculates:

```text
AvgNet = AvgSell - AvgBuy
```

Exposure is calculated as:

```text
Exposure = NetPosition × AvgSell
```

---

## 18. RMS Entity Code

For each position record, the script creates:

```text
CLIENTID|ALL|ALL|ALL
```

Example:

```text
CLIENT001|ALL|ALL|ALL
```

This value is stored as:

```text
RMSEntityCode
```

---

## 19. Position Turnover

Position turnover is calculated as:

```text
TurnoverValue = BUYAMT + SELLAMT
```

---

# Holding Processing

## 20. Holding File

The script supports holding files in:

* `.CSV`
* `.XLS`
* `.XLSX`

It can also process pipe-separated holding data.

The required holding information is normalized into:

```text
UserID
isin
quantity
buy_avg_price
```

---

## 21. Holding Column Mapping

For header-based files, the script supports multiple names for the same logical field.

| Source Field    | Internal Field  |
| --------------- | --------------- |
| `ACCOUNT ID`    | `UserID`        |
| `ACCOUNTID`     | `UserID`        |
| `USERID`        | `UserID`        |
| `ISIN`          | `isin`          |
| `QUANTITY`      | `quantity`      |
| `QTY`           | `quantity`      |
| `BUYAVGPRICE`   | `buy_avg_price` |
| `BUY AVG PRICE` | `buy_avg_price` |
| `AVERAGE PRICE` | `buy_avg_price` |

This allows the script to process holding files with slightly different column naming conventions.

---

## 22. Holding Data Cleaning

The script converts:

```text
quantity
buy_avg_price
```

to numeric values.

Invalid numeric values are converted to:

```text
0
```

`UserID` and `ISIN` values are converted to strings and stripped of surrounding whitespace.

Records with:

```text
quantity <= 0
```

are removed.

---

# Holding Instrument Matching

## 23. NSE Instrument Matching

Holding records are matched using their ISIN.

The script first searches instruments belonging to:

```text
ExchangeSegment = 1
```

which represents:

```text
NSECM
```

---

## 24. BSE Fallback

If a holding's ISIN cannot be found in NSECM, the script searches:

```text
ExchangeSegment = 11
```

which represents:

```text
BSECM
```

Therefore, the matching sequence is:

```text
Holding ISIN
     ↓
Search NSECM
     ↓
Not Found
     ↓
Search BSECM
```

This allows equity holdings to be matched against either NSE or BSE instrument records.

---

## 25. Holding User Matching

Holding records are matched with `AspNetUsers` using:

```text
UserID → ClientId
```

The corresponding:

```text
EntityId
```

is then attached to the holding record.

---

## 26. Skipped Holding Records

A holding record is skipped when:

* `EntityId` cannot be found.
* `InstrumentId` cannot be found.

Skipped records are written to:

```text
upload_holding_skip.log
```

The log contains the corresponding skip reason.

---

# Holding Calculations

## 27. Buy and Sell Quantities

For holding data, the script treats the holding as a buy-side position:

```text
BUYQTY  = quantity
SELLQTY = 0
```

The buy amount is:

```text
BUYAMT = quantity × buy_avg_price
```

The sell amount is:

```text
SELLAMT = 0
```

---

## 28. Holding Position Values

The script calculates:

```text
LongPosition  = BUYQTY
ShortPosition = 0
NetPosition   = BUYQTY
```

Open quantities are initialized to:

```text
OpenBuyQuantity  = 0
OpenSellQuantity = 0
```

Net position in lots is calculated using:

```text
NetPositionInLot = NetPosition / LotSize
```

---

## 29. Holding Average and Trade Values

For holding records:

```text
AvgBuy         = buy_avg_price
AvgSell        = 0
TradeBuyValue  = BUYAMT
TradeSellValue = 0
```

The script calculates:

```text
TradeNetValue = -BUYAMT
AvgNet        = -buy_avg_price
```

Exposure is:

```text
Exposure = NetPosition × AvgBuy
```

---

## 30. Holding RMS Entity and Turnover

The RMS entity code is:

```text
UserID|ALL|ALL|ALL
```

Example:

```text
CLIENT001|ALL|ALL|ALL
```

Holding turnover is:

```text
TurnoverValue = BUYAMT
```

---

# Database Upload

## 31. Target Table

Both position and holding records are uploaded to:

```text
InstrumentStatistics
```

The same database insertion mechanism is used for both types of data.

---

## 32. Insert / Update Behavior

The database operation uses:

```sql
ON DUPLICATE KEY UPDATE
```

Therefore, if a matching record already exists, selected position fields are updated instead of creating a duplicate record.

Updated fields include:

```text
NetPosition
NetPositionInLot
TradeBuyValue
TradeSellValue
TradeNetValue
LongPosition
ShortPosition
```

`OpenBuyQuantity` and `OpenSellQuantity` are incremented using their existing values plus the incoming values.

---

## 33. Batch Processing

Database records are processed in batches of:

```text
5000
```

Each batch is inserted using `executemany()` and committed to the database.

If an insertion error occurs, the current transaction is rolled back and the error is logged.

---

# Command-Line Arguments

## 34. Database Arguments

| Argument     | Default                          | Description    |
| ------------ | -------------------------------- | -------------- |
| `--user`     | From global config / `root`      | MySQL username |
| `--password` | From global config / `root`      | MySQL password |
| `--host`     | From global config / `127.0.0.1` | MySQL host     |
| `--database` | From global config / `Blitz`     | Database name  |
| `--port`     | From global config / `3306`      | MySQL port     |

---

## 35. File Arguments

| Argument            | Default                                          | Description       |
| ------------------- | ------------------------------------------------ | ----------------- |
| `--position-file`   | Today's `Bull8_POSYYYYMMDD.CSV`                  | Position file     |
| `--holding-file`    | Today's `Bull8_HOLDINGYYYYMMDD.CSV`              | Holding file      |
| `--instrument-file` | `./InstrumentMaster/Instruments/instruments.csv` | Instrument master |

If the file arguments are not supplied, the script automatically builds the daily filenames using the current date.

---

## 36. Example

Using the default daily files:

```bash
python upload_holding_position_db.py
```

Using custom files:

```bash
python upload_holding_position_db.py \
    --position-file /data/Bull8_POS20260917.CSV \
    --holding-file /data/Bull8_HOLDING20260917.CSV \
    --instrument-file ./InstrumentMaster/Instruments/instruments.csv
```

Using custom database settings:

```bash
python upload_holding_position_db.py \
    --host 127.0.0.1 \
    --port 3306 \
    --user root \
    --password root \
    --database Blitz
```

---

# Validation and Error Handling

## 37. Required File Validation

Before processing begins, the script verifies that the following files exist:

```text
Position File
Holding File
Instrument Master
```

If any required file is missing, the script exits with status:

```text
1
```

---

## 38. Database Connection

The script connects to MySQL using the configured database parameters.

If the connection fails:

```text
Database connection failed
```

is logged and processing stops.

---

## 39. Empty User Database

If no records are found in `AspNetUsers`, the script does not continue with position and holding uploads.

This prevents records from being uploaded without a valid application entity mapping.

---

## 40. Skipped Data

Individual position or holding records can be skipped when their user or instrument mapping is unavailable.

These records do not stop the entire upload process.

Instead, they are written to the appropriate skip log for investigation.

---

# Logging

## 41. Log Files

The main process log is:

```text
holding_position_upload.log
```

Position-specific skipped records are logged to:

```text
upload_fno_position.log
```

Holding-specific skipped records are logged to:

```text
upload_holding_skip.log
```

The logs provide information about:

* File processing
* Instrument loading
* User loading
* Database connection
* Skipped records
* Insert progress
* Processing errors
* Completion status

---

# Operational Verification

## 42. Pre-BOD Verification

Before running the script, verify:

* Position file is available.
* Holding file is available.
* Instrument master is available.
* Database is reachable.
* `AspNetUsers` contains the required client mappings.
* Input files correspond to the required trading date.

---

## 43. Post-Execution Verification

After execution, check for:

```text
All files processed successfully!
```

Also verify:

* Position records were uploaded.
* Holding records were uploaded.
* Unexpected skipped records are investigated.
* `InstrumentStatistics` contains the expected records.
* No significant instrument or entity mapping failures occurred.

---

## 44. Database Verification

To verify uploaded records:

```sql
SELECT COUNT(*)
FROM InstrumentStatistics;
```

To inspect client-level records:

```sql
SELECT
    RMSEntityCode,
    InstrumentId,
    ClientId,
    ExchangeSegment,
    InstrumentName,
    LongPosition,
    ShortPosition,
    NetPosition,
    AvgBuy,
    AvgSell,
    Exposure
FROM InstrumentStatistics
LIMIT 20;
```

---

# Key Takeaway

`upload_holding_position_db.py` prepares the daily **position and holding state** required by the RMS/trading system.

The complete flow is:

```text
Position File ─────────┐
                       │
                       ▼
                 Read & Prepare
                       │
                       ▼
Instrument Master ──► Match Instrument
                       │
AspNetUsers ────────► Match Entity
                       │
                       ▼
                 Calculate Values
                       │
                       ▼
              InstrumentStatistics
                       ▲
                       │
                 Calculate Values
                       ▲
                       │
Holding File ──────────┘
```

The script uses the instrument master and `AspNetUsers` to enrich both position and holding data before performing an **insert/update operation on `InstrumentStatistics`**.

Position and holding records are processed separately but ultimately contribute to the same database table, allowing the system to maintain the required client-level instrument statistics for BOD processing.
