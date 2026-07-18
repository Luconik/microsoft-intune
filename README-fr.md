# Microsoft Intune × Aruba Central NAC

> 🇫🇷 Français | 🇬🇧 [English](README.md)

[![Microsoft Intune](https://img.shields.io/badge/Microsoft%20Intune-Gestion%20des%20terminaux-0078D4?logo=microsoft)](https://learn.microsoft.com/fr-fr/mem/intune/)
[![Aruba Central NAC](https://img.shields.io/badge/HPE%20Aruba-Central%20NAC-FF8300)](https://arubanetworking.hpe.com/techdocs/NAC/)
![Authentification](https://img.shields.io/badge/Authentification-EAP--TLS%20%7C%20802.1X-2EA44F)
![Plateformes](https://img.shields.io/badge/Plateformes-Windows%20%7C%20macOS%20%7C%20iOS-lightgrey)

Guides bilingues pas à pas pour déployer une authentification Wi-Fi par certificat avec **Microsoft Intune**, **Microsoft Entra ID** et **HPE Aruba Central NAC**.

> [!NOTE]
> Ce dépôt documente l’écosystème Microsoft et la configuration des terminaux. La configuration complémentaire d’Aruba Central NAC se trouve dans [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

## Périmètre du lab

```text
Terminal géré par Intune
        │
        │  Autorité de certification + certificat client SCEP + profil Wi-Fi
        ▼
Point d’accès Aruba / WPA2-Enterprise
        │
        │  802.1X / EAP-TLS
        ▼
Aruba Central NAC
        │
        │  Vérification OAuth2 de l’identité et de la conformité
        ▼
Microsoft Entra ID + Intune
        │
        ▼
Accès réseau et attribution du rôle
```

La documentation couvre le parcours complet :

1. Configurer les prérequis uniques Entra ID et Apple APNs.
2. Déployer depuis Intune les profils Certificat de confiance, SCEP et Wi-Fi.
3. Enrôler et valider les terminaux Windows, macOS et iOS/iPadOS.
4. Vérifier l’authentification par certificat, la conformité et l’attribution du rôle NAC.

## Par où commencer

| Étape | Guide | Périmètre |
|---|---|---|
| 1 | [Prérequis](prerequisites/README-fr.md) | App Registration Entra ID, DNS, permissions API, secret client et Apple APNs |
| 2 | [Vue d’ensemble EAP-TLS](eap-tls/) | Architecture et index des guides par plateforme |
| 3 | [Aruba Central NAC](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune) | Identity store, rôles, politiques d’autorisation, SSID et validation NAC |

## Guides par plateforme

| Plateforme | Configuration Intune | Parcours du terminal | Langues |
|---|---|---|---|
| Windows 10/11 | Certificat de confiance, SCEP, Wi-Fi | Validation des certificats et du Wi-Fi | [Français](eap-tls/windows/README-fr.md) · [English](eap-tls/windows/README.md) |
| macOS 14+ | Certificat de confiance, SCEP, Wi-Fi | Enrôlement Company Portal, Trousseaux, profils et conformité | [Français](eap-tls/macos/README-fr.md) · [English](eap-tls/macos/README.md) |
| iOS/iPadOS 16+ | Certificat de confiance, SCEP, Wi-Fi | Enrôlement Company Portal, profil MDM et conformité | [Français](eap-tls/ios/README-fr.md) · [English](eap-tls/ios/README.md) |

## Prérequis

- Un tenant Microsoft Intune actif
- Des droits d’administration Microsoft Entra ID
- HPE Aruba Central avec NAC activé
- Un point d’accès Aruba et un SSID Wi-Fi d’entreprise
- Un compte de test et un terminal enrôlé pour chaque plateforme
- Un identifiant Apple d’entreprise pour le renouvellement annuel du certificat APNs

> [!IMPORTANT]
> Les exemples utilisent des noms propres au lab, comme `luconik-corp`. Remplacez les identifiants de tenant, domaines, noms de certificats, SSID, URL et affectations par les valeurs adaptées à votre environnement. Ne publiez jamais de secret client ni d’identifiant de production.

## Structure du dépôt

```text
microsoft-intune/
├── README.md
├── README-fr.md
├── prerequisites/
│   ├── README.md
│   └── README-fr.md
└── eap-tls/
    ├── README.md
    ├── windows/
    ├── macos/
    └── ios/
```

Chaque dossier de plateforme contient les documentations française et anglaise ainsi que les captures utilisées pour la validation.

## Ressources associées

- [HPE Aruba Networking TechDocs — Central NAC](https://arubanetworking.hpe.com/techdocs/NAC/)
- [Microsoft Learn — Documentation Microsoft Intune](https://learn.microsoft.com/fr-fr/mem/intune/)
- [Microsoft Learn — Profils de certificats SCEP](https://learn.microsoft.com/fr-fr/mem/intune/protect/certificates-scep-configure)
- [Dépôt complémentaire — Guides HPE Aruba](https://github.com/Luconik/hpe-aruba-guides)

## Avertissement

Ce dépôt est une ressource indépendante de lab et d’apprentissage. Il ne constitue pas une documentation officielle HPE Aruba Networking ou Microsoft. Avant toute utilisation en production, validez les paramètres, les exigences de sécurité et le comportement des produits à l’aide de la documentation actuelle des éditeurs.

---

Maintenu par [Nicolas Culetto](https://github.com/Luconik) · Dernière mise à jour : mai 2026
