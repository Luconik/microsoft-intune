# Microsoft Intune × Aruba Central NAC

[![Microsoft Intune](https://img.shields.io/badge/Microsoft%20Intune-Device%20Management-0078D4?logo=microsoft)](https://learn.microsoft.com/mem/intune/)
[![Aruba Central NAC](https://img.shields.io/badge/HPE%20Aruba-Central%20NAC-FF8300)](https://arubanetworking.hpe.com/techdocs/NAC/)
![Authentication](https://img.shields.io/badge/Authentication-EAP--TLS%20%7C%20802.1X-2EA44F)
![Platforms](https://img.shields.io/badge/Platforms-Windows%20%7C%20macOS%20%7C%20iOS-lightgrey)

Step-by-step, bilingual guides for deploying certificate-based Wi-Fi authentication with **Microsoft Intune**, **Microsoft Entra ID**, and **HPE Aruba Central NAC**.

> [!NOTE]
> This repository documents the Microsoft ecosystem and endpoint configuration. The complementary Aruba Central NAC configuration lives in [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

## What this lab covers

```text
Intune-managed endpoint
        │
        │  Trusted CA + SCEP client certificate + Wi-Fi profile
        ▼
Aruba AP / WPA2-Enterprise
        │
        │  802.1X / EAP-TLS
        ▼
Aruba Central NAC
        │
        │  OAuth2 identity and compliance lookup
        ▼
Microsoft Entra ID + Intune
        │
        ▼
Network access and role assignment
```

The documentation follows the complete deployment path:

1. Configure the one-time Entra ID and Apple APNs prerequisites.
2. Deploy Trusted Certificate, SCEP, and Wi-Fi profiles from Intune.
3. Enroll and validate Windows, macOS, and iOS/iPadOS endpoints.
4. Verify certificate authentication, compliance, and NAC role assignment.

## Start here

| Step | Guide | Scope |
|---|---|---|
| 1 | [Prerequisites](prerequisites/) | Entra ID App Registration, DNS records, API permissions, client secret, and Apple APNs |
| 2 | [EAP-TLS overview](eap-tls/) | Architecture and platform guide index |
| 3 | [Aruba Central NAC](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune) | Identity store, roles, authorization policies, SSID, and NAC validation |

## Platform guides

| Platform | Intune configuration | Endpoint workflow | Languages |
|---|---|---|---|
| Windows 10/11 | Trusted Certificate, SCEP, Wi-Fi | Certificate and Wi-Fi validation | [English](eap-tls/windows/README.md) · [Français](eap-tls/windows/README-fr.md) |
| macOS 14+ | Trusted Certificate, SCEP, Wi-Fi | Company Portal enrollment, Keychain, profiles, and compliance | [English](eap-tls/macos/README.md) · [Français](eap-tls/macos/README-fr.md) |
| iOS/iPadOS 16+ | Trusted Certificate, SCEP, Wi-Fi | Company Portal enrollment, MDM profile, and compliance | [English](eap-tls/ios/README.md) · [Français](eap-tls/ios/README-fr.md) |

## Prerequisites

- An active Microsoft Intune tenant
- Microsoft Entra ID administrative access
- HPE Aruba Central with NAC enabled
- An Aruba AP and an enterprise Wi-Fi SSID
- A test account and enrolled endpoint for each target platform
- A corporate Apple ID for the yearly APNs certificate renewal when deploying Apple devices

> [!IMPORTANT]
> Examples use lab-specific names such as `luconik-corp`. Replace tenant IDs, domains, certificate names, SSIDs, URLs, and assignments with values appropriate for your environment. Never commit client secrets or production credentials.

## Repository structure

```text
microsoft-intune/
├── README.md
├── prerequisites/
│   ├── README.md
│   └── README-fr.md
└── eap-tls/
    ├── README.md
    ├── windows/
    ├── macos/
    └── ios/
```

Each platform folder contains English and French documentation plus the screenshots used for validation.

## Related resources

- [HPE Aruba Networking TechDocs — Central NAC](https://arubanetworking.hpe.com/techdocs/NAC/)
- [Microsoft Learn — Microsoft Intune documentation](https://learn.microsoft.com/mem/intune/)
- [Microsoft Learn — SCEP certificate profiles](https://learn.microsoft.com/mem/intune/protect/certificates-scep-configure)
- [Companion repository — HPE Aruba guides](https://github.com/Luconik/hpe-aruba-guides)

## Disclaimer

This is an independent lab and learning resource. It is not official HPE Aruba Networking or Microsoft documentation. Validate settings, security requirements, and product behavior against current vendor documentation before using them in production.

---

Maintained by [Nicolas Culetto](https://github.com/Luconik) · Last updated: May 2026
