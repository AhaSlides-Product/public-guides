# AhaSlides – Okta SCIM 2.0 Provisioning Configuration Guide

> **Integration type:** SCIM 2.0 Automated User Provisioning  
> **Protocol:** SCIM 2.0 (RFC 7643 / RFC 7644)  
> **Authentication:** Bearer token (API key)

---

## Overview

This guide walks your Okta administrator through enabling **automated user provisioning** for AhaSlides using the SCIM 2.0 protocol. Once configured, Okta will automatically:

- **Create** AhaSlides accounts for newly assigned users
- **Update** user profile attributes (name, email, active status)
- **Deactivate** users when they are unassigned from the AhaSlides app in Okta

---

## Prerequisites

| Requirement | Details |
|---|---|
| Okta admin access | Super Admin or App Admin role in your Okta org |
| AhaSlides Enterprise plan | SCIM provisioning is an Enterprise-tier feature |
| AhaSlides SCIM API key | Provided by your AhaSlides account manager (see Step 1) |
| AhaSlides app in Okta | The AhaSlides SAML app should already be added to your Okta org. SCIM can be configured separately if you are only using provisioning without SSO. |

> **Note:** Contact your AhaSlides account manager or write to [hi@ahaslides.com](mailto:hi@ahaslides.com) to have SCIM provisioning enabled for your organization and to receive your SCIM API key.

---

## Step 1 – Obtain a SCIM API Key

A SCIM API key authenticates Okta's provisioning requests to AhaSlides. This key is generated once and must be kept confidential.

> Your AhaSlides account manager will generate this key and share it with you securely.

**Key format example:**
```
aha_scim_a1b2c3d4e5f6...  (128+ character token)
```

> **Important:** Copy and store the key immediately upon receipt — it cannot be retrieved again after initial generation.

---

## Step 2 – Configure SCIM Provisioning in Okta

1. Sign in to the **Okta Admin Console** (`https://<your-org>.okta.com/admin`).
2. Go to **Applications → Applications** and open the **AhaSlides** app.
3. Click the **Provisioning** tab, then click **Configure API Integration**.
4. Check **Enable API integration**.
5. Enter the following values:

| Field | Value |
|---|---|
| **SCIM connector base URL** | `https://presenter.ahaslides.com/api/scim/v2` |
| **Unique identifier field for users** | `userName` |
| **Authentication mode** | `Bearer Token` |
| **Bearer token** | `<your-scim-api-key>` |

> **Note:** If your Okta org shows **"Bearer Token"** as an authentication mode option, select it and paste your SCIM API key directly into the **Token** field — Okta will automatically send it as an `Authorization: Bearer <token>` header. If you only see **"HTTP Header"**, select that and enter `Bearer <your-scim-api-key>` as the full header value.

6. Click **Test API Credentials** to verify the connection.
7. Click **Save**.

---

## Step 3 – Enable Provisioning Features

After saving the API integration, configure which provisioning actions Okta should perform:

1. In the **Provisioning** tab, click **To App** (under Settings).
2. Enable the following features:

| Feature | Enable? | Notes |
|---|---|---|
| **Create Users** | ✅ Yes | Creates a new AhaSlides account when a user is assigned |
| **Update User Attributes** | ✅ Yes | Syncs profile changes from Okta to AhaSlides |
| **Deactivate Users** | ✅ Yes | Sets `active = false` when a user is unassigned or deactivated in Okta |
| **Sync Password** | ❌ No | Not supported — AhaSlides uses SSO for authentication |

3. Click **Save**.

---

## Step 4 – Configure Attribute Mappings

AhaSlides supports the following SCIM attributes. Verify the attribute mappings in Okta match the table below:

| Okta attribute | SCIM attribute sent to AhaSlides | AhaSlides field |
|---|---|---|
| `user.login` | `userName` | `email` (used as the unique user identifier) |
| `user.displayName` | `displayName` | `firstName` + `lastName` (parsed from display name) |
| `user.activated` | `active` | `active` (account status) |

> **Important:** AhaSlides uses `userName` as the user's **email address**. Ensure that Okta maps `user.login` (the user's email) to the SCIM `userName` field.

### Name Parsing

When AhaSlides receives a `displayName` value, it parses it as follows:
- The **first word** becomes `firstName`
- All **remaining words** become `lastName`

