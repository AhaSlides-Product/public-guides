# AhaSlides – Okta SAML 2.0 SSO Configuration Guide

> **Integration type:** SAML 2.0 Single Sign-On (SSO)  
> **Supported Okta flows:** SP-initiated SSO via HTTP-POST binding

---

## Overview

This guide walks your Okta administrator through connecting **AhaSlides** to your Okta organization using SAML 2.0. Once configured, users in your organization can sign in to AhaSlides through Okta without creating a separate AhaSlides password.

---

## Prerequisites

Before you begin, make sure you have:

| Requirement | Details |
|---|---|
| Okta admin access | Super Admin or App Admin role in your Okta org |
| AhaSlides Enterprise plan | SAML SSO is an Enterprise-tier feature |
| A dedicated SSO domain | The email domain(s) your organization uses (e.g. `yourcompany.com`) |

> **Note:** Contact your AhaSlides account manager or write to [hi@ahaslides.com](mailto:hi@ahaslides.com) to have the SAML feature enabled and to receive your **Provider Name** (a unique identifier for your organization in AhaSlides, e.g. `yourcompany`).

---

## Step 1 – Create the AhaSlides App in Okta

1. Sign in to the **Okta Admin Console** (`https://<your-org>.okta.com/admin`).
2. In the left sidebar, go to **Applications → Applications**.
3. Click **Create App Integration**.
4. Select **SAML 2.0** and click **Next**.
5. Enter an **App name** (e.g. `AhaSlides`) and optionally upload the AhaSlides logo. Click **Next**.

---

## Step 2 – Configure SAML Settings in Okta

On the **Configure SAML** tab, fill in the following values:

### General

| Field | Value |
|---|---|
| **Single sign-on URL (ACS URL)** | `https://presenter.ahaslides.com/p/saml/<your-provider-name>/login/callback` |
| **Audience URI (SP Entity ID / Issuer)** | `<your-provider-name>` *(the unique Provider Name given by AhaSlides)* |
| **Name ID format** | `EmailAddress` |
| **Application username** | `Email` |
| **Response** | `Signed` |
| **Assertion Signature** | `Signed` |
| **Signature Algorithm** | `RSA-SHA256` |
| **Digest Algorithm** | `SHA256` |
| **Authentication context class** | Leave as default (or select `PasswordProtectedTransport`) |
| **Request compression** | `Uncompressed` |

> **Replace** `<your-provider-name>` with the Provider Name you received from AhaSlides.
>
> **Example ACS URL:** `https://presenter.ahaslides.com/p/saml/yourcompany/login/callback`

### Attribute Statements (optional but recommended)

AhaSlides can receive user profile data via SAML attribute statements. Configure the following attributes in Okta:

| Name | Name format | Value |
|---|---|---|
| `email` | `Unspecified` | `user.email` |
| `firstName` | `Unspecified` | `user.firstName` |
| `lastName` | `Unspecified` | `user.lastName` |

> If these attributes are not configured, AhaSlides falls back to using the **NameID** value as the user's email address.

### Group Attribute Statements

Group-based provisioning is not required for basic SSO. You may skip this section.

---

## Step 3 – Retrieve the Okta IdP Metadata

1. After saving the SAML app in Okta, go to the app's **Sign On** tab.
2. Scroll to the **SAML Signing Certificates** section.
3. Click **View IdP metadata** (or **View Setup Instructions**).
4. Copy the following values — you will need them in Step 4:

| Value to collect | Where to find it |
|---|---|
| **IdP SSO URL (Entry Point)** | Identity Provider Single Sign‑On URL |
| **IdP Issuer** | Identity Provider Issuer |
| **X.509 Certificate** | The PEM-encoded certificate (without `-----BEGIN CERTIFICATE-----` headers, unless required) |

Alternatively, download the **metadata XML file** and send it to your AhaSlides account manager — they will handle the import and complete the configuration on your behalf.

---

## Step 4 – Retrieve the AhaSlides SP Metadata (optional)

AhaSlides exposes a Service Provider metadata endpoint. You can use this to auto-configure Okta, or to share the SP metadata for troubleshooting:

```
GET https://presenter.ahaslides.com/p/saml/<your-provider-name>/metadata
```

This endpoint returns an XML document compatible with standard SAML metadata importers.

---

## Step 5 – Assign Users in Okta

1. In the Okta Admin Console, go to your **AhaSlides** application.
2. Click the **Assignments** tab.
3. Assign the app to the **groups** or **individual users** who should have SSO access to AhaSlides.
4. Ensure all assigned users have an email address that belongs to one of the SSO domains registered in Step 4.

> Users with email domains **not** in the allowed SSO domains list will receive an "Email and domain are not match" error and cannot complete SSO login.

---

## Step 6 – Test the SSO Login

### SP-initiated login

1. Navigate to the AhaSlides login page: `https://ahaslides.com/login`
2. Enter your corporate email address and click **Sign in with SSO** (or similar).
3. You will be redirected to the Okta login page.
4. Sign in with your Okta credentials.
5. Upon success, Okta posts a SAML assertion to the ACS URL and you are redirected back to AhaSlides as a logged-in user.

### IdP-initiated login

1. In the Okta End User Dashboard, locate the **AhaSlides** tile.
2. Click the tile; Okta posts a SAML assertion directly to the ACS URL.
3. You should land on the AhaSlides home page as a logged-in user.

### Direct login URL

```
https://presenter.ahaslides.com/p/saml/<your-provider-name>/login
```

You may share this URL with your users as a direct SSO entry point.

---

## User Provisioning Behavior

| Scenario | Behavior |
|---|---|
| User exists in AhaSlides with matching email | User is signed in; account is associated with your organization if not already |
| User does not exist yet | A new AhaSlides account is automatically created using the email and name from the SAML assertion |
| User account is inactive | Login is rejected with an error |
| Email domain not in allowed SSO domains | Login is rejected with an error |

---

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| Redirect to `/error` after Okta login | Invalid SAML assertion or mismatched ACS URL | Verify the **Single sign-on URL** in Okta exactly matches the ACS URL in Step 2 |
| "Email and domain are not match" | User's email domain is not in the allowed SSO domains | Contact your AhaSlides account manager to add the domain |
| "Account not found" | Provider name in the URL does not match any account | Confirm the Provider Name value with your AhaSlides account manager |
| Certificate validation failure | Expired or incorrect certificate | Download the latest signing certificate from Okta and share it with your AhaSlides account manager |
| "Email is required for SSO" | Okta is not sending the email attribute | Add the `email` attribute statement in Okta (see Step 2) or set Name ID format to `EmailAddress` |

---

## Support

For assistance with this configuration, contact:

- **AhaSlides Support:** [https://help.ahaslides.com](https://help.ahaslides.com)
- **Email:** [hi@ahaslides.com](mailto:hi@ahaslides.com)
