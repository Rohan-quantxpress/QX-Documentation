# Signals

Send trading or strategy signals from a source strategy to destination strategies within the platform.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    signals = [
        {
            "ID":"BLT_123",
            "SourceStrategy": "MyStrategy",
            "DestinationStrategy": "TargetStrategy",
            "SourceSID": "inst_001",
            "InstanceRunningMode": "Started",
            "GlobalAction": "Signal",
            "Instruments": [
                {
                    "ExchangeSegment": "NSEFO",
                    "InstrumentName": "NIFTY26FEB2625500PE",
                    "Action": "BUY",
                    "Lot": "1",
                    "TimeStamp": "06-07-2026 09:15:00",
                    "InfoText": "Entry Signal"
                }
            ]
        }
    ]

    response = client.send_signals(signals)
    print(response)
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var signals = new List<SignalRequest>
    {
        new SignalRequest
        {
            SourceStrategy = "MyStrategy",
            DestinationStrategy = "TargetStrategy",
            SourceSID = "inst_001",
            InstanceRunningMode = "Started",
            GlobalAction = "Signal",
            Instruments = new List<SignalInstrument>
            {
                new SignalInstrument
                {
                    ExchangeSegment = "NSEFO",
                    InstrumentName = "NIFTY26FEB2625500PE",
                    Action = "BUY",
                    Lot = "1",
                    TimeStamp = "06-07-2026 09:15:00",
                    InfoText = "Entry Signal"
                }
            }
        }
    };

    var response = await client.SendSignalsAsync(signals);
    Console.WriteLine(response);
    ```

---

## Endpoint

```
POST /signals
```

### Request

```bash
curl -X POST "https://{baseurl}/api_interactive/api/v1/signals" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '[
    {
        "SourceStrategy": "MyStrategy",
        "DestinationStrategy": "TargetStrategy",
        "SourceSID": "inst_001",
        "InstanceRunningMode": "Started",
        "GlobalAction": "Signal",
        "Instruments": [
            {
                "ExchangeSegment": "NSEFO",
                "InstrumentName": "NIFTY26FEB2625500PE",
                "Action": "BUY",
                "Lot": "1",
                "TimeStamp": "06-07-2026 09:15:00",
                "InfoText": "Entry Signal"
            }
        ]
    }
]'
```

### Request parameters

| Field | Type | Description |
|-------|------|-------------|
| `SourceStrategy` | string | Name of the source strategy |
| `DestinationStrategy` | string | Name of the destination strategy |
| `SourceSID` | string | Source instance ID |
| `ID` | string | Unique identifier for the signal (optional) |
| `SL` | string | Stop-loss value for the signal (optional) |
| `InstanceRunningMode` | string | Running mode (`Started`, `Stopped`) |
| `GlobalAction` | string | Action type (`Signal`, `Start`, `Stop`) |
| `Instruments` | object[] | List of instrument actions |

### Response

```json
{
    "status": "success",
    "message": "request processed successfully.",
    "data": {
        "message": "Signal received successfully."
    }
}
```

### Instrument action fields

| Field | Type | Description |
|-------|------|-------------|
| `ExchangeSegment` | string | Exchange segment (e.g., `NSECM`, `NSEFO`, `BSECM`, `BSEFO`) |
| `InstrumentName` | string | Instrument name / symbol |
| `Action` | string | Action (`ENTERLONG`, `BUY`, `ENTERSHORT`, `SELL`, `GOFLAT`, `SETPOSITION`) |
| `Lot` | string | Number of lots |
| `TimeStamp` | string | Signal timestamp (`DD-MM-YYYY HH:mm:ss`) |
| `InfoText` | string | Additional signal information |
