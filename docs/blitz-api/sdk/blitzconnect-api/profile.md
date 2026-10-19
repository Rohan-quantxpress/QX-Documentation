# Profile

Retrieve the authenticated user's profile.

=== "Python"

    ```python
    from blitzsdk import InteractiveApiClient

    client = InteractiveApiClient(app_key="YOUR_KEY", user_id="YOUR_USER")

    profile = client.get_profile()
    print(profile["ClientId"], profile["UserName"], profile["IsKYC"])
    ```

=== "C#"

    ```csharp
    using BlitzConnect;
    using BlitzConnect.Services;

    var config = new BlitzConfig { AppKey = "YOUR_KEY", UserId = "YOUR_USER" };
    var client = new BlitzInteractiveApiClient(config);
    await client.LoginAsync();

    var profile = await client.GetProfileAsync();
    Console.WriteLine($"{profile.ClientId} {profile.UserName} KYC:{profile.IsKYC}");
    ```

---

## Endpoint

```
GET /profile
```

### Request

```bash
curl "https://{baseurl}/api_interactive/api/v1/profile" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Response

The response is a single profile object:

```json
{
  "Id": "85abbeeb-2e7f-44a5-b668-84720d29ceed",
  "FirstName": "Algomind",
  "LastName": "Algo",
  "PhoneNumber": "5645467890",
  "Address": "Delhi Pitampura",
  "City": "New Delhi",
  "PinCode": 110034,
  "Country": "India",
  "DateOfBirth": "2003-02-07T00:00:00",
  "IsTemporaryPassword": false,
  "UserType": "Client",
  "ClientType": 1,
  "BranchId": "322887ac-728e-45dd-83fb-f899237758b9",
  "PanNumber": "PLOSA0987F",
  "IsKYC": true,
  "ClientId": "Algo123",
  "IsActivated": true,
  "IsSuspended": false,
  "IsDeleted": false,
  "UserName": "Algo123",
  "NormalizedUserName": "ALGO123",
  "Email": "nehaverma2097@gmail.com",
  "NormalizedEmail": "NEHAVERMA2097@GMAIL.COM",
  "EmailConfirmed": false,
  "PhoneNumberConfirmed": false,
  "TwoFactorEnabled": false,
  "LockoutEnd": null,
  "LockoutEnabled": true,
  "AccessFailedCount": 0,
  "CreatedAt": "0001-01-01T00:00:00",
  "UpdatedAt": "2026-04-06T11:24:15",
  "UpdatedBy": "superadmin",
  "RiskLevel": 0,
  "Is2FAEmail": true,
  "Is2FAPhone": false,
  "LastLogin": "2026-09-17T13:22:42",
  "SubscriptionExpiry": "2027-04-06T16:49:26",
  "IsPro": false,
  "CountryCode": "+91",
  "AutoSquareOfEnabled": false,
  "TPOmsRoute": false,
  "IsTOTPEnabled": false
}
```

### Profile fields

| Field | Type | Description |
|-------|------|-------------|
| `Id` | string | User's unique entity identifier |
| `FirstName` | string | First name |
| `LastName` | string | Last name |
| `PhoneNumber` | string | Phone number |
| `Address` | string | Residential / registered address |
| `City` | string | City |
| `PinCode` | int | PIN code |
| `Country` | string | Country |
| `DateOfBirth` | DateTime | Date of birth |
| `IsTemporaryPassword` | bool | Whether on a temporary password |
| `UserType` | string | User type |
| `ClientType` | int | Numeric client type |
| `BranchId` | string | Branch identifier |
| `PanNumber` | string | PAN number |
| `IsKYC` | bool | Whether KYC is complete |
| `ClientId` | string | Trading client identifier (e.g. `Algo123`) |
| `IsActivated` | bool | Whether account is activated |
| `IsSuspended` | bool | Whether account is suspended |
| `IsDeleted` | bool | Whether account is marked deleted |
| `UserName` | string | Login username |
| `NormalizedUserName` | string | Uppercase-normalized username |
| `Email` | string | Email address |
| `NormalizedEmail` | string | Uppercase-normalized email |
| `EmailConfirmed` | bool | Whether email is confirmed |
| `PhoneNumberConfirmed` | bool | Whether phone is confirmed |
| `TwoFactorEnabled` | bool | Whether 2FA is enabled |
| `LockoutEnd` | DateTime? | Lockout end (`null` if not locked) |
| `LockoutEnabled` | bool | Whether lockout is enabled |
| `AccessFailedCount` | int | Failed sign-in attempts |
| `CreatedAt` | DateTime | Account creation timestamp |
| `UpdatedAt` | DateTime | Last update timestamp |
| `UpdatedBy` | string | Who last updated the profile |
| `RiskLevel` | int | Risk level |
| `Is2FAEmail` | bool | Whether email 2FA is enabled |
| `Is2FAPhone` | bool | Whether phone 2FA is enabled |
| `LastLogin` | DateTime | Last login timestamp |
| `SubscriptionExpiry` | DateTime | Subscription expiry |
| `IsPro` | bool | Whether on a Pro subscription |
| `CountryCode` | string | Country dialing code |
| `AutoSquareOfEnabled` | bool | Whether auto-square-off is enabled |
| `TPOmsRoute` | bool | Whether TP OMS routing is enabled |
| `IsTOTPEnabled` | bool | Whether TOTP-based 2FA is enabled |