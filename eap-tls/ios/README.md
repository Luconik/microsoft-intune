# EAP-TLS — iOS/iPadOS with Microsoft Intune + Aruba Central NAC

> 🇫🇷 [Français](README-fr.md) | 🇬🇧 English

![Platform](https://img.shields.io/badge/platform-iOS%2F%20iPadOS%2016%2B-lightgrey?logo=apple)
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
- [Part 2 — iOS/iPadOS Enrollment via Company Portal](#part-2--iosipados-enrollment-via-company-portal)
  - [2.1 Install Company Portal](#21-install-company-portal)
  - [2.2 Sign in and enroll](#22-sign-in-and-enroll)
  - [2.3 Install the management profile](#23-install-the-management-profile)
  - [2.4 Complete enrollment](#24-complete-enrollment)
- [Part 3 — Validation](#part-3--validation)
- [References](#references)

---

## Overview

This guide covers the **Microsoft Intune** side of the EAP-TLS configuration for iOS/iPadOS — Intune profiles (Trusted Certificate, SCEP, Wi-Fi) and Company Portal enrollment.

```
iOS/iPadOS device (Intune-managed)
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
> This guide covers Intune configuration only. For the Aruba Central NAC side (identity store, roles, authorization policies, SSID), see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

> [!NOTE]
> The Apple MDM Push Certificate (APNs) is a one-time prerequisite for all Apple platforms. If you have already completed the macOS guide, this is already configured. Otherwise, see [prerequisites/README.md — Part 1](../../prerequisites/README.md#part-1--apple-mdm-push-certificate-apns).

---

## Prerequisites

- Aruba Central NAC configured — see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune)
- Microsoft Intune tenant active
- **Apple MDM Push Certificate** configured in Intune — see [prerequisites guide Part 1](../../prerequisites/README.md#part-1--apple-mdm-push-certificate-apns)
- iOS 16 / iPadOS 16 or later
- Microsoft **Company Portal** installed from the App Store

---

## Part 1 — Intune Configuration Profiles

Three profiles must be created in Intune **in this order**:

| # | Profile type | Name | Purpose |
|---|---|---|---|
| 1 | Trusted Certificate | `Luconik Trusted - iOS` | Deploy the Central NAC root CA to the device |
| 2 | SCEP Certificate | `Luconik SCEP - iOS` | Request a client certificate from Central NAC |
| 3 | Wi-Fi | `Luconik Wi-Fi - iOS` | Configure 802.1X EAP-TLS on the SSID |

> [!IMPORTANT]
> Always create profiles in this order. The SCEP profile depends on the Trusted Certificate profile, and the Wi-Fi profile depends on both.

---

### 1.1 Trusted Certificate

Navigate to:
```
Intune Admin Center → Devices → Configuration → + Create → iOS/iPadOS → Templates → Trusted certificate
```

<p align="center"><img src="screenshots/196-intune-ios-trusted-create-profile-template.png" alt="Intune - Select Trusted certificate template" width="900"/></p>

**Basics** — set the name to `Luconik Trusted - iOS`.

<p align="center"><img src="screenshots/197-intune-ios-trusted-basics.png" alt="Intune - Trusted certificate basics" width="900"/></p>

**Configuration settings** — upload the root CA certificate (`.cer`) downloaded from Central NAC.

<p align="center"><img src="screenshots/198-intune-ios-trusted-config-cert-uploaded.png" alt="Intune - Trusted certificate upload" width="900"/></p>

**Assignments** — assign to **All devices** and **All users**.

<p align="center"><img src="screenshots/199-intune-ios-trusted-assignments.png" alt="Intune - Trusted certificate assignments" width="900"/></p>

**Review + create** — verify the summary then click **Create**.

<p align="center"><img src="screenshots/200-intune-ios-trusted-review.png" alt="Intune - Trusted certificate review" width="900"/></p>

---

### 1.2 SCEP Certificate

Navigate to:
```
Intune Admin Center → Devices → Configuration → + Create → iOS/iPadOS → Templates → SCEP certificate
```

<p align="center"><img src="screenshots/201-intune-ios-scep-create-profile-template.png" alt="Intune - Select SCEP certificate template" width="900"/></p>

**Basics** — set the name to `Luconik SCEP - iOS`.

<p align="center"><img src="screenshots/202-intune-ios-scep-basics.png" alt="Intune - SCEP basics" width="900"/></p>

**Configuration settings**

| Setting | Value |
|---|---|
| Certificate type | `User` |
| Subject name format | `CN={{UserPrincipalName}}` |
| Subject alternative name | `URI` → `cnac+intune:///?DeviceId={{DeviceId}}` |
| Certificate validity period | `1 Years` |
| Key usage | `Digital Signature`, `Key Encipherment` |
| Key size (bits) | `2048` |
| Root Certificate | `Luconik Trusted - iOS` |
| Extended key usage | `Client Authentication` — `1.3.6.1.5.5.7.3.2` |
| Renewal threshold (%) | `20` |
| SCEP Server URLs | Central NAC SCEP URL |

<p align="center"><img src="screenshots/203-intune-ios-scep-config-top.png" alt="Intune - SCEP config top" width="900"/></p>

> [!NOTE]
> The Root Certificate picker lists available Trusted Certificate profiles. Select `Luconik Trusted - iOS` created in step 1.1.

<p align="center"><img src="screenshots/204-intune-ios-scep-config-root-cert-picker.png" alt="Intune - SCEP root certificate picker" width="900"/></p>

<p align="center"><img src="screenshots/205-intune-ios-scep-config-eku-scep-url.png" alt="Intune - SCEP EKU and SCEP URL" width="900"/></p>

**Assignments** — assign to **All devices** and **All users**.

<p align="center"><img src="screenshots/206-intune-ios-scep-assignments.png" alt="Intune - SCEP assignments" width="900"/></p>

**Review + create**

<p align="center"><img src="screenshots/207-intune-ios-scep-review.png" alt="Intune - SCEP review" width="900"/></p>

> [!IMPORTANT]
> The SCEP profile must use **Certificate type: User** with `CN={{UserPrincipalName}}`. Using `Device` type will cause Central NAC authorization to fail (Deny All) as the NAC cannot map the certificate to an Entra ID user or group.

---

### 1.3 Wi-Fi

Navigate to:
```
Intune Admin Center → Devices → Configuration → + Create → iOS/iPadOS → Templates → Wi-Fi
```

<p align="center"><img src="screenshots/209-intune-ios-wifi-create-profile-template.png" alt="Intune - Select Wi-Fi template" width="900"/></p>

**Basics** — set the name to `Luconik Wi-Fi - iOS`.

<p align="center"><img src="screenshots/210-intune-ios-wifi-basics.png" alt="Intune - Wi-Fi basics" width="900"/></p>

**Configuration settings**

| Setting | Value |
|---|---|
| Wi-Fi type | `Enterprise` |
| Network name | `luconik-corp` |
| SSID | `luconik-corp` |
| Connect automatically | `Enable` |
| Hidden network | `Disable` |
| Security type | `WPA/WPA2-Enterprise` |
| EAP type | `EAP - TLS` |
| Certificate server names | `luconik-corp` |
| Root certificates for server validation | `Luconik Trusted - iOS` |
| Authentication method | `Certificates` |
| Client certificate | `Luconik SCEP - iOS` |

<p align="center"><img src="screenshots/211-intune-ios-wifi-config-root-cert-picker.png" alt="Intune - Wi-Fi root cert picker" width="900"/></p>

<p align="center"><img src="screenshots/212-intune-ios-wifi-config-top.png" alt="Intune - Wi-Fi config top" width="900"/></p>

<p align="center"><img src="screenshots/213-intune-ios-wifi-config-client-cert-picker.png" alt="Intune - Wi-Fi client cert picker" width="900"/></p>

<p align="center"><img src="screenshots/214-intune-ios-wifi-config-bottom.png" alt="Intune - Wi-Fi config bottom" width="900"/></p>

**Assignments** — assign to **All devices** and **All users**.

<p align="center"><img src="screenshots/215-intune-ios-wifi-assignments.png" alt="Intune - Wi-Fi assignments" width="900"/></p>

**Review + create**

<p align="center"><img src="screenshots/216-intune-ios-wifi-review.png" alt="Intune - Wi-Fi review" width="900"/></p>

After creating all three profiles, the configuration list should show:

<p align="center"><img src="screenshots/217-intune-config-profiles-all-10-policies.png" alt="Intune - All configuration profiles" width="900"/></p>

---

## Part 2 — iOS/iPadOS Enrollment via Company Portal

### 2.1 Install Company Portal

Search for **Intune Company Portal** in the App Store and install it.

<p align="center"><img src="screenshots/164-ipad-en-appstore-search-intune.png" alt="App Store - Search Intune Company Portal" width="700"/></p>

<p align="center"><img src="screenshots/165-ipad-en-appstore-cp-listing.png" alt="App Store - Intune Company Portal listing" width="700"/></p>

<p align="center"><img src="screenshots/166-ipad-en-appstore-cp-open.png" alt="App Store - Open Company Portal" width="700"/></p>

---

### 2.2 Sign in and enroll

Open **Company Portal** and tap **Sign in**.

<p align="center"><img src="screenshots/167-ipad-en-cp-splash.png" alt="Company Portal - Splash screen" width="700"/></p>

Sign in with the corporate Entra ID account.

<p align="center"><img src="screenshots/168-ipad-en-cp-signin-empty.png" alt="Company Portal - Sign in" width="700"/></p>

<p align="center"><img src="screenshots/169-ipad-en-cp-signin-email.png" alt="Company Portal - Email" width="700"/></p>

<p align="center"><img src="screenshots/170-ipad-en-cp-signin-password.png" alt="Company Portal - Password" width="700"/></p>

Approve the MFA request in Microsoft Authenticator.

<p align="center"><img src="screenshots/171-ipad-en-cp-mfa-authenticator.png" alt="Company Portal - MFA Authenticator" width="700"/></p>

Allow notifications when prompted.

<p align="center"><img src="screenshots/172-ipad-en-cp-notifications-access.png" alt="Company Portal - Notifications" width="700"/></p>

Tap **Begin** to start device enrollment.

<p align="center"><img src="screenshots/173-ipad-en-cp-setup-begin.png" alt="Company Portal - Set up MSFT access" width="700"/></p>

Review the privacy information — what the organization can and cannot see on the device.

<p align="center"><img src="screenshots/174-ipad-en-cp-privacy-cant.png" alt="Company Portal - Privacy" width="700"/></p>

<p align="center"><img src="screenshots/175-ipad-en-cp-setup-step1-done.png" alt="Company Portal - Step 1 done" width="700"/></p>

---

### 2.3 Install the management profile

Company Portal opens a browser to download the management profile. Tap **Allow** when prompted.

<p align="center"><img src="screenshots/176-ipad-en-cp-download-profile-allow.png" alt="Company Portal - Allow profile download" width="700"/></p>

<p align="center"><img src="screenshots/177-ipad-en-cp-download-profile-downloaded.png" alt="Company Portal - Profile downloaded" width="700"/></p>

<p align="center"><img src="screenshots/178-ipad-en-cp-download-profile-continue.png" alt="Company Portal - Continue to Company Portal" width="700"/></p>

<p align="center"><img src="screenshots/179-ipad-en-cp-setup-steps12-done.png" alt="Company Portal - Steps 1 and 2 done" width="700"/></p>

Go to **Settings → Profile Downloaded** to install the management profile.

<p align="center"><img src="screenshots/180-ipad-en-cp-setup-install-instructions.png" alt="Company Portal - Install instructions" width="700"/></p>

<p align="center"><img src="screenshots/181-ipad-en-settings-profile-downloaded.png" alt="Settings - Profile Downloaded" width="700"/></p>

<p align="center"><img src="screenshots/182-ipad-en-settings-profile-review.png" alt="Settings - Profile review" width="700"/></p>

Tap **Install** and enter the device passcode.

<p align="center"><img src="screenshots/183-ipad-en-settings-profile-passcode.png" alt="Settings - Enter passcode" width="700"/></p>

<p align="center"><img src="screenshots/184-ipad-en-settings-profile-install-confirm.png" alt="Settings - Install confirmation" width="700"/></p>

Review the MDM warning — Root Certificate and Mobile Device Management sections.

<p align="center"><img src="screenshots/185-ipad-en-settings-profile-warning-mdm.png" alt="Settings - MDM warning" width="700"/></p>

Tap **Trust** to confirm remote management enrollment.

<p align="center"><img src="screenshots/186-ipad-en-settings-profile-trust-remote.png" alt="Settings - Remote Management trust" width="700"/></p>

The profile is installed.

<p align="center"><img src="screenshots/187-ipad-en-settings-profile-installed.png" alt="Settings - Profile installed" width="700"/></p>

<p align="center"><img src="screenshots/188-ipad-en-settings-profile-view.png" alt="Settings - Profile view" width="700"/></p>

---

### 2.4 Complete enrollment

Return to Company Portal. Select the device category: `RootCA-Installed`.

<p align="center"><img src="screenshots/189-ipad-en-cp-device-category-selected.png" alt="Company Portal - Device category selected" width="700"/></p>

<p align="center"><img src="screenshots/190-ipad-en-cp-setup-step3-done.png" alt="Company Portal - Step 3 done" width="700"/></p>

Enrollment is complete — all four steps are checked.

<p align="center"><img src="screenshots/191-ipad-en-cp-enrollment-complete.png" alt="Company Portal - You're all set" width="700"/></p>

The device is enrolled and **can access company resources**.

<p align="center"><img src="screenshots/192-ipad-en-cp-device-compliant.png" alt="Company Portal - Device compliant" width="700"/></p>

---

## Part 3 — Validation

### 3.1 Profiles in Settings

Navigate to:
```
Settings → General → VPN & Device Management
```

The Management Profile should show:

| Content | Expected |
|---|---|
| Mobile Device Management | Present |
| Wi-Fi Network | `luconik-corp` |
| SCEP Device Identity Certificates | 2 certificates |
| Certificates | 2 + 5 certificates |
| ACME Device Identity Certificate | Present |

<p align="center"><img src="screenshots/218-ipad-en-settings-profile-deployed-full.png" alt="Settings - Full profile deployed" width="700"/></p>

---

### 3.2 Intune Admin Center

Navigate to:
```
Intune Admin Center → Devices → iOS/iPadOS devices
```

The device should appear as **Compliant**.

<p align="center"><img src="screenshots/195-intune-admin-ipad-devices-list-compliant.png" alt="Intune Admin - iOS devices list" width="900"/></p>

---

### 3.3 Aruba Central NAC

> [!NOTE]
> The Central NAC validation screenshots are in the [hpe-aruba-guides / central-nac-intune / ios](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune/ios/screenshots) repository.

The enrolled user should appear as **Accepted** in Central NAC → Monitoring → Clients, with:

| Field | Expected value |
|---|---|
| Status | Accepted |
| Authentication Type | EAP-TLS (Certificate) |
| Certificate Status | Valid |
| Assigned Role | per NAC policy (e.g. `admin-role`) |
| Identity Store | Luconik_EntraID |
| Matching Rule | admin |

> [!TIP]
> If the client appears as **Rejected** with `Deny All` matching rule, check that the SCEP profile uses **Certificate type: User** (not Device). See the note in [section 1.2](#12-scep-certificate).

---

## References

- 📘 [Aruba Central NAC — UEM Onboarding with Intune](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)
- [Microsoft Intune — SCEP Certificate Profiles for iOS](https://learn.microsoft.com/en-us/mem/intune/protect/certificates-scep-configure)
- [Microsoft Intune — iOS/iPadOS enrollment](https://learn.microsoft.com/en-us/mem/intune/enrollment/ios-enroll)
- [Apple Push Certificates Portal](https://identity.apple.com/pushcert/)

---

## File structure

```
eap-tls/ios/
├── README.md               ← This file (EN)
├── README-fr.md            ← French version
└── screenshots/
    ├── 131-ipad-appstore-search-intune.png     ← FR enrollment (reference)
    ├── ...
    ├── 164-ipad-en-appstore-search-intune.png  ← EN enrollment
    ├── ...
    ├── 196-intune-ios-trusted-create-profile-template.png
    ├── ...
    └── 223-central-nac-client-detail-accepted.png
```

---

*Last updated: May 2026 — [@Luconik](https://github.com/Luconik)*
