# 📡 SENSORS.md — Phase 2 : Intégration Sensorielle

> **Projet Ghost Home** | Documentation Technique  
> *Architecture Air-Gapped / Privacy-First*

---

## 🎯 Objectif

Déployer les modules sensoriels (Vue & Ouïe) en garantissant une **isolation réseau totale** vis-à-vis du Cloud. Aucune donnée ne quitte le périmètre local.

---

## 👁️ Module Vision : Caméra "Ghost Mode"

### Matériel

| Composant | Spécification |
|-----------|---------------|
| Modèle | **Reolink E1 Zoom** |
| Firmware | Version récente (mise à jour recommandée avant isolation) |
| Connectivité | Wi-Fi 2.4/5 GHz, WPA2 |

### Problématique

Le firmware Reolink impose deux contraintes majeures :

1. **Interface Web désactivée** — Les ports HTTP/HTTPS sont bloqués par défaut, forçant l'usage de l'application mobile propriétaire
2. **Dépendance Cloud** — Sans intervention, la caméra communique avec les serveurs Reolink
3. **Routeur FAI limité** — Aucune option de blocage internet par adresse MAC disponible

### 🛡️ Solution : Procédure "Ghost Gateway"

#### Étape 1 — Initialisation Contrôlée

Connexion temporaire via l'application mobile Reolink :

```
Réseau      : Wi-Fi Principal (WPA2)
Application : Reolink (iOS/Android)
Mode        : Configuration initiale
```

#### Étape 2 — Déverrouillage des Protocoles

Dans l'application mobile, naviguer vers les réglages avancés :

```
Paramètres > Réglages Avancés > Port Settings
```

Activer manuellement les protocoles suivants :

| Protocole | Port | Usage |
|-----------|------|-------|
| RTSP | 554 | Flux vidéo temps réel |
| RTMP | 1935 | Streaming |
| ONVIF | 8000 | Découverte & contrôle standardisé |
| HTTPS | 443 | Interface Web sécurisée |

#### Étape 3 — Isolation "Ghost Gateway"

> ⚠️ **WARNING : ÉTAPE CRITIQUE**  
> Cette manipulation coupe définitivement l'accès Cloud. La caméra restera accessible uniquement sur le réseau local. Toute reconfiguration nécessitera une réinitialisation usine.

Configuration réseau statique avec **passerelle volontairement invalide** :

```
Mode IP     : Statique
Adresse IP  : 192.168.1.X        # IP fixe sur le LAN
Masque      : 255.255.255.0
Passerelle  : 192.168.1.222      # ← FAUSSE GATEWAY (routeur = .1)
DNS 1       : 0.0.0.0            # ou 127.0.0.1
DNS 2       : 0.0.0.0
IPv6        : DÉSACTIVÉ
```

**Principe du hack** : En déclarant une passerelle inexistante, tout trafic destiné à sortir du LAN (→ Internet) est routé vers le vide. La caméra reste pleinement fonctionnelle localement.

#### Étape 4 — Intégration Home Assistant

```yaml
# Configuration via l'intégration officielle Reolink
Intégration : Reolink
Méthode     : IP Locale (192.168.1.X)
Protocole   : RTSP/ONVIF
```

### Validation

| Test | Résultat Attendu |
|------|------------------|
| Ping depuis le LAN | ✅ Réponse OK |
| Flux RTSP dans HA | ✅ Vidéo temps réel |
| Connexion via 4G (hors LAN) | ❌ Échec confirmé |
| App Reolink (hors LAN) | ❌ "Caméra hors ligne" |

**Statut : OPÉRATIONNEL — Isolation validée.**

---

## ⚡ Module Sensoriel : Infrastructure Zigbee

### Matériel

| Composant | Spécification |
|-----------|---------------|
| Modèle | **Sonoff Zigbee 3.0 USB Dongle Plus** |
| Variante | **Modèle P** (Texas Instruments) |
| Chipset | CC2652P |
| Interface | USB — CP210x UART Bridge |

### Pré-requis : Virtualisation KVM/QEMU

Le dongle USB branché sur l'hôte Linux n'est pas visible par défaut dans la VM Home Assistant.

#### Configuration USB Passthrough

```
Application : virt-manager
VM          : Home Assistant OS

Navigation :
  Détails de la VM > Ajouter un matériel > USB Host Device

Sélection :
  Périphérique : "CP210x UART Bridge" / "ITead Sonoff Zigbee"
```

Après ajout, redémarrer la VM pour prise en compte.

### Configuration Logicielle : ZHA

Utilisation de l'intégration native **Zigbee Home Automation (ZHA)**.

> ⚠️ **WARNING : SÉLECTION DU TYPE DE RADIO**  
> Le choix du protocole radio est **irréversible sans réinitialisation** du réseau Zigbee. Une erreur ici corrompra la configuration.

#### Tableau de Compatibilité Radio

| Modèle Dongle | Chipset | Type Radio ZHA |
|---------------|---------|----------------|
| Sonoff **P** | CC2652P (Texas Instruments) | **ZNP** ✅ |
| Sonoff **E** | EFR32MG21 (Silicon Labs) | EZSP |

```
Configuration ZHA :

Type de radio    : ZNP (Texas Instruments Z-Stack)
Port série       : /dev/ttyUSB0 (ou auto-détecté)
Débit            : 115200
Contrôle de flux : Matériel

Réseau Zigbee    : Nouveau réseau
Canal            : 15 (défaut)
```

### Validation

| Test | Résultat Attendu |
|------|------------------|
| Détection dongle dans HA | ✅ `/dev/ttyUSB0` visible |
| Initialisation ZHA | ✅ Réseau créé (Canal 15) |
| Appairage capteur test | ✅ Découverte < 60s |

**Statut : OPÉRATIONNEL — Maillage Zigbee actif.**

---

## 🛡️ Récapitulatif OpSec

### Matrice d'Isolation

| Module | Accès LAN | Accès WAN | Cloud Vendor |
|--------|-----------|-----------|--------------|
| Reolink E1 Zoom | ✅ | ❌ Bloqué | ❌ Aucun |
| Zigbee (ZHA) | ✅ | N/A | ❌ Aucun |
| Home Assistant | ✅ | Contrôlé | ❌ Aucun |

### Principes Appliqués

1. **Zero Trust Network** — Chaque périphérique IoT est considéré comme hostile par défaut
2. **Air-Gap Logique** — Isolation via configuration réseau (Ghost Gateway)
3. **Protocoles Ouverts** — RTSP, ONVIF, Zigbee = standards non-propriétaires
4. **Données Locales** — Aucun flux vidéo/capteur ne transite par Internet

---

## 📋 Checklist Phase 2

- [x] Caméra initialisée via app mobile
- [x] Ports RTSP/ONVIF/HTTPS activés
- [x] Ghost Gateway configurée (fausse passerelle)
- [x] Isolation WAN validée (test 4G)
- [x] Intégration Reolink dans Home Assistant
- [x] USB Passthrough configuré (virt-manager)
- [x] Dongle Zigbee détecté par la VM
- [x] ZHA configuré (ZNP / Canal 15)
- [x] Réseau Zigbee opérationnel

---

*Documentation générée pour le projet Ghost Home*  
*Souveraineté des données — Contrôle total — Zero Cloud*
