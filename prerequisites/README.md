# EAP-TLS Prerequisites — Microsoft Entra ID + Apple APNs

> 🇫🇷 [Français](README-fr.md) | 🇬🇧 English

![Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-required-blue?logo=microsoft)
![Intune](https://img.shields.io/badge/Microsoft%20Intune-required-blue?logo=microsoft)
![APNs](https://img.shields.io/badge/Apple%20APNs-required%20for%20macOS%20%2F%20iOS-lightgrey?logo=apple)
![Last updated](https://img.shields.io/badge/updated-May%202026-orange)

---

## Table of contents

- [Overview](#overview)
- [Part 0 — Microsoft Entra ID](#part-0--microsoft-entra-id)
  - [0.1 Add and verify a custom domain](#01-add-and-verify-a-custom-domain)
  - [0.2 Verify Intune CNAME DNS records](#02-verify-intune-cname-dns-records)
  - [0.3 Create an App Registration](#03-create-an-app-registration)
  - [0.4 Create a Client Secret](#04-create-a-client-secret)
  - [0.5 Add API permissions](#05-add-api-permissions)
- [Part 1 — Apple MDM Push Certificate (APNs)](#part-1--apple-mdm-push-certificate-apns)
  - [1.1 Open Apple enrollment in Intune](#11-open-apple-enrollment-in-intune)
  - [1.2 Download the CSR](#12-download-the-csr)
  - [1.3 Sign in to Apple Push Certificates Portal](#13-sign-in-to-apple-push-certificates-portal)
  - [1.4 Create a new Push Certificate](#14-create-a-new-push-certificate)
  - [1.5 Upload the CSR and download the certificate](#15-upload-the-csr-and-download-the-certificate)
  - [1.6 Upload the certificate to Intune](#16-upload-the-certificate-to-intune)
- [Next steps](#next-steps)

---

## Overview

This guide covers the two one-time prerequisites required before deploying EAP-TLS profiles with Microsoft Intune:

| Prerequisite | Required for |
|---|---|
| Microsoft Entra ID App Registration | All platforms — needed for Aruba Central NAC OAuth2 identity store |
| Apple MDM Push Certificate (APNs) | macOS and iOS/iPadOS only |

> [!NOTE]
> These steps are performed once per tenant. If your Entra ID App Registration and APNs certificate are already configured, skip directly to the platform guides.

---

## Part 0 — Microsoft Entra ID

### 0.1 Add and verify a custom domain

Navigate to:
```
Entra ID → Custom domains → + Add custom domain
```

<p align="center"><img src="screenshots/01-entra-id-custom-domain-add.png" alt="Entra - Add custom domain" width="900"/></p>

Add the TXT verification record to your DNS registrar.

<p align="center"><img src="screenshots/02-entra-id-custom-domain-txt-record.png" alt="Entra - TXT verification record" width="900"/></p>

<p align="center"><img src="screenshots/03-entra-id-custom-domain-portal.png" alt="Entra - Custom domain in portal" width="900"/></p>

<p align="center"><img src="screenshots/04-dns-registrar-records.png" alt="DNS - Registrar records" width="900"/></p>

<p align="center"><img src="screenshots/05-dns-registrar-txt-record-add.png" alt="DNS - Add TXT record" width="900"/></p>

Return to Entra ID and click **Verify**.

<p align="center"><img src="screenshots/06-entra-id-custom-domain-verify.png" alt="Entra - Verify domain" width="900"/></p>

---

### 0.2 Verify Intune CNAME DNS records

Required for automatic device enrollment in Intune.

<p align="center"><img src="screenshots/07-intune-dns-cname-records.png" alt="Intune - DNS CNAME records" width="900"/></p>

---

### 0.3 Create an App Registration

Navigate to:
```
Entra ID → App registrations → + New registration
```

<p align="center"><img src="screenshots/08-entra-id-app-registration-new.png" alt="Entra - New App Registration" width="900"/></p>

Configure the redirect URI.

<p align="center"><img src="screenshots/09-entra-id-app-registration-redirect-uri.png" alt="Entra - Redirect URI" width="900"/></p>

> [!NOTE]
> Keep the **Application (client) ID** and **Directory (tenant) ID** — they are required when configuring the Aruba Central Intune extension and the NAC OAuth identity store.

---

### 0.4 Create a Client Secret

Navigate to:
```
Application → Certificates & secrets → + New client secret
```

<p align="center"><img src="screenshots/10-entra-id-client-secret-new.png" alt="Entra - New client secret" width="900"/></p>

> [!WARNING]
> Copy the **value** immediately — it won't be shown again after you leave this page.

<p align="center"><img src="screenshots/11-entra-id-client-secret-value.png" alt="Entra - Client secret value" width="900"/></p>

<p align="center"><img src="screenshots/12-entra-id-client-secret-overview.png" alt="Entra - Client secret overview" width="900"/></p>

---

### 0.5 Add API permissions

Navigate to:
```
Application → API permissions → + Add permission → Microsoft Graph → Intune
```

<p align="center"><img src="screenshots/13-entra-id-api-permissions-add.png" alt="Entra - Add API permissions" width="900"/></p>

<p align="center"><img src="screenshots/14-entra-id-api-permissions-graph-intune.png" alt="Entra - Graph + Intune permissions" width="900"/></p>

---

## Part 1 — Apple MDM Push Certificate (APNs)

> [!IMPORTANT]
> Required for macOS and iOS/iPadOS only. Skip if you are deploying to Windows endpoints only.
> This is a one-time setup per tenant — the same certificate covers both macOS and iOS/iPadOS.

### 1.1 Open Apple enrollment in Intune

Navigate to:
```
Intune Admin Center → Devices → Enrollment → Apple tab
```

<p align="center"><img src="screenshots/71-intune-apns-enrollment-apple-tab.png" alt="Intune - Apple enrollment tab" width="900"/></p>

Click **Apple MDM Push Certificate**.

---

### 1.2 Download the CSR

Check **I agree** to grant Microsoft permission to send data to Apple, then click **Download your CSR**.

<p align="center"><img src="screenshots/72-intune-apns-configure-mdm-push-agree.png" alt="Intune - Configure MDM Push Certificate" width="900"/></p>

---

### 1.3 Sign in to Apple Push Certificates Portal

Go to [https://identity.apple.com](https://identity.apple.com) and sign in with a **corporate Apple ID**.

<p align="center"><img src="screenshots/73-intune-apns-apple-id-login.png" alt="Apple ID - Sign in" width="900"/></p>

> [!WARNING]
> Use a shared corporate Apple ID, not a personal one. The certificate renewal will require the same Apple ID each year.

---

### 1.4 Create a new Push Certificate

Click **Create a Certificate**, then accept the Terms of Use.

<p align="center"><img src="screenshots/74-intune-apns-push-portal-existing-certs.png" alt="Apple Push Portal - Existing certificates" width="900"/></p>

<p align="center"><img src="screenshots/75-intune-apns-push-portal-terms.png" alt="Apple Push Portal - Terms of Use" width="900"/></p>

<p align="center"><img src="screenshots/76-intune-apns-push-portal-terms-accepted.png" alt="Apple Push Portal - Terms accepted" width="900"/></p>

---

### 1.5 Upload the CSR and download the certificate

Upload the `IntuneCSR.csr` file downloaded in step 1.2, then click **Upload**.

<p align="center"><img src="screenshots/77-intune-apns-push-portal-upload-csr.png" alt="Apple Push Portal - Upload CSR" width="900"/></p>

Download the generated `.pem` certificate.

<p align="center"><img src="screenshots/78-intune-apns-push-portal-confirmation.png" alt="Apple Push Portal - Confirmation" width="900"/></p>

---

### 1.6 Upload the certificate to Intune

Back in Intune, enter the Apple ID used, upload the `.pem` file, then click **Upload**.

<p align="center"><img src="screenshots/79-intune-apns-configure-mdm-push-upload-pem.png" alt="Intune - Upload MDM Push Certificate" width="900"/></p>

The certificate is now configured and active.

<p align="center"><img src="screenshots/80-intune-apns-configure-mdm-push-configured.png" alt="Intune - MDM Push Certificate configured" width="900"/></p>

---

## Next steps

Once prerequisites are configured, proceed to the platform guides:

| Platform | Guide |
|---|---|
| Windows | [../windows/README.md](../windows/README.md) |
| macOS | [../macos/README.md](../macos/README.md) |
| iOS/iPadOS | [../ios/README.md](../ios/README.md) |

For the Aruba Central NAC configuration (identity store, roles, authorization policies, SSID), see:

→ [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune)

---

## File structure

```
eap-tls/prerequisites/
├── README.md               ← This file (EN)
├── README-fr.md            ← French version
└── screenshots/
    ├── 01-entra-id-custom-domain-add.png
    ├── ...
    ├── 14-entra-id-api-permissions-graph-intune.png
    ├── 71-intune-apns-enrollment-apple-tab.png
    ├── ...
    └── 80-intune-apns-configure-mdm-push-configured.png
```

---

*Last updated: May 2026 — [@Luconik](https://github.com/Luconik)*
