<div align="center">

# ☁️ OpenStack Cloud Privé

**Déploiement d'une infrastructure cloud privée avec OpenStack & DevStack**

[![OpenStack](https://img.shields.io/badge/OpenStack-ED1944?style=for-the-badge&logo=openstack&logoColor=white)](https://openstack.org)
[![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)](https://virtualbox.org)
[![DevStack](https://img.shields.io/badge/DevStack-FF6600?style=for-the-badge&logo=openstack&logoColor=white)](https://docs.openstack.org/devstack)

</div>

---

![1_3lkRxLi9DK0RQXh1aZOx0A(1_3lkRxLi9DK0RQXh1aZOx0A.png)

## 

- [Introduction](#-introduction)
- [Objectifs du Projet](#-objectifs-du-projet)
- [Architecture Générale](#️-architecture-générale)
- [Composants OpenStack](#-composants-openstack-déployés)
- [Installation avec DevStack](#️-installation-avec-devstack)
- [Interface Horizon — Dashboard](#️-interface-horizon--dashboard)
- [Configuration du Réseau Virtuel](#-configuration-du-réseau-virtuel)
- [Gestion des VMs](#-gestion-des-images-et-création-des-machines-virtuelles)
- [Tests et Validation](#-tests-et-validation)
- [Gestion des Ressources](#-gestion-des-ressources)
- [Captures d'Installation](#-captures-décran--installation)
- [Captures OpenStack](#-captures-décran--openstack-horizon)
- [Conclusion](#-conclusion-et-perspectives)

---

## Introduction

Ce projet, réalisé dans le cadre **Administration Systèmes & Réseaux**, porte sur la mise en place d'un **cloud privé complet** à l'aide d'**OpenStack** déployé via **DevStack** sur Ubuntu 24.04 LTS.

**OpenStack** est une plateforme cloud open-source de référence dans l'industrie, offrant une suite complète de services **IaaS (Infrastructure as a Service)**. Elle est utilisée par de grandes organisations pour gérer des infrastructures cloud privées et publiques à grande échelle.

L'objectif principal est de comprendre et de déployer une infrastructure cloud complète : de l'installation des composants OpenStack jusqu'à la gestion des machines virtuelles, du réseau virtuel et du stockage.

---

## Objectifs du Projet

| # | Objectif |
|---|----------|
| 1 | Installer et configurer **OpenStack via DevStack** sur une machine virtuelle Ubuntu 24.04 |
| 2 | Déployer les services essentiels : **Keystone, Nova, Neutron, Glance, Cinder, Horizon** |
| 3 | Créer et gérer des **machines virtuelles** via l'interface Horizon et la ligne de commande |
| 4 | Configurer le **réseau virtuel** : réseaux provider, privé, routeurs et IPs flottantes |
| 5 | Gérer les ressources : **flavors, volumes de stockage, quotas** par projet |
| 6 | Valider le déploiement par des **tests de connectivité et SSH** |

---

## Architecture Générale

L'architecture retenue est une installation **All-in-One (AIO)**, où tous les services OpenStack sont déployés sur un seul nœud. Cette approche est idéale pour un environnement de développement et d'apprentissage.

```
┌─────────────────────────────────────────────────────────┐
│                   Windows 10/11 (Hôte)                  │
│                   Oracle VirtualBox                     │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │           Ubuntu 24.04 LTS — Machine Virtuelle    │  │
│  │         4 vCPUs · 8+ Go RAM · 80+ Go Disque       │  │
│  │                                                   │  │
│  │   ┌─────────────────────────────────────────┐     │  │
│  │   │        DevStack — OpenStack AIO         │     │  │
│  │   │  Keystone · Nova · Neutron · Glance     │     │  │
│  │   │       Cinder · Horizon                  │     │  │
│  │   │                                         │     │  │
│  │   │  [VM1] 10.0.0.138 ←→ [VM2] 10.0.0.x     │     │  │
│  │   │         ↕ Réseau privé 10.0.0.0/24      │     │  │
│  │   │     [Routeur R1] ←→ public-network      │     │  │
│  │   └─────────────────────────────────────────┘     │  │
│  │                                                   │  │
│  │   Accès : http://<IP>/dashboard (Horizon)         │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

| Élément | Détail |
|---------|--------|
| **Environnement hôte** | Windows 10/11 avec Oracle VirtualBox |
| **Machine virtuelle** | Ubuntu 24.04 LTS (Noble Numbat) |
| **Ressources VM** | 4 vCPUs, 8+ Go RAM, 80+ Go disque |
| **Plateforme cloud** | DevStack (distribution de développement OpenStack) |
| **Accès** | Interface web Horizon via `http://<IP>/dashboard` |

---

## Composants OpenStack Déployés

| Composant | Nom du Service | Rôle |
|-----------|---------------|------|
| **Keystone** | Identity Service | Authentification et autorisation des utilisateurs et services |
| **Nova** | Compute Service | Gestion du cycle de vie des machines virtuelles (VMs) |
| **Neutron** | Networking Service | Réseau virtuel SDN : réseaux, sous-réseaux, routeurs |
| **Glance** | Image Service | Stockage et gestion des images disques (cirros, Ubuntu…) |
| **Cinder** | Block Storage | Volumes de stockage persistants attachables aux VMs |
| **Horizon** | Dashboard | Interface graphique web d'administration et de gestion |

---

## Installation avec DevStack

DevStack est un outil de déploiement rapide d'OpenStack destiné aux environnements de développement et de test. Il automatise l'installation de tous les composants OpenStack à partir des sources.

### Prérequis Système

| Ressource | Minimum | Recommandé |
|-----------|---------|------------|
| **OS** | Ubuntu 24.04 LTS | Ubuntu 24.04 LTS |
| **RAM** | 8 Go | 16 Go |
| **CPU** | 4 vCPUs (VT-x/AMD-V activé) | 8 vCPUs |
| **Disque** | 50 Go | 80+ Go |
| **Réseau** | Connexion Internet active | Bridge ou Host-Only VirtualBox |

---

### Étapes d'Installation

#### Étape 1 — Mise à jour du système et création de l'utilisateur `stack`

```bash
sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo apt update && sudo apt upgrade -y
```

#### Étape 2 — Installation des dépendances

```bash
sudo apt install -y git curl net-tools
```

#### Étape 3 — Clonage du dépôt DevStack

```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

#### Étape 4 — Configuration du fichier `local.conf`

```ini
[[local|localrc]]
ADMIN_PASSWORD=ENSAb2024
DATABASE_PASSWORD=ENSAb2024
RABBIT_PASSWORD=ENSAb2024
SERVICE_PASSWORD=ENSAb2024
enable_service horizon
LOGFILE=$DEST/logs/stack.sh.log
LOGDAYS=2
```

#### Étape 5 — Lancement de l'installation

```bash
./stack.sh
```

> La durée d'installation est de **15 à 30 minutes**. Une fois terminé, OpenStack est accessible via `http://<IP>/dashboard` avec les identifiants `admin / ENSAb2024`.

---

## Interface Horizon — Dashboard

**Horizon** est l'interface graphique web d'OpenStack. Elle permet aux administrateurs et utilisateurs de gérer l'ensemble des ressources cloud sans recourir à la ligne de commande.

Après connexion en tant qu'administrateur, le tableau de bord affiche la **synthèse des quotas** :
- **Compute** : instances, vCPUs, RAM
- **Volume** : volumes, instantanés
- **Réseau** : IPs flottantes, groupes de sécurité, réseaux, routeurs

---

## Configuration du Réseau Virtuel

**Neutron** est le service réseau d'OpenStack. Il implémente le **Software Defined Networking (SDN)** pour créer et gérer des réseaux virtuels isolés, des sous-réseaux, des routeurs et des IPs flottantes.

### 4.1 Réseau Public (Provider/Externe)

Le réseau **provider** est le réseau externe permettant la communication avec l'extérieur. Il est créé par l'administrateur en cochant `External Network`.

### 4.2 Réseau Privé

Le réseau privé **10.0.0.0/24** est créé pour les communications internes entre VMs. Ce réseau est connecté au routeur **R1** qui assure le routage vers le réseau public.

### 4.3 Routeur R1

Le routeur Neutron (**R1**) assure la connexion entre le réseau privé des VMs et le réseau public externe. La gateway externe est configurée sur `public-network`.

```
[VM1] [VM2] ─── réseau privé 10.0.0.0/24 ─── [Routeur R1] ─── public-network ─── Internet
```

### 4.4 Groupe de Sécurité

Le groupe **`ssh-ping-security-group`** autorise :
- Connexions **SSH** (port 22/TCP)
- Requêtes **ICMP** (ping)

---

## Gestion des Images et Création des Machines Virtuelles

### 5.1 Image Glance (cirros)

**Glance** est le service de gestion des images disques. L'image **cirros** (système minimal pour tests cloud) est importée au format **QCOW2** (format natif KVM/QEMU).

### 5.2 Paire de Clés SSH

Une paire de clés asymétriques est générée pour sécuriser l'accès SSH. La clé publique est injectée dans la VM au démarrage via Nova.

### 5.3 Lancement d'une Instance (VM)

La création d'une VM passe par l'assistant **"Lancer Instance"** d'Horizon :

| Étape | Configuration |
|-------|--------------|
| **Détails** | Nom de l'instance |
| **Source** | Image cirros (QCOW2, ~20 Mo) |
| **Gabarit** | Flavor (m1.tiny / m1.small…) |
| **Réseaux** | private-network |
| **Sécurité** | ssh-ping-security-group |
| **Paire de clés** | Clé SSH générée |

### 5.4 IPs Flottantes

Les IPs flottantes permettent d'accéder aux VMs depuis l'extérieur :
- **VM1** → IP flottante : `172.24.4.168`
- **VM2** → IP flottante : `172.24.4.122`

---

## ✓ Tests et Validation

### Connexion SSH depuis Windows

```powershell
ssh -i "C:\Users\Fatima Zahraa\.ssh\ssh-key.pem" cirros@172.24.4.168
```

### Tests de connectivité réseau (depuis VM2)

```bash
# Test Internet — Google DNS
ping 8.8.8.8    # 3 paquets — 0% perte ✔

# Test inter-VMs — IP privée de VM1
ping 10.0.0.138 # 4 paquets — 0% perte ✔
```

### Récapitulatif des Tests

| Test | Résultat |
|------|---------|
| ✓ Connexion au Dashboard Horizon | `admin` authentifié avec succès |
| ✓ Création de VM1 et VM2 | Status : **Active**, Running |
| ✓ Assignation d'IPs flottantes | `172.24.4.168` (VM1) · `172.24.4.122` (VM2) |
| ✓ Connexion SSH depuis Windows | Authentification par clé RSA réussie |
| ✓ Ping Internet (8.8.8.8) | 0% perte — routage externe fonctionnel |
| ✓ Ping inter-VMs (10.0.0.0/24) | Communication interne OK |

---

## Gestion des Ressources

### Flavors (Gabarits CPU/RAM/Disque)

| Flavor | vCPUs | RAM | Disque | Usage typique |
|--------|-------|-----|--------|---------------|
| **m1.tiny** | 1 | 512 Mo | 1 Go | Tests légers |
| **m1.small** | 1 | 2 048 Mo | 20 Go | Applications web |
| **m1.medium** | 2 | 4 096 Mo | 40 Go | Services intermédiaires |

### Quotas par Projet

| Ressource | Limite |
|-----------|--------|
| Instances | 10 maximum |
| vCPUs | 20 cores |
| RAM | 50 Go |
| Volumes | 10 |
| Stockage volumes | 1 000 Go |
| IPs flottantes | 50 |

---

## Captures d'écran — Installation

> Captures réalisées lors de l'installation de DevStack sur Ubuntu 24.04

<table>
<tr>
<td align="center" width="50%">

![Mise à jour système](Installation%20Screenshot's/Screenshot%202026-02-18%20164400.png)

</td>
<td align="center" width="50%">

![Installation dépendances](Installation%20Screenshot's/Screenshot%202026-02-18%20164401.png)

</td>
</tr>
<tr>
<td align="center" width="50%">

![Clonage DevStack 20%](Installation%20Screenshot's/Screenshot%202026-02-18%20164402.png)

</td>
<td align="center" width="50%">

![Clonage DevStack terminé](Installation%20Screenshot's/Screenshot%202026-02-18%20164403.png)

</td>
</tr>
<tr>
<td align="center" width="50%">

![local.conf](Installation%20Screenshot's/Screenshot%202026-02-18%20164404.png)

</td>
<td align="center" width="50%">

![Lancement stack.sh](Installation%20Screenshot's/Screenshot%202026-02-18%20164405.png)

</td>
</tr>
</table>

---

## Captures d'écran — OpenStack Horizon

> Captures réalisées lors de la configuration et utilisation de l'infrastructure cloud

<table>
<tr>
<td align="center" width="50%">

![Dashboard Horizon](OpenStack%20Screenshot's/Screenshot%202026-05-18%20154853.png)

</td>
<td align="center" width="50%">

![Réseau public](OpenStack%20Screenshot's/Screenshot%202026-05-18%20155239.png)

</td>
</tr>
</table>

<table>
<tr>
<td align="center" width="33%">

![Sous-réseau public](OpenStack%20Screenshot's/Screenshot%202026-05-18%20155741.png)

</td>
<td align="center" width="33%">

![Détails sous-réseau public](OpenStack%20Screenshot's/Screenshot%202026-05-18%20155822.png)

</td>
<td align="center" width="33%">

![Réseau privé](OpenStack%20Screenshot's/Screenshot%202026-05-18%20155923.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![Config sous-réseau privé](OpenStack%20Screenshot's/Screenshot%202026-05-18%20160335.png)

</td>
<td align="center" width="33%">

![Détails sous-réseau privé](OpenStack%20Screenshot's/Screenshot%202026-05-18%20160542.png)

</td>
<td align="center" width="33%">

![Routeur R1](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161005.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![Interface routeur](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161042.png)

</td>
<td align="center" width="33%">

![Groupe de sécurité](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161115.png)

</td>
<td align="center" width="33%">

![Règles SSH](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161425.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![Règles ICMP](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161729.png)

</td>
<td align="center" width="33%">

![Règles configurées](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161823.png)

</td>
<td align="center" width="33%">

![Image Glance](OpenStack%20Screenshot's/Screenshot%202026-05-18%20161919.png)

</td>
</tr>
</table>

<table>
<tr>
<td align="center" width="33%">

![Paires de clés SSH](OpenStack%20Screenshot's/Screenshot%202026-05-18%20162047.png)

</td>
<td align="center" width="33%">

![Lancement VM1](OpenStack%20Screenshot's/Screenshot%202026-05-18%20162154.png)

</td>
<td align="center" width="33%">

![Source image cirros](OpenStack%20Screenshot's/Screenshot%202026-05-18%20162213.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![Sélection flavor](OpenStack%20Screenshot's/Screenshot%202026-05-18%20162253.png)

</td>
<td align="center" width="33%">

![Configuration réseau VM](OpenStack%20Screenshot's/Screenshot%202026-05-18%20162313.png)

</td>
<td align="center" width="33%">

![Groupe sécurité VM](OpenStack%20Screenshot's/Screenshot%202026-05-18%20162502.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![VM1 active](OpenStack%20Screenshot's/Screenshot%202026-05-18%20164219.png)

</td>
<td align="center" width="33%">

![Interfaces routeur R1](OpenStack%20Screenshot's/Screenshot%202026-05-18%20164256.png)

</td>
<td align="center" width="33%">

![IPs flottantes](OpenStack%20Screenshot's/Screenshot%202026-05-18%20164433.png)

</td>
</tr>
</table>

<table>
<tr>
<td align="center" width="33%">

![IP flottante associée](OpenStack%20Screenshot's/Screenshot%202026-05-18%20165826.png)

</td>
<td align="center" width="33%">

![Gestion IPs flottantes](OpenStack%20Screenshot's/Screenshot%202026-05-18%20165937.png)

</td>
<td align="center" width="33%">

![Connexion SSH](OpenStack%20Screenshot's/Screenshot%202026-05-18%20170039.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![Test ping](OpenStack%20Screenshot's/Screenshot%202026-05-18%20170117.png)

</td>
<td align="center" width="33%">

![Deux instances actives](OpenStack%20Screenshot's/Screenshot%202026-05-18%20171446.png)

</td>
<td align="center" width="33%">

![Tests ping inter-VM](OpenStack%20Screenshot's/Screenshot%202026-05-18%20171903.png)

</td>
</tr>
<tr>
<td align="center" width="33%">

![Topologie réseau](OpenStack%20Screenshot's/Screenshot%202026-05-18%20172321.png)

</td>
<td align="center" width="33%">

![Flavors](OpenStack%20Screenshot's/Screenshot%202026-05-18%20172450.png)

</td>
<td></td>
</tr>
</table>

---

## Conclusion et Perspectives

### Bilan du Projet

Ce projet a permis de déployer avec succès une infrastructure cloud privée complète basée sur OpenStack et DevStack. L'ensemble des objectifs fixés ont été atteints :

- ✓ Installation d'OpenStack (DevStack) sur **Ubuntu 24.04** dans une VM VirtualBox
- ✓ Déploiement des **6 services** : Keystone, Nova, Neutron, Glance, Cinder, Horizon
- ✓ Création et gestion de machines virtuelles (**VM1, VM2**) via Horizon et CLI
- ✓ Configuration du **réseau virtuel** : réseau public, réseau privé (10.0.0.0/24), routeur R1
- ✓ Attribution d'**IPs flottantes** et tests de connectivité SSH et ping validés
- ✓ Gestion des ressources : **quotas, flavors, groupes de sécurité**

### Conclusion

OpenStack est une solution cloud privée **robuste et flexible**, largement adoptée dans l'industrie. Ce projet a démontré la faisabilité de déployer une plateforme IaaS fonctionnelle avec DevStack dans un environnement de laboratoire, tout en couvrant les aspects essentiels d'un cloud : **compute, réseau, stockage et interface de gestion**. Les connaissances acquises constituent une base solide pour aborder des déploiements cloud en production.

---

<div align="center">

**ENSA Berrechid — Génie Informatique S8 — 2025/2026**

Made by [EL OUAFIR Mohamed](https://github.com) & [ENNAQUI Zakaria](https://github.com/zakariaennaqui)

[![GitHub](https://img.shields.io/badge/GitHub-OpenStack--CloudPrive-181717?style=flat-square&logo=github)](https://github.com/zakariaennaqui/OpenStack-CloudPrive)

</div>
