# EAP-TLS — Windows with Microsoft Intune + Aruba Central NAC

> 🇫🇷 [Français](README-fr.md) | 🇬🇧 English

![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-lightgrey?logo=windows)
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
- [Part 2 — Validation on Windows endpoint](#part-2--validation-on-windows-endpoint)
  - [2.1 Certificates in certmgr](#21-certificates-in-certmgr)
  - [2.2 Wi-Fi connection](#22-wi-fi-connection)
- [References](#references)

---

## Overview

This guide covers the **Microsoft Intune** side of the EAP-TLS configuration for Windows — Intune profiles (Trusted Certificate, SCEP, Wi-Fi) and endpoint validation.

```
Windows endpoint (Intune-managed)
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
> This guide covers Intune profile configuration only. For the one-time prerequisites (Entra ID App Registration), see [../../prerequisites/README.md](../../prerequisites/README.md). For the Aruba Central NAC configuration, see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

---

## Prerequisites

- One-time prerequisites completed — see [../../prerequisites/README.md](../../prerequisites/README.md)
- Aruba Central NAC configured — see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune)
- SCEP URL and root CA certificate retrieved from Central NAC (step 3.8 of the NAC guide)
- Microsoft Intune tenant active
- Windows 10 or Windows 11 device enrolled in Intune

---

## Part 1 — Intune Configuration Profiles

Three profiles must be created in Intune **in this order**:

| # | Profile type | Name | Purpose |
|---|---|---|---|
| 1 | Trusted Certificate | `Luconik Trusted` | Deploy the Central NAC root CA to the device |
| 2 | SCEP Certificate | `Luconik SCEP` | Request a client certificate from Central NAC |
| 3 | Wi-Fi | `Luconik Wi-Fi` | Configure 802.1X EAP-TLS on the SSID |

> [!IMPORTANT]
> Always create profiles in this order. The SCEP profile depends on the Trusted Certificate profile, and the Wi-Fi profile depends on both.

---

### 1.1 Trusted Certificate

Navigate to:
```
Intune Admin Center → Devices → Configuration → + Create profile → Trusted Certificate
```

<p align="center"><img src="screenshots/45-intune-trusted-certificate-create.png" alt="Intune - Create Trusted Certificate profile" width="900"/></p>

Set the profile name.

<p align="center"><img src="screenshots/46-intune-trusted-certificate-name.png" alt="Intune - Trusted Certificate name" width="900"/></p>

**Configuration settings** — import the root CA certificate (`.cer`) downloaded from Central NAC (step 3.8 of the NAC guide).

<p align="center"><img src="screenshots/47-intune-trusted-certificate-import.png" alt="Intune - Import CA certificate" width="900"/></p>

**Assignments** — assign to target users and devices.

<p align="center"><img src="screenshots/48-intune-trusted-certificate-assign-users-devices.png" alt="Intune - Assign to users and devices" width="900"/></p>

**Review + create** — verify the summary then click **Create**.

<p align="center"><img src="screenshots/49-intune-trusted-certificate-review-create.png" alt="Intune - Review and create" width="900"/></p>

---

### 1.2 SCEP Certificate

Navigate to:
```
Intune Admin Center → Devices → Configuration → + Create profile → SCEP Certificate
```

<p align="center"><img src="screenshots/50-intune-scep-certificate-profile-create.png" alt="Intune - Create SCEP Certificate profile" width="900"/></p>

**Configuration settings**

| Setting | Value |
|---|---|
| Certificate type | `User` |
| Subject name format | `CN={{UserPrincipalName}}` |
| Certificate validity period | `1 Years` |
| Key usage | `Digital Signature`, `Key Encipherment` |
| Key size (bits) | `2048` |
| Root Certificate | `Luconik Trusted` |
| Extended key usage | `Client Authentication` — `1.3.6.1.5.5.7.3.2` |
| Renewal threshold (%) | `20` |
| SCEP Server URLs | Central NAC SCEP URL (from step 3.8) |

<p align="center"><img src="screenshots/51-intune-scep-certificate-config1.png" alt="Intune - SCEP config 1" width="900"/></p>

<p align="center"><img src="screenshots/52-intune-scep-certificate-config2.png" alt="Intune - SCEP config 2" width="900"/></p>

<p align="center"><img src="screenshots/53-intune-scep-certificate-config3.png" alt="Intune - SCEP config 3" width="900"/></p>

Enter the SCEP Server URL retrieved from Central NAC.

<p align="center"><img src="screenshots/54-intune-scep-certificate-scep-url.png" alt="Intune - SCEP URL" width="900"/></p>

<p align="center"><img src="screenshots/55-intune-scep-certificate-scep-url-detail.png" alt="Intune - SCEP URL detail" width="900"/></p>

---

### 1.3 Wi-Fi

Navigate to:
```
Intune Admin Center → Devices → Configuration → + Create profile → Wi-Fi (Windows 10 and later)
```

<p align="center"><img src="screenshots/56-intune-wifi-profile-windows-create.png" alt="Intune - Create Wi-Fi profile" width="900"/></p>

**Configuration settings**

| Setting | Value |
|---|---|
| SSID | `luconik-corp` |
| Connect automatically | `Enable` |
| Security type | `WPA2-Enterprise` |
| EAP type | `EAP-TLS` |
| Certificate server names | `luconik-corp` |
| Root certificates for server validation | `Luconik Trusted` |
| Client certificates | `Luconik SCEP` |

<p align="center"><img src="screenshots/57-intune-wifi-profile-windows-config1.png" alt="Intune - Wi-Fi config 1" width="900"/></p>

<p align="center"><img src="screenshots/58-intune-wifi-profile-windows-config2.png" alt="Intune - Wi-Fi config 2" width="900"/></p>

<p align="center"><img src="screenshots/59-intune-wifi-profile-windows-config3.png" alt="Intune - Wi-Fi config 3" width="900"/></p>

<p align="center"><img src="screenshots/60-intune-wifi-profile-windows-config4.png" alt="Intune - Wi-Fi config 4" width="900"/></p>

<p align="center"><img src="screenshots/61-intune-wifi-profile-windows-config5.png" alt="Intune - Wi-Fi config 5" width="900"/></p>

<p align="center"><img src="screenshots/62-intune-wifi-profile-windows-config6.png" alt="Intune - Wi-Fi config 6" width="900"/></p>

---

## Part 2 — Validation on Windows endpoint

### 2.1 Certificates in certmgr

Open **certmgr.msc** on an Intune-managed Windows device.

In **Trusted Root Certification Authorities**, verify the Central NAC root CA is present.

<p align="center"><img src="screenshots/63-test-certmgr-root-user-certificate.png" alt="certmgr - Root CA certificate" width="900"/></p>

In **Personal → Certificates**, verify the SCEP client certificate is present and valid.

<p align="center"><img src="screenshots/64-test-certmgr-certificates-detail1.png" alt="certmgr - Certificate detail 1" width="900"/></p>

<p align="center"><img src="screenshots/65-test-certmgr-certificates-detail2.png" alt="certmgr - Certificate detail 2" width="900"/></p>

---

### 2.2 Wi-Fi connection

The `luconik-corp` SSID should appear in known Wi-Fi networks.

<p align="center"><img src="screenshots/66-test-windows-wifi-ssid-known-networks.png" alt="Windows - SSID in known networks" width="900"/></p>

The connection is established automatically using the EAP-TLS certificate.

<p align="center"><img src="screenshots/67-test-windows-wifi-connect-certificate.png" alt="Windows - Wi-Fi connected via EAP-TLS" width="900"/></p>

> [!NOTE]
> For Central NAC validation (client status, assigned role, authentication events), see [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune/windows/screenshots) — screenshots 68→70.

---

## References

- 📘 [Aruba Central NAC — UEM Onboarding with Intune](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)
- [Microsoft Intune — SCEP Certificate Profiles](https://learn.microsoft.com/en-us/mem/intune/protect/certificates-scep-configure)
- [Microsoft Intune — Wi-Fi profiles for Windows](https://learn.microsoft.com/en-us/mem/intune/configuration/wi-fi-settings-windows)

---

## File structure

```
eap-tls/windows/
├── README.md               ← This file (EN)
├── README-fr.md            ← French version
└── screenshots/
    ├── 45-intune-trusted-certificate-create.png
    ├── ...
    └── 67-test-windows-wifi-connect-certificate.png
```

---

*Last updated: May 2026 — [@Luconik](https://github.com/Luconik)*
