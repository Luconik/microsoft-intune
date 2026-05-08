# EAP-TLS — Windows avec Microsoft Intune + Aruba Central NAC

> 🇫🇷 Français | 🇬🇧 [English](README.md)

![Plateforme](https://img.shields.io/badge/plateforme-Windows%2010%2F11-lightgrey?logo=windows)
![Intune](https://img.shields.io/badge/Microsoft%20Intune-requis-blue?logo=microsoft)
![Auth](https://img.shields.io/badge/auth-EAP--TLS%20%2F%20802.1X-green)
![Mise à jour](https://img.shields.io/badge/mis%20à%20jour-Mai%202026-orange)

---

## Table des matières

- [Présentation](#présentation)
- [Prérequis](#prérequis)
- [Partie 1 — Profils de configuration Intune](#partie-1--profils-de-configuration-intune)
  - [1.1 Trusted Certificate](#11-trusted-certificate)
  - [1.2 SCEP Certificate](#12-scep-certificate)
  - [1.3 Wi-Fi](#13-wi-fi)
- [Partie 2 — Validation sur le poste Windows](#partie-2--validation-sur-le-poste-windows)
  - [2.1 Certificats dans certmgr](#21-certificats-dans-certmgr)
  - [2.2 Connexion Wi-Fi](#22-connexion-wi-fi)
- [Références](#références)

---

## Présentation

Ce guide couvre la partie **Microsoft Intune** de la configuration EAP-TLS pour Windows — profils Intune (Trusted Certificate, SCEP, Wi-Fi) et validation sur le poste.

```
Poste Windows (géré par Intune)
    │
    │  Certificat SCEP délivré par le CA Central NAC
    ▼
Aruba AP (802.1X EAP-TLS)
    │
    │  Authentification RADIUS
    ▼
Aruba Central NAC
    │
    │  Vérification de conformité via OAuth2
    ▼
Microsoft Intune / Entra ID
    │
    ▼
Accès réseau accordé (rôle assigné par la politique NAC)
```

> [!NOTE]
> Ce guide couvre uniquement la configuration des profils Intune. Pour les prérequis uniques (App Registration Entra ID), voir [../../../prerequisites/README-fr.md](../../../prerequisites/README-fr.md). Pour la configuration Aruba Central NAC, voir [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

---

## Prérequis

- Prérequis uniques complétés — voir [../../../prerequisites/README-fr.md](../../../prerequisites/README-fr.md)
- Aruba Central NAC configuré — voir [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune)
- URL SCEP et certificat CA racine récupérés depuis Central NAC (étape 3.8 du guide NAC)
- Tenant Microsoft Intune actif
- Poste Windows 10 ou Windows 11 enrôlé dans Intune

---

## Partie 1 — Profils de configuration Intune

Trois profils doivent être créés dans Intune **dans cet ordre** :

| # | Type de profil | Nom | Rôle |
|---|---|---|---|
| 1 | Trusted Certificate | `Luconik Trusted` | Déployer le CA racine de Central NAC sur l'appareil |
| 2 | SCEP Certificate | `Luconik SCEP` | Demander un certificat client auprès de Central NAC |
| 3 | Wi-Fi | `Luconik Wi-Fi` | Configurer le 802.1X EAP-TLS sur le SSID |

> [!IMPORTANT]
> Toujours créer les profils dans cet ordre. Le profil SCEP dépend du profil Trusted Certificate, et le profil Wi-Fi dépend des deux.

---

### 1.1 Trusted Certificate

Naviguer vers :
```
Intune Admin Center → Appareils → Configuration → + Créer un profil → Trusted Certificate
```

<p align="center"><img src="screenshots/45-intune-trusted-certificate-create.png" alt="Intune - Créer profil Trusted Certificate" width="900"/></p>

Définir le nom du profil.

<p align="center"><img src="screenshots/46-intune-trusted-certificate-name.png" alt="Intune - Nom Trusted Certificate" width="900"/></p>

**Paramètres de configuration** — importer le certificat CA racine (`.cer`) téléchargé depuis Central NAC (étape 3.8 du guide NAC).

<p align="center"><img src="screenshots/47-intune-trusted-certificate-import.png" alt="Intune - Import certificat CA" width="900"/></p>

**Affectations** — affecter aux utilisateurs et appareils cibles.

<p align="center"><img src="screenshots/48-intune-trusted-certificate-assign-users-devices.png" alt="Intune - Affecter aux utilisateurs et appareils" width="900"/></p>

**Vérifier + créer** — vérifier le résumé puis cliquer sur **Créer**.

<p align="center"><img src="screenshots/49-intune-trusted-certificate-review-create.png" alt="Intune - Vérifier et créer" width="900"/></p>

---

### 1.2 SCEP Certificate

Naviguer vers :
```
Intune Admin Center → Appareils → Configuration → + Créer un profil → SCEP Certificate
```

<p align="center"><img src="screenshots/50-intune-scep-certificate-profile-create.png" alt="Intune - Créer profil SCEP Certificate" width="900"/></p>

**Paramètres de configuration**

| Paramètre | Valeur |
|---|---|
| Type de certificat | `User` |
| Format du nom de l'objet | `CN={{UserPrincipalName}}` |
| Période de validité | `1 Ans` |
| Utilisation de la clé | `Digital Signature`, `Key Encipherment` |
| Taille de la clé (bits) | `2048` |
| Certificat racine | `Luconik Trusted` |
| Utilisation étendue de la clé | `Client Authentication` — `1.3.6.1.5.5.7.3.2` |
| Seuil de renouvellement (%) | `20` |
| URL du serveur SCEP | URL SCEP de Central NAC (étape 3.8) |

<p align="center"><img src="screenshots/51-intune-scep-certificate-config1.png" alt="Intune - SCEP config 1" width="900"/></p>

<p align="center"><img src="screenshots/52-intune-scep-certificate-config2.png" alt="Intune - SCEP config 2" width="900"/></p>

<p align="center"><img src="screenshots/53-intune-scep-certificate-config3.png" alt="Intune - SCEP config 3" width="900"/></p>

Renseigner l'URL SCEP récupérée depuis Central NAC.

<p align="center"><img src="screenshots/54-intune-scep-certificate-scep-url.png" alt="Intune - URL SCEP" width="900"/></p>

<p align="center"><img src="screenshots/55-intune-scep-certificate-scep-url-detail.png" alt="Intune - Détail URL SCEP" width="900"/></p>

---

### 1.3 Wi-Fi

Naviguer vers :
```
Intune Admin Center → Appareils → Configuration → + Créer un profil → Wi-Fi (Windows 10 et versions ultérieures)
```

<p align="center"><img src="screenshots/56-intune-wifi-profile-windows-create.png" alt="Intune - Créer profil Wi-Fi" width="900"/></p>

**Paramètres de configuration**

| Paramètre | Valeur |
|---|---|
| SSID | `luconik-corp` |
| Se connecter automatiquement | `Activer` |
| Type de sécurité | `WPA2-Enterprise` |
| Type EAP | `EAP-TLS` |
| Noms des serveurs de certificats | `luconik-corp` |
| Certificats racines pour la validation | `Luconik Trusted` |
| Certificats client | `Luconik SCEP` |

<p align="center"><img src="screenshots/57-intune-wifi-profile-windows-config1.png" alt="Intune - Wi-Fi config 1" width="900"/></p>

<p align="center"><img src="screenshots/58-intune-wifi-profile-windows-config2.png" alt="Intune - Wi-Fi config 2" width="900"/></p>

<p align="center"><img src="screenshots/59-intune-wifi-profile-windows-config3.png" alt="Intune - Wi-Fi config 3" width="900"/></p>

<p align="center"><img src="screenshots/60-intune-wifi-profile-windows-config4.png" alt="Intune - Wi-Fi config 4" width="900"/></p>

<p align="center"><img src="screenshots/61-intune-wifi-profile-windows-config5.png" alt="Intune - Wi-Fi config 5" width="900"/></p>

<p align="center"><img src="screenshots/62-intune-wifi-profile-windows-config6.png" alt="Intune - Wi-Fi config 6" width="900"/></p>

---

## Partie 2 — Validation sur le poste Windows

### 2.1 Certificats dans certmgr

Ouvrir **certmgr.msc** sur un poste Windows géré par Intune.

Dans **Autorités de certification racines de confiance**, vérifier la présence du CA racine Central NAC.

<p align="center"><img src="screenshots/63-test-certmgr-root-user-certificate.png" alt="certmgr - Certificat CA racine" width="900"/></p>

Dans **Personnel → Certificats**, vérifier la présence et la validité du certificat client SCEP.

<p align="center"><img src="screenshots/64-test-certmgr-certificates-detail1.png" alt="certmgr - Détail certificat 1" width="900"/></p>

<p align="center"><img src="screenshots/65-test-certmgr-certificates-detail2.png" alt="certmgr - Détail certificat 2" width="900"/></p>

---

### 2.2 Connexion Wi-Fi

Le SSID `luconik-corp` doit apparaître dans les réseaux Wi-Fi connus.

<p align="center"><img src="screenshots/66-test-windows-wifi-ssid-known-networks.png" alt="Windows - SSID dans les réseaux connus" width="900"/></p>

La connexion s'établit automatiquement via le certificat EAP-TLS.

<p align="center"><img src="screenshots/67-test-windows-wifi-connect-certificate.png" alt="Windows - Wi-Fi connecté via EAP-TLS" width="900"/></p>

> [!NOTE]
> Pour la validation dans Aruba Central NAC (statut client, rôle assigné, événements d'authentification), voir [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune/windows/screenshots) — captures 68→70.

---

## Références

- 📘 [Aruba Central NAC — UEM Onboarding with Intune](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)
- [Microsoft Intune — Profils de certificat SCEP](https://learn.microsoft.com/fr-fr/mem/intune/protect/certificates-scep-configure)
- [Microsoft Intune — Profils Wi-Fi pour Windows](https://learn.microsoft.com/fr-fr/mem/intune/configuration/wi-fi-settings-windows)

---

## Structure des fichiers

```
eap-tls/windows/
├── README.md               ← Version anglaise
├── README-fr.md            ← Ce fichier (FR)
└── screenshots/
    ├── 45-intune-trusted-certificate-create.png
    ├── ...
    └── 67-test-windows-wifi-connect-certificate.png
```

---

*Dernière mise à jour : Mai 2026 — [@Luconik](https://github.com/Luconik)*
