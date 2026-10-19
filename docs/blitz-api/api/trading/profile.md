# **Profile API**

The **Profile API** allows you to retrieve the authenticated user's profile details.

Using this API, you can fetch the client's account, KYC, contact, and risk information in a single call.

---

# Base URL

```
https://{baseurl}/api_interactive/api/v1
```

---

# Headers

| Header        | Value                 | Required |
| ------------- | --------------------- | -------- |
| Content-Type  | application/json      | Yes      |
| Accept        | application/json      | Yes      |
| Authorization | Bearer {access_token} | Yes      |

---

# Endpoint

| **Type** | **Endpoint** | **Description**            |
| -------- | ------------ | -------------------------- |
| GET      | `/profile`   | Retrieve the user profile  |

---

# Get Profile {#get-profile}

Returns the profile of the authenticated user, including account status, KYC status, contact details, and risk configuration.

## What is the use of this API?

- **Account Verification:** Confirm `IsKYC`, `IsActivated`, and `IsSuspended` status of the account.
- **Client Details:** Retrieve `ClientId`, `UserName`, `Email`, and `PhoneNumber` for reconciliation.
- **Risk Configuration:** Check `RiskLevel`, `IsPro`, and `SubscriptionExpiry` for feature access decisions.

## cURL Example

```bash
curl -X 'GET' \
  'https://{baseurl}/api_interactive/api/v1/profile' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {access_token}'
```

## Response

### Success Response (200 OK)

<details>
<summary>Click to view Profile JSON Body</summary>

```json
{
    "Id":  "feb7f05d-df0b-4d00-93fc-3e59b7edf4bf",
    "FirstName":  "Sameer",
    "LastName":  "Devkar",
    "PhoneNumber":  "9685321047",
    "Address":  "Pune,Maharashtra",
    "City":  "Pune",
    "PinCode":  411006,
    "Country":  "India",
    "DateOfBirth":  "1970-01-01T00:00:00",
    "IsTemporaryPassword":  false,
    "UserType":  "Client",
    "ClientType":  1,
    "BranchId":  "322887ac-728e-45dd-83fb-f899237758b9",
    "PanNumber":  "ABCED4161D",
    "IsKYC":  true,
    "ClientId":  "SAM001",
    "IsActivated":  true,
    "IsSuspended":  false,
    "IsDeleted":  false,
    "UserName":  "SAM001",
    "NormalizedUserName":  "SAM001",
    "Email":  "s.devkar@quantxpress.com",
    "NormalizedEmail":  "S.DEVKAR@QUANTXPRESS.COM",
    "EmailConfirmed":  false,
    "PhoneNumberConfirmed":  false,
    "TwoFactorEnabled":  false,
    "LockoutEnd":  null,
    "LockoutEnabled":  true,
    "AccessFailedCount":  0,
    "CreatedAt":  "2026-09-21T18:58:32",
    "UpdatedAt":  "2026-09-21T18:58:32",
    "UpdatedBy":  "System",
    "RiskLevel":  0,
    "Is2FAEmail":  false,
    "Is2FAPhone":  false,
    "LastLogin":  "2026-09-22T11:56:23",
    "SubscriptionExpiry":  "2027-04-06T17:18:37",
    "IsPro":  false,
    "CountryCode":  "+91",
    "AutoSquareOfEnabled":  false,
    "TPOmsRoute":  false,
    "IsTOTPEnabled":  false
}
```
</details>

## Response Field Descriptions

| Field                    | Type     | Description                                            |
| ------------------------ | -------- | ------------------------------------------------------ |
| `Id`                     | string   | User's unique entity identifier                        |
| `FirstName`              | string   | First name of the user                                 |
| `LastName`               | string   | Last name of the user                                  |
| `PhoneNumber`            | string   | Phone number                                           |
| `Address`                | string   | Residential / registered address                       |
| `City`                   | string   | City of residence                                      |
| `PinCode`                | integer  | Postal / PIN code                                      |
| `Country`                | string   | Country of residence                                   |
| `DateOfBirth`            | string   | Date of birth (ISO 8601)                               |
| `IsTemporaryPassword`    | boolean  | Whether the user is on a temporary password            |
| `UserType`               | string   | User type (e.g., Admin, User)                          |
| `ClientType`             | integer  | Numeric client type                                    |
| `BranchId`               | string   | Branch identifier                                      |
| `PanNumber`              | string   | PAN number                                             |
| `IsKYC`                  | boolean  | Whether KYC is completed                               |
| `ClientId`               | string   | The trading client identifier (e.g., `Algo123`)         |
| `IsActivated`            | boolean  | Whether the account is activated                       |
| `IsSuspended`            | boolean  | Whether the account is suspended                       |
| `IsDeleted`              | boolean  | Whether the account is marked deleted                  |
| `UserName`               | string   | Login username                                         |
| `NormalizedUserName`     | string   | Uppercase-normalized username                          |
| `Email`                  | string   | Email address                                          |
| `NormalizedEmail`        | string   | Uppercase-normalized email                             |
| `EmailConfirmed`         | boolean  | Whether the email is confirmed                         |
| `PhoneNumberConfirmed`   | boolean  | Whether the phone is confirmed                         |
| `TwoFactorEnabled`       | boolean  | Whether 2FA is enabled                                 |
| `LockoutEnd`             | string   | Lockout end timestamp (`null` if not locked)           |
| `LockoutEnabled`         | boolean  | Whether account lockout is enabled                     |
| `AccessFailedCount`      | integer  | Number of failed sign-in attempts                      |
| `CreatedAt`              | string   | Account creation timestamp (ISO 8601)                  |
| `UpdatedAt`              | string   | Last update timestamp (ISO 8601)                       |
| `UpdatedBy`              | string   | Who last updated the profile                           |
| `RiskLevel`              | integer  | Risk level assigned to the account                     |
| `Is2FAEmail`             | boolean  | Whether email-based 2FA is enabled                     |
| `Is2FAPhone`             | boolean  | Whether phone-based 2FA is enabled                     |
| `LastLogin`              | string   | Last login timestamp (ISO 8601)                        |
| `SubscriptionExpiry`     | string   | Subscription expiry timestamp (ISO 8601)               |
| `IsPro`                  | boolean  | Whether the user is on a Pro subscription              |
| `CountryCode`            | string   | Country dialing code                                   |
| `AutoSquareOfEnabled`    | boolean  | Whether auto-square-off is enabled                     |
| `TPOmsRoute`             | boolean  | Whether TP OMS routing is enabled                      |
| `IsTOTPEnabled`          | boolean  | Whether TOTP-based 2FA is enabled                      |

### Error Response (404 Not Found)

If no profile is found for the authenticated user, the API returns `404` with:

```json
{
  "message": "Profile not found."
}
```
