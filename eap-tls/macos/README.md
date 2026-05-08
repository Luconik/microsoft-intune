# EAP-TLS — macOS with Microsoft Intune + Aruba Central NAC

> 🇫🇷 [Français](README-fr.md) | 🇬🇧 English

![Platform](https://img.shields.io/badge/platform-macOS%2014%2B-lightgrey?logo=apple)
![Intune](https://img.shields.io/badge/Microsoft%20Intune-required-blue?logo=microsoft)
![Auth](https://img.shields.io/badge/auth-EAP--TLS%20%2F%20802.1X-green)
![Last updated](https://img.shields.io/badge/updated-May%202026-orange)

---

## Table of contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Part 1 — Intune Configuration Profiles](#part-1--intune-configuration-profiles)
  - [1.1 Trusted Certificate](#11-trusted-certificate)
  - [1.2 SCEP Certificate](#12-scep-certificate)
  - [1.3 Wi-Fi](#13-wi-fi)
- [Part 2 — macOS Enrollment via Company Portal](#part-2--macos-enrollment-via-company-portal)
  - [2.1 Install Company Portal](#21-install-company-portal)
  - [2.2 Sign in and enroll](#22-sign-in-and-enroll)
  - [2.3 Install the management profile](#23-install-the-management-profile)
  - [2.4 Complete enrollment](#24-complete-enrollment)
- [Part 3 — Validation](#part-3--validation)
- [References](#references)

---

## Overview

This guide covers the **Microsoft Intune** side of the EAP-TLS configuration for macOS — Intune profiles (Trusted Certificate, SCEP, Wi-Fi) and Company Portal enrollment.

```
macOS endpoint (Intune-managed)
    │
    │  SCEP certificate issued by Central NAC CA
    ▼
Aruba AP (802.1X EAP-TLS)
    │
    │  RADIUS authentication
    ▼
Aruba Central NAC
    │
    │  Compliance check via OAuth2
    ▼
Microsoft Intune / Entra ID
    │
    ▼
Network access granted (role assigned by NAC policy)
```

> [!NOTE]
> This guide covers Intune configuration only. For the one-time prerequisites (Entra ID App Registration + Apple APNs certificate), see [../prerequisites/README.md](../prerequisites/README.md). For the Aruba Central NAC configuration, see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

---

## Prerequisites

- One-time prerequisites completed — see [../prerequisites/README.md](../prerequisites/README.md)
  - Entra ID App Registration configured
  - **Apple MDM Push Certificate (APNs)** configured in Intune
- Aruba Central NAC configured — see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune)
- SCEP URL and root CA certificate retrieved from Central NAC
- macOS 14 Sonoma or later
- Microsoft **Company Portal** installed on the device

---

## Part 1 — Intune Configuration Profiles

Three profiles must be created in Intune **in this order**:

| # | Profile type | Name | Purpose |
|---|---|---|---|
| 1 | Trusted Certificate | `Luconik Trusted - macOS` | Deploy the Central NAC root CA to the device |
| 2 | SCEP Certificate | `Luconik SCEP - macOS` | Request a client certificate from Central NAC |
| 3 | Wi-Fi | `Luconik Wi-Fi - macOS` | Configure 802.1X EAP-TLS on the SSID |

> [!IMPORTANT]
> Always create profiles in this order. The SCEP profile depends on the Trusted Certificate profile, and the Wi-Fi profile depends on both.

---

### 1.1 Trusted Certificate

Navigate to:
```
Intune Admin Center → Devices → macOS → Configuration → + Create → Templates → Trusted certificate
```

<p align="center"><img src="screenshots/81-intune-macos-trusted-profile-select.png" alt="Intune - Select Trusted certificate template" width="900"/></p>

**Basics** — set the name to `Luconik Trusted - macOS`.

<p align="center"><img src="screenshots/82-intune-macos-trusted-basics.png" alt="Intune - Trusted certificate basics" width="900"/></p>

**Configuration settings** — upload the root CA certificate (`.cer`) downloaded from Central NAC. Set **Deployment Channel** to `Device Channel`.

<p align="center"><img src="screenshots/83-intune-macos-trusted-config-cert-uploaded.png" alt="Intune - Trusted certificate upload" width="900"/></p>

**Assignments** — assign to **All devices** and **All users**.

<p align="center"><img src="screenshots/84-intune-macos-trusted-assignments.png" alt="Intune - Trusted certificate assignments" width="900"/></p>

**Review + create** — verify the summary then click **Create**.

<p align="center"><img src="screenshots/85-intune-macos-trusted-review-create.png" alt="Intune - Trusted certificate review" width="900"/></p>

<p align="center"><img src="screenshots/86-intune-macos-trusted-assigned-all.png" alt="Intune - Trusted certificate assigned" width="900"/></p>

---

### 1.2 SCEP Certificate

Navigate to:
```
Intune Admin Center → Devices → macOS → Configuration → + Create → Templates → SCEP certificate
```

<p align="center"><img src="screenshots/87-intune-macos-scep-profile-select.png" alt="Intune - Select SCEP certificate template" width="900"/></p>

**Basics** — set the name to `Luconik SCEP - macOS`.

<p align="center"><img src="screenshots/88-intune-macos-scep-basics.png" alt="Intune - SCEP basics" width="900"/></p>

**Configuration settings**

| Setting | Value |
|---|---|
| Deployment Channel | `Device Channel` |
| Certificate type | `User` |
| Subject name format | `CN={{UserPrincipalName}}` |
| Subject alternative name | `User principal name (UPN)` → `{{UserPrincipalName}}` |
| Certificate validity period | `1 Years` |
| Key usage | `Digital signature` |
| Key size (bits) | `2048` |
| Root Certificate | `Luconik Trusted - macOS` |
| Extended key usage | `Client Authentication` — `1.3.6.1.5.5.7.3.2` |
| Renewal threshold (%) | `20` |
| SCEP Server URLs | Central NAC SCEP URL |

<p align="center"><img src="screenshots/89-intune-macos-scep-config-top.png" alt="Intune - SCEP config top" width="900"/></p>

<p align="center"><img src="screenshots/90-intune-macos-scep-config-bottom.png" alt="Intune - SCEP config bottom" width="900"/></p>

**Review + create**

<p align="center"><img src="screenshots/91-intune-macos-scep-review-create-top.png" alt="Intune - SCEP review top" width="900"/></p>

<p align="center"><img src="screenshots/92-intune-macos-scep-review-create-bottom.png" alt="Intune - SCEP review bottom" width="900"/></p>

**Assignments** — assign to **All devices** and **All users**.

<p align="center"><img src="screenshots/93-intune-macos-scep-assigned-all.png" alt="Intune - SCEP assigned" width="900"/></p>

---

### 1.3 Wi-Fi

Navigate to:
```
Intune Admin Center → Devices → macOS → Configuration → + Create → Templates → Wi-Fi
```

<p align="center"><img src="screenshots/94-intune-macos-wifi-profile-select.png" alt="Intune - Select Wi-Fi template" width="900"/></p>

**Basics** — set the name to `Luconik Wi-Fi - macOS`.

<p align="center"><img src="screenshots/95-intune-macos-wifi-basics.png" alt="Intune - Wi-Fi basics" width="900"/></p>

**Configuration settings**

| Setting | Value |
|---|---|
| SSID | `luconik-corp` |
| Connect automatically | `Enable` |
| Hidden network | `Disable` |
| EAP type | `EAP - TLS` |
| Certificate server names | `luconik-corp` |
| Root certificates for server validation | `Luconik Trusted - macOS` |
| Authentication method | `Certificates` |
| Client certificate | `Luconik SCEP - macOS` |

<p align="center"><img src="screenshots/96-intune-macos-wifi-config.png" alt="Intune - Wi-Fi config" width="900"/></p>

**Review + create**

<p align="center"><img src="screenshots/97-intune-macos-wifi-review-create.png" alt="Intune - Wi-Fi review" width="900"/></p>

**Assignments** — assign to **All devices** and **All users**.

<p align="center"><img src="screenshots/98-intune-macos-wifi-assigned-all.png" alt="Intune - Wi-Fi assigned" width="900"/></p>

---

## Part 2 — macOS Enrollment via Company Portal

### 2.1 Install Company Portal

Download the Company Portal `.pkg` from Microsoft:

```
https://go.microsoft.com/fwlink/?linkid=853070
```

<p align="center"><img src="screenshots/99-macos-cp-installer-download.png" alt="Company Portal - Download installer" width="900"/></p>

Run the installer and follow the wizard — Introduction → License → Installation type → Authenticate with Touch ID or password → Success → move the installer to Trash.

<p align="center"><img src="screenshots/100-macos-cp-install-intro.png" alt="Company Portal - Introduction" width="900"/></p>
<p align="center"><img src="screenshots/101-macos-cp-install-license.png" alt="Company Portal - License" width="900"/></p>
<p align="center"><img src="screenshots/102-macos-cp-install-license-agree.png" alt="Company Portal - Accept license" width="900"/></p>
<p align="center"><img src="screenshots/103-macos-cp-install-type.png" alt="Company Portal - Installation type" width="900"/></p>
<p align="center"><img src="screenshots/104-macos-cp-install-auth.png" alt="Company Portal - Authentication" width="900"/></p>
<p align="center"><img src="screenshots/105-macos-cp-install-success.png" alt="Company Portal - Success" width="900"/></p>
<p align="center"><img src="screenshots/106-macos-cp-install-trash-installer.png" alt="Company Portal - Move to Trash" width="900"/></p>

---

### 2.2 Sign in and enroll

Open **Company Portal** and sign in with the corporate Entra ID account.

<p align="center"><img src="screenshots/107-macos-cp-signin.png" alt="Company Portal - Sign in" width="900"/></p>

Click **Begin** to start device enrollment.

<p align="center"><img src="screenshots/108-macos-cp-setup-begin.png" alt="Company Portal - Set up MSFT access" width="900"/></p>

Review the privacy information — what the organization can and cannot see on the device.

<p align="center"><img src="screenshots/109-macos-cp-privacy-review.png" alt="Company Portal - Privacy review" width="900"/></p>

---

### 2.3 Install the management profile

Click **Download profile** in Company Portal.

<p align="center"><img src="screenshots/110-macos-cp-install-mgmt-profile.png" alt="Company Portal - Install management profile" width="900"/></p>

macOS opens **System Settings → General → Device Management** automatically.

<p align="center"><img src="screenshots/111-macos-syssettings-profile-downloaded.png" alt="System Settings - Profile downloaded" width="900"/></p>

The profile appears as **Not installed** — click it to review, then click **Install**.

<p align="center"><img src="screenshots/112-macos-syssettings-profile-pending.png" alt="System Settings - Profile pending" width="900"/></p>

<p align="center"><img src="screenshots/113-macos-syssettings-profile-review.png" alt="System Settings - Profile review" width="900"/></p>

Enter the macOS user password to authorize MDM enrollment.

<p align="center"><img src="screenshots/114-macos-syssettings-mdm-enroll-password.png" alt="System Settings - MDM password" width="900"/></p>

The profile is installed — the Mac is now **managed by MSFT**.

<p align="center"><img src="screenshots/115-macos-syssettings-profile-enrolled.png" alt="System Settings - Profile enrolled" width="900"/></p>

---

### 2.4 Complete enrollment

Return to Company Portal — the profile download completes automatically.

<p align="center"><img src="screenshots/116-macos-cp-mgmt-profile-downloading.png" alt="Company Portal - Profile downloading" width="900"/></p>

<p align="center"><img src="screenshots/117-macos-cp-enrollment-complete.png" alt="Company Portal - You're all set" width="900"/></p>

Select the device category when prompted: `RootCA-Installed`.

<p align="center"><img src="screenshots/118-macos-cp-device-category.png" alt="Company Portal - Device category" width="900"/></p>

<p align="center"><img src="screenshots/119-macos-cp-device-category-selected.png" alt="Company Portal - Category selected" width="900"/></p>

---

## Part 3 — Validation

### 3.1 Certificates in Keychain Access

Open **Keychain Access** → **System** keychain — the Intune MDM Device certificate should be present.

<p align="center"><img src="screenshots/120-macos-keychain-system-intune-cert.png" alt="Keychain - System - Intune MDM cert" width="900"/></p>

In the **login** keychain, verify:

| Certificate | Expected |
|---|---|
| `Cloud Authentication Private Root CA (powered by HPE Aruba)` | Trusted |
| `Cloud Authentication SCEP RA (powered by HPE Aruba)` | Present |
| `<user@domain>` | Client certificate ×2 (SCEP-issued) |

<p align="center"><img src="screenshots/121-macos-keychain-login-aruba-ca-scep.png" alt="Keychain - Login - Aruba CA and SCEP certs" width="900"/></p>

---

### 3.2 Profiles in System Settings

Navigate to:
```
System Settings → General → Device Management
```

Three profiles should appear under **User (Managed)**:

| Profile | Type |
|---|---|
| Credential Profile | Trusted Certificate |
| SCEP Profile | SCEP Certificate |
| WiFi Profile | Wi-Fi |

<p align="center"><img src="screenshots/122-macos-syssettings-profiles-deployed.png" alt="System Settings - All profiles deployed" width="900"/></p>

---

### 3.3 Wi-Fi connection

The `luconik-corp` SSID should be connected with **WPA2 Enterprise** security.

<p align="center"><img src="screenshots/123-macos-wifi-connected-wpa2-enterprise.png" alt="Wi-Fi - luconik-corp connected" width="900"/></p>

---

### 3.4 Aruba Central NAC

> [!NOTE]
> The Central NAC validation screenshots are in the [hpe-aruba-guides / central-nac-intune / macos](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune/macos/screenshots) repository.

The enrolled user should appear as **Accepted** in Central NAC → Monitoring → Clients, with:

| Field | Expected value |
|---|---|
| Status | Accepted |
| Authentication Type | EAP-TLS (Certificate) |
| Certificate Status | Valid |
| Identity Store | Luconik_EntraID |
| Vendor / Model/OS | Apple / Mac OS |

---

### 3.5 Intune Admin Center

Navigate to:
```
Intune Admin Center → Devices → macOS devices
```

The device should be **Compliant** and all three configuration profiles should show **Succeeded**.

<p align="center"><img src="screenshots/126-intune-admin-macos-device-compliant.png" alt="Intune Admin - macOS devices list" width="900"/></p>

<p align="center"><img src="screenshots/127-intune-admin-device-overview.png" alt="Intune Admin - Device overview" width="900"/></p>

<p align="center"><img src="screenshots/128-intune-admin-device-config-succeeded.png" alt="Intune Admin - Configuration profiles succeeded" width="900"/></p>

---

## References

- 📘 [Aruba Central NAC — UEM Onboarding with Intune](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)
- [Microsoft Intune — SCEP Certificate Profiles](https://learn.microsoft.com/en-us/mem/intune/protect/certificates-scep-configure)
- [Microsoft Intune — macOS enrollment](https://learn.microsoft.com/en-us/mem/intune/enrollment/macos-enroll)
- [Apple Push Certificates Portal](https://identity.apple.com/pushcert/)

---

## File structure

```
eap-tls/macos/
├── README.md               ← This file (EN)
├── README-fr.md            ← French version
└── screenshots/
    ├── 81-intune-macos-trusted-profile-select.png
    ├── ...
    ├── 130-intune-macos-profiles-list.png
    └── fr/
        ├── 107-macos-cp-signin-fr.png
        └── ...
```

---

*Last updated: May 2026 — [@Luconik](https://github.com/Luconik)*