Example: `"Jane Smith"` → `firstName: "Jane"`, `lastName: "Smith"`

---

## Step 5 – Assign Users to the AhaSlides App

1. In the Okta Admin Console, go to the **AhaSlides** app → **Assignments** tab.
2. Assign the app to **groups** or **individual users** who should have access.
3. Once assigned, Okta will automatically call the AhaSlides SCIM API to provision those users.

---

## Supported SCIM Endpoints

AhaSlides implements the following SCIM 2.0 endpoints under the base URL `https://presenter.ahaslides.com/api/scim/v2`:

| HTTP Method | Endpoint | Description |
|---|---|---|
| `GET` | `/Users` | List all provisioned users (supports `filter`, `startIndex`, `count`) |
| `GET` | `/Users/{id}` | Retrieve a single user by their AhaSlides SCIM ID |
| `POST` | `/Users` | Create a new user |
| `PATCH` | `/Users/{id}` | Partially update a user (e.g. deactivate, change attributes) |
| `PUT` | `/Users/{id}` | Replace all user attributes |

### List Users – Query Parameters

| Parameter | Description |
|---|---|
| `filter` | Filter by email: `userName eq "user@example.com"` |
| `startIndex` | 1-based index for pagination (default: `1`) |
| `count` | Number of results per page (default: `10`) |

### Example: List Users Response

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 42,
  "startIndex": 1,
  "itemsPerPage": 10,
  "Resources": [...]
}
```

### Example: Create User Request Body

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "userName": "jane.smith@yourcompany.com",
  "displayName": "Jane Smith",
  "active": true,
  "externalId": "okta-user-id-123"
}
```

### Example: Create User Response

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "456",
  "externalId": "okta-user-id-123",
  "userName": "jane.smith@yourcompany.com",
  "displayName": "Jane Smith",
  "active": true
}
```

### Example: Deactivate User (PATCH)

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "Replace",
      "path": "active",
      "value": false
    }
  ]
}
```

---

## User Provisioning Behavior

| Scenario | Behavior |
|---|---|
| User assigned in Okta for the first time | AhaSlides checks if a user with that email exists. If yes, links them to your organization. If no, creates a new account with the `Editor` role. |
| User's display name updated in Okta | AhaSlides updates `firstName` and `lastName` accordingly |
| User deactivated or unassigned in Okta | AhaSlides sets `active = false` — the user can no longer log in |
| User reactivated in Okta | AhaSlides sets `active = true` — login is re-enabled |
| User already in AhaSlides with a different organization | The user is re-associated to your organization and their role is updated |

> **Role assignment:** All SCIM-provisioned users are assigned the `Editor` role by default. Roles can be adjusted manually by your AhaSlides organization owner after provisioning.

---

## Authentication & Security

- All SCIM API requests must include the `Authorization: Bearer <your-scim-api-key>` header.
- The API key is hashed with a one-way hash before storage; AhaSlides cannot recover the plaintext key.
- If the key is compromised, contact your AhaSlides account manager to revoke the old key, generate a new one, and update the Bearer token in Okta immediately.
- The SCIM endpoint uses HTTPS exclusively; HTTP connections are not supported.

---

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| `401 Unauthorized` on Test API Credentials | Invalid or inactive API key | Contact your AhaSlides account manager to issue a new SCIM API key, then update the Bearer token in Okta |
| Users not being provisioned | Okta feature "Create Users" not enabled | Enable **Create Users** in the Provisioning → To App settings (Step 3) |
| Users not being deactivated | Okta feature "Deactivate Users" not enabled | Enable **Deactivate Users** in the Provisioning → To App settings |
| Incorrect name in AhaSlides | `displayName` not mapped in Okta | Verify the `displayName` attribute mapping in Okta points to `user.displayName` |
| Filter queries returning no results | Email case mismatch | AhaSlides stores emails in **lowercase**; ensure Okta sends lowercase emails |
| `500 Internal Server Error` on user creation | Duplicate or invalid data | Check that `userName` is a valid email address; contact AhaSlides support with the error details |

---

## Support

For assistance with this configuration, contact:

- **AhaSlides Support:** [https://help.ahaslides.com](https://help.ahaslides.com)  
- **Email:** [hi@ahaslides.com](mailto:hi@ahaslides.com)
