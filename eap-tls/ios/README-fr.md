# EAP-TLS — iOS/iPadOS avec Microsoft Intune + Aruba Central NAC

> 🇫🇷 Français | 🇬🇧 [English](README.md)

![Plateforme](https://img.shields.io/badge/plateforme-iOS%2F%20iPadOS%2016%2B-lightgrey?logo=apple)
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
- [Partie 2 — Enrôlement iOS/iPadOS via Company Portal](#partie-2--enrôlement-iosipados-via-company-portal)
  - [2.1 Installer Company Portal](#21-installer-company-portal)
  - [2.2 Se connecter et enrôler](#22-se-connecter-et-enrôler)
  - [2.3 Installer le profil de gestion](#23-installer-le-profil-de-gestion)
  - [2.4 Finaliser l'enrôlement](#24-finaliser-lenrôlement)
- [Partie 3 — Validation](#partie-3--validation)
- [Références](#références)

---

## Présentation

Ce guide couvre la partie **Microsoft Intune** de la configuration EAP-TLS pour iOS/iPadOS — profils Intune (Trusted Certificate, SCEP, Wi-Fi) et enrôlement via Company Portal.

```
Appareil iOS/iPadOS (géré par Intune)
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
> Ce guide couvre uniquement la configuration Intune. Pour la partie Aruba Central NAC (identity store, rôles, politiques d'autorisation, SSID), voir [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune).

> [!NOTE]
> Le certificat Apple MDM Push (APNs) est un prérequis unique pour toutes les plateformes Apple. Si le guide macOS a déjà été suivi, il est déjà configuré. Sinon, voir [eap-tls/macos/README-fr.md — Partie 0](../macos/README-fr.md#partie-0--certificat-apple-mdm-push).

---

## Prérequis

- Aruba Central NAC configuré — voir [hpe-aruba-guides / central-nac-intune](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune)
- Tenant Microsoft Intune actif
- **Certificat Apple MDM Push** configuré dans Intune — voir [guide macOS Partie 0](../macos/README-fr.md#partie-0--certificat-apple-mdm-push)
- iOS 16 / iPadOS 16 ou version ultérieure
- Application Microsoft **Company Portal** installée depuis l'App Store

---

## Partie 1 — Profils de configuration Intune

Trois profils doivent être créés dans Intune **dans cet ordre** :

| # | Type de profil | Nom | Rôle |
|---|---|---|---|
| 1 | Trusted Certificate | `Luconik Trusted - iOS` | Déployer le CA racine de Central NAC sur l'appareil |
| 2 | SCEP Certificate | `Luconik SCEP - iOS` | Demander un certificat client auprès de Central NAC |
| 3 | Wi-Fi | `Luconik Wi-Fi - iOS` | Configurer le 802.1X EAP-TLS sur le SSID |

> [!IMPORTANT]
> Toujours créer les profils dans cet ordre. Le profil SCEP dépend du profil Trusted Certificate, et le profil Wi-Fi dépend des deux.

---

### 1.1 Trusted Certificate

Naviguer vers :
```
Intune Admin Center → Appareils → Configuration → + Créer → iOS/iPadOS → Modèles → Trusted certificate
```

<p align="center"><img src="screenshots/196-intune-ios-trusted-create-profile-template.png" alt="Intune - Sélectionner Trusted certificate" width="900"/></p>

**Basics** — nom : `Luconik Trusted - iOS`.

<p align="center"><img src="screenshots/197-intune-ios-trusted-basics.png" alt="Intune - Trusted certificate basics" width="900"/></p>

**Paramètres de configuration** — uploader le certificat CA racine (`.cer`) téléchargé depuis Central NAC.

<p align="center"><img src="screenshots/198-intune-ios-trusted-config-cert-uploaded.png" alt="Intune - Trusted certificate upload" width="900"/></p>

**Affectations** — affecter à **Tous les appareils** et **Tous les utilisateurs**.

<p align="center"><img src="screenshots/199-intune-ios-trusted-assignments.png" alt="Intune - Trusted certificate affectations" width="900"/></p>

**Vérifier + créer** — vérifier le résumé puis cliquer sur **Créer**.

<p align="center"><img src="screenshots/200-intune-ios-trusted-review.png" alt="Intune - Trusted certificate révision" width="900"/></p>

---

### 1.2 SCEP Certificate

Naviguer vers :
```
Intune Admin Center → Appareils → Configuration → + Créer → iOS/iPadOS → Modèles → SCEP certificate
```

<p align="center"><img src="screenshots/201-intune-ios-scep-create-profile-template.png" alt="Intune - Sélectionner SCEP certificate" width="900"/></p>

**Basics** — nom : `Luconik SCEP - iOS`.

<p align="center"><img src="screenshots/202-intune-ios-scep-basics.png" alt="Intune - SCEP basics" width="900"/></p>

**Paramètres de configuration**

| Paramètre | Valeur |
|---|---|
| Type de certificat | `User` |
| Format du nom de l'objet | `CN={{UserPrincipalName}}` |
| Autre nom de l'objet | `URI` → `cnac+intune:///?DeviceId={{DeviceId}}` |
| Période de validité | `1 Ans` |
| Utilisation de la clé | `Digital Signature`, `Key Encipherment` |
| Taille de la clé (bits) | `2048` |
| Certificat racine | `Luconik Trusted - iOS` |
| Utilisation étendue de la clé | `Client Authentication` — `1.3.6.1.5.5.7.3.2` |
| Seuil de renouvellement (%) | `20` |
| URL du serveur SCEP | URL SCEP de Central NAC |

<p align="center"><img src="screenshots/203-intune-ios-scep-config-top.png" alt="Intune - SCEP config haut" width="900"/></p>

> [!NOTE]
> Le sélecteur de certificat racine liste les profils Trusted Certificate disponibles. Sélectionner `Luconik Trusted - iOS` créé à l'étape 1.1.

<p align="center"><img src="screenshots/204-intune-ios-scep-config-root-cert-picker.png" alt="Intune - SCEP sélecteur certificat racine" width="900"/></p>

<p align="center"><img src="screenshots/205-intune-ios-scep-config-eku-scep-url.png" alt="Intune - SCEP EKU et URL SCEP" width="900"/></p>

**Affectations** — affecter à **Tous les appareils** et **Tous les utilisateurs**.

<p align="center"><img src="screenshots/206-intune-ios-scep-assignments.png" alt="Intune - SCEP affectations" width="900"/></p>

**Vérifier + créer**

<p align="center"><img src="screenshots/207-intune-ios-scep-review.png" alt="Intune - SCEP révision" width="900"/></p>

> [!IMPORTANT]
> Le profil SCEP doit utiliser **Type de certificat : User** avec `CN={{UserPrincipalName}}`. L'utilisation du type `Device` entraîne un échec de l'autorisation Central NAC (règle Deny All) car le NAC ne peut pas mapper le certificat à un utilisateur ou groupe Entra ID.

---

### 1.3 Wi-Fi

Naviguer vers :
```
Intune Admin Center → Appareils → Configuration → + Créer → iOS/iPadOS → Modèles → Wi-Fi
```

<p align="center"><img src="screenshots/209-intune-ios-wifi-create-profile-template.png" alt="Intune - Sélectionner Wi-Fi" width="900"/></p>

**Basics** — nom : `Luconik Wi-Fi - iOS`.

<p align="center"><img src="screenshots/210-intune-ios-wifi-basics.png" alt="Intune - Wi-Fi basics" width="900"/></p>

**Paramètres de configuration**

| Paramètre | Valeur |
|---|---|
| Type Wi-Fi | `Enterprise` |
| Nom du réseau | `luconik-corp` |
| SSID | `luconik-corp` |
| Se connecter automatiquement | `Activer` |
| Réseau masqué | `Désactiver` |
| Type de sécurité | `WPA/WPA2-Enterprise` |
| Type EAP | `EAP - TLS` |
| Noms des serveurs de certificats | `luconik-corp` |
| Certificats racines pour la validation | `Luconik Trusted - iOS` |
| Méthode d'authentification | `Certificats` |
| Certificat client | `Luconik SCEP - iOS` |

<p align="center"><img src="screenshots/211-intune-ios-wifi-config-root-cert-picker.png" alt="Intune - Wi-Fi sélecteur cert racine" width="900"/></p>

<p align="center"><img src="screenshots/212-intune-ios-wifi-config-top.png" alt="Intune - Wi-Fi config haut" width="900"/></p>

<p align="center"><img src="screenshots/213-intune-ios-wifi-config-client-cert-picker.png" alt="Intune - Wi-Fi sélecteur cert client" width="900"/></p>

<p align="center"><img src="screenshots/214-intune-ios-wifi-config-bottom.png" alt="Intune - Wi-Fi config bas" width="900"/></p>

**Affectations** — affecter à **Tous les appareils** et **Tous les utilisateurs**.

<p align="center"><img src="screenshots/215-intune-ios-wifi-assignments.png" alt="Intune - Wi-Fi affectations" width="900"/></p>

**Vérifier + créer**

<p align="center"><img src="screenshots/216-intune-ios-wifi-review.png" alt="Intune - Wi-Fi révision" width="900"/></p>

Après création des trois profils, la liste des configurations doit afficher :

<p align="center"><img src="screenshots/217-intune-config-profiles-all-10-policies.png" alt="Intune - Tous les profils de configuration" width="900"/></p>

---

## Partie 2 — Enrôlement iOS/iPadOS via Company Portal

### 2.1 Installer Company Portal

Rechercher **Intune Company Portal** dans l'App Store et l'installer.

<p align="center"><img src="screenshots/164-ipad-en-appstore-search-intune.png" alt="App Store - Recherche Intune Company Portal" width="700"/></p>

<p align="center"><img src="screenshots/165-ipad-en-appstore-cp-listing.png" alt="App Store - Fiche Intune Company Portal" width="700"/></p>

<p align="center"><img src="screenshots/166-ipad-en-appstore-cp-open.png" alt="App Store - Ouvrir Company Portal" width="700"/></p>

---

### 2.2 Se connecter et enrôler

Ouvrir **Company Portal** et appuyer sur **Sign in**.

<p align="center"><img src="screenshots/167-ipad-en-cp-splash.png" alt="Company Portal - Écran d'accueil" width="700"/></p>

Se connecter avec le compte d'entreprise Entra ID.

<p align="center"><img src="screenshots/168-ipad-en-cp-signin-empty.png" alt="Company Portal - Connexion" width="700"/></p>

<p align="center"><img src="screenshots/169-ipad-en-cp-signin-email.png" alt="Company Portal - Email" width="700"/></p>

<p align="center"><img src="screenshots/170-ipad-en-cp-signin-password.png" alt="Company Portal - Mot de passe" width="700"/></p>

Approuver la demande MFA dans Microsoft Authenticator.

<p align="center"><img src="screenshots/171-ipad-en-cp-mfa-authenticator.png" alt="Company Portal - MFA Authenticator" width="700"/></p>

Autoriser les notifications lorsque demandé.

<p align="center"><img src="screenshots/172-ipad-en-cp-notifications-access.png" alt="Company Portal - Notifications" width="700"/></p>

Appuyer sur **Begin** pour démarrer l'enrôlement.

<p align="center"><img src="screenshots/173-ipad-en-cp-setup-begin.png" alt="Company Portal - Configurer l'accès MSFT" width="700"/></p>

Consulter les informations de confidentialité.

<p align="center"><img src="screenshots/174-ipad-en-cp-privacy-cant.png" alt="Company Portal - Confidentialité" width="700"/></p>

<p align="center"><img src="screenshots/175-ipad-en-cp-setup-step1-done.png" alt="Company Portal - Étape 1 terminée" width="700"/></p>

---

### 2.3 Installer le profil de gestion

Company Portal ouvre un navigateur pour télécharger le profil de gestion. Appuyer sur **Allow** lorsque demandé.

<p align="center"><img src="screenshots/176-ipad-en-cp-download-profile-allow.png" alt="Company Portal - Autoriser le téléchargement" width="700"/></p>

<p align="center"><img src="screenshots/177-ipad-en-cp-download-profile-downloaded.png" alt="Company Portal - Profil téléchargé" width="700"/></p>

<p align="center"><img src="screenshots/178-ipad-en-cp-download-profile-continue.png" alt="Company Portal - Continuer" width="700"/></p>

<p align="center"><img src="screenshots/179-ipad-en-cp-setup-steps12-done.png" alt="Company Portal - Étapes 1 et 2 terminées" width="700"/></p>

Aller dans **Réglages → Profil téléchargé** pour installer le profil de gestion.

<p align="center"><img src="screenshots/180-ipad-en-cp-setup-install-instructions.png" alt="Company Portal - Instructions d'installation" width="700"/></p>

<p align="center"><img src="screenshots/181-ipad-en-settings-profile-downloaded.png" alt="Réglages - Profil téléchargé" width="700"/></p>

<p align="center"><img src="screenshots/182-ipad-en-settings-profile-review.png" alt="Réglages - Révision du profil" width="700"/></p>

Appuyer sur **Installer** et saisir le code d'accès de l'appareil.

<p align="center"><img src="screenshots/183-ipad-en-settings-profile-passcode.png" alt="Réglages - Code d'accès" width="700"/></p>

<p align="center"><img src="screenshots/184-ipad-en-settings-profile-install-confirm.png" alt="Réglages - Confirmation installation" width="700"/></p>

Consulter l'avertissement MDM — sections Certificat racine et Gestion des appareils mobiles.

<p align="center"><img src="screenshots/185-ipad-en-settings-profile-warning-mdm.png" alt="Réglages - Avertissement MDM" width="700"/></p>

Appuyer sur **Trust** pour confirmer l'inscription à la gestion à distance.

<p align="center"><img src="screenshots/186-ipad-en-settings-profile-trust-remote.png" alt="Réglages - Faire confiance à la gestion à distance" width="700"/></p>

Le profil est installé.

<p align="center"><img src="screenshots/187-ipad-en-settings-profile-installed.png" alt="Réglages - Profil installé" width="700"/></p>

<p align="center"><img src="screenshots/188-ipad-en-settings-profile-view.png" alt="Réglages - Vue du profil" width="700"/></p>

---

### 2.4 Finaliser l'enrôlement

Retourner dans Company Portal. Sélectionner la catégorie d'appareil : `RootCA-Installed`.

<p align="center"><img src="screenshots/189-ipad-en-cp-device-category-selected.png" alt="Company Portal - Catégorie sélectionnée" width="700"/></p>

<p align="center"><img src="screenshots/190-ipad-en-cp-setup-step3-done.png" alt="Company Portal - Étape 3 terminée" width="700"/></p>

L'enrôlement est terminé — les quatre étapes sont cochées.

<p align="center"><img src="screenshots/191-ipad-en-cp-enrollment-complete.png" alt="Company Portal - C'est prêt !" width="700"/></p>

L'appareil est enrôlé et **peut accéder aux ressources d'entreprise**.

<p align="center"><img src="screenshots/192-ipad-en-cp-device-compliant.png" alt="Company Portal - Appareil conforme" width="700"/></p>

---

## Partie 3 — Validation

### 3.1 Profils dans Réglages

Naviguer vers :
```
Réglages → Général → VPN et gestion de l'appareil
```

Le profil de gestion doit afficher :

| Contenu | Attendu |
|---|---|
| Gestion des appareils mobiles | Présent |
| Réseau Wi-Fi | `luconik-corp` |
| Certificats SCEP d'identité d'appareil | 2 certificats |
| Certificats | 2 + 5 certificats |
| Certificat ACME d'identité d'appareil | Présent |

<p align="center"><img src="screenshots/218-ipad-en-settings-profile-deployed-full.png" alt="Réglages - Profil complet déployé" width="700"/></p>

---

### 3.2 Intune Admin Center

Naviguer vers :
```
Intune Admin Center → Appareils → Appareils iOS/iPadOS
```

L'appareil doit apparaître comme **Conforme**.

<p align="center"><img src="screenshots/195-intune-admin-ipad-devices-list-compliant.png" alt="Intune Admin - Liste appareils iOS" width="900"/></p>

---

### 3.3 Aruba Central NAC

> [!NOTE]
> Les captures de validation Central NAC sont dans le dépôt [hpe-aruba-guides / central-nac-intune / ios](https://github.com/Luconik/hpe-aruba-guides/tree/main/central-nac-intune/ios/screenshots).

L'utilisateur enrôlé doit apparaître comme **Accepted** dans Central NAC → Monitoring → Clients, avec :

| Champ | Valeur attendue |
|---|---|
| Statut | Accepted |
| Type d'authentification | EAP-TLS (Certificate) |
| Statut du certificat | Valide |
| Rôle assigné | selon la politique NAC (ex. `admin-role`) |
| Identity Store | Luconik_EntraID |
| Matching Rule | admin |

> [!TIP]
> Si le client apparaît comme **Rejected** avec la règle `Deny All`, vérifier que le profil SCEP utilise **Type de certificat : User** (pas Device). Voir la note en [section 1.2](#12-scep-certificate).

---

## Références

- 📘 [Aruba Central NAC — UEM Onboarding with Intune](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)
- [Microsoft Intune — Profils de certificat SCEP pour iOS](https://learn.microsoft.com/fr-fr/mem/intune/protect/certificates-scep-configure)
- [Microsoft Intune — Enrôlement iOS/iPadOS](https://learn.microsoft.com/fr-fr/mem/intune/enrollment/ios-enroll)
- [Portail Apple Push Certificates](https://identity.apple.com/pushcert/)

---

## Structure des fichiers

```
eap-tls/ios/
├── README.md               ← Version anglaise
├── README-fr.md            ← Ce fichier (FR)
└── screenshots/
    ├── 131-ipad-appstore-search-intune.png     ← Enrôlement FR (référence)
    ├── ...
    ├── 164-ipad-en-appstore-search-intune.png  ← Enrôlement EN
    ├── ...
    ├── 196-intune-ios-trusted-create-profile-template.png
    ├── ...
    └── 223-central-nac-client-detail-accepted.png
```

---

*Dernière mise à jour : Mai 2026 — [@Luconik](https://github.com/Luconik)*
