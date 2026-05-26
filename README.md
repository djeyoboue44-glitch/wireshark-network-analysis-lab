# wireshark-network-analysis-lab

# 🔍 Analyse de Trafic Réseau avec Wireshark — Détection d'un Port Scan

> Lab pratique de cybersécurité : capture de trafic réseau, filtrage avancé et détection d'un scan de ports Nmap dans un environnement virtualisé.

---

## 🧪 Environnement du Lab

| Élément | Détail |
|--------|--------|
| **Hyperviseur** | VMware Workstation |
| **Machine attaquante** | Kali Linux — `192.168.1.49` |
| **Machine cible** | Ubuntu (djeyoboue-VMware) — `192.168.1.70` |
| **Outil de capture** | Wireshark 4.6.4 |
| **Interface capturée** | `ens33` |
| **Outil d'attaque** | Nmap 7.99 |

---

## 🎯 Objectifs

- Capturer du trafic réseau en temps réel sur une interface VMware
- Appliquer des filtres d'affichage ciblés pour isoler des comportements suspects
- Détecter et analyser un port scan TCP SYN lancé depuis Kali Linux
- Identifier les indicateurs de compromission (IoC) visibles dans les paquets
- Documenter les findings comme un analyste SOC

---

## 📸 Captures d'écran & Analyse

### 1. Interface Wireshark — Sélection de l'interface

![Interface Wireshark](screenshots/01_wireshark_interface.png)

Lancement de Wireshark 4.6.4 sur Ubuntu. L'interface `ens33` est sélectionnée pour la capture — c'est l'interface réseau de la VM connectée au réseau local virtuel.

---

### 2. Capture en direct sur ens33

![Capture live](screenshots/02_capture_live.png)

Trafic réseau en cours de capture. On observe :
- Trafic **TLSv1.2** (paquets chiffrés vers l'extérieur)
- Requêtes **DNS** vers `192.168.1.1` (résolution de noms)
- Requêtes **ARP** de `SamsungElect_71:cc:...` cherchant `192.168.1.1`
- Trafic **NTP** (synchronisation de temps)
- Requêtes DNS vers `telegraf` et `telegraf.localdomain` (stack de monitoring)

---

### 3. Filtre : Paquets SYN uniquement

**Filtre utilisé :**
```
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

![Filtre SYN](screenshots/03_filter_syn.png)

Ce filtre isole les **paquets TCP SYN purs** (début de connexion, sans ACK). On observe 3 paquets SYN provenant de sources différentes — comportement normal à faible volume. Ce filtre est fondamental pour détecter un port scan SYN stealth.

---

### 4. Filtre : Trafic source 192.168.1.49 (Kali)

**Filtre utilisé :**
```
ip.src == 192.168.1.49
```

![Filtre IP source Kali](screenshots/04_filter_ip_src.png)

Isolation du trafic émis par la machine Kali Linux. On voit :
- Des requêtes **DNS** initiales vers `telegraf` (comportement normal)
- À partir du timestamp **16.80s**, une explosion de paquets **TCP SYN** vers `192.168.1.70` sur des ports variés (143, 5900, 587, 110, 139, 135...) — **signe clair d'un port scan**

---

### 5. Filtre : Paquets RST depuis Kali

**Filtre utilisé :**
```
ip.src == 192.168.1.49 and tcp.flags.reset == 1
```

![Filtre RST](screenshots/05_filter_rst.png)

Les paquets **TCP RST** indiquent que la cible a refusé les connexions (ports fermés). Seulement 2 RST visibles — la machine Ubuntu a rejeté les tentatives de connexion sur les ports scannés, ce qui confirme qu'ils étaient fermés ou filtrés.

---

### 6. Port Scan SYN — Vue complète

**Filtre utilisé :**
```
ip.src == 192.168.1.49 and tcp.flags.syn == 1 and tcp.flags.ack == 0
```

![Port scan SYN](screenshots/06_port_scan_syn.png)

**Détection confirmée du port scan.** En moins de **2 millisecondes** (timestamps 16.802611 → 16.803239), Kali envoie des SYN vers plus de 20 ports différents sur `192.168.1.70` depuis le même port source `35914`. Liste partielle des ports ciblés : 53, 80, 110, 113, 135, 139, 143, 256, 443, 445, 554, 587, 993, 995, 1025, 1720, 1723, 3306, 5900, 8080, 8888...

> ⚠️ **Indicateur de compromission (IoC)** : 1 000 SYN envoyés en moins de 2ms depuis une seule IP source vers une seule IP cible = comportement caractéristique d'un **scan SYN stealth Nmap**.

---

### 7. Conversations TCP — Statistics

![Conversations TCP](screenshots/07_conversations.png)

La vue **Statistics > Conversations** confirme l'ampleur du scan :
- **1 000 flux TCP distincts** entre `192.168.1.49:35914` et `192.168.1.70` sur des ports différents
- Chaque conversation = 1 paquet de 60 bytes (SYN uniquement, sans réponse)
- Le tab **TCP · 1000** est explicite : 1 000 tentatives de connexion enregistrées

---

### 8. Scan Nmap côté attaquant (Kali)

![Nmap Kali](screenshots/08_nmap_scan.png)

Côté Kali Linux, les commandes Nmap exécutées contre `192.168.1.70` :

```bash
sudo nmap -sS 192.168.1.70   # SYN Stealth Scan → port 22/tcp open (SSH)
sudo nmap -sV 192.168.1.70   # Version scan → OpenSSH 10.2p1 Ubuntu 2ubuntu3.2
sudo nmap -A 192.168.1.70    # Aggressive scan → OS fingerprint Linux
```

**Résultat :** Seul le port **22/tcp (SSH)** est ouvert sur la cible. L'OS fingerprint correspond à un noyau Linux.

---

### 9. Endpoints TCP — Vue globale

![Endpoints](screenshots/09_endpoints.png)

La vue **Statistics > Endpoints** confirme :
- `192.168.1.49` : **1 000 paquets émis / 60 kB** (100% du trafic sortant = attaquant)
- `192.168.1.70` : **1 paquet reçu par port** (0 bytes envoyés = cible silencieuse)
- 1 001 endpoints TCP distincts recensés

---

## 🔎 Filtres Wireshark Utilisés

| Filtre | Objectif |
|--------|----------|
| `tcp.flags.syn == 1 and tcp.flags.ack == 0` | Isoler les SYN purs (début de connexion) |
| `ip.src == 192.168.1.49` | Trafic émis par la machine Kali |
| `ip.src == 192.168.1.49 and tcp.flags.reset == 1` | Ports refusés par la cible |
| `ip.src == 192.168.1.49 and tcp.flags.syn == 1 and tcp.flags.ack == 0` | Port scan SYN complet |

---

## 🚨 Findings & Indicateurs de Compromission (IoC)

| IoC | Valeur |
|-----|--------|
| **IP attaquante** | `192.168.1.49` (Kali Linux) |
| **IP cible** | `192.168.1.70` (Ubuntu) |
| **Port source fixe** | `35914` |
| **Nombre de SYN envoyés** | ~1 000 |
| **Durée du scan** | < 2 ms |
| **Port ouvert découvert** | `22/tcp` (SSH — OpenSSH 10.2p1) |
| **Type de scan** | TCP SYN Stealth (`nmap -sS`) |
| **Technique de détection** | Filtrage Wireshark + Statistics > Conversations/Endpoints |

---

## 🛡️ Compétences Démontrées

- Capture de trafic réseau sur interface VMware avec Wireshark
- Rédaction et application de filtres d'affichage BPF/Wireshark
- Identification d'un port scan SYN dans une capture PCAP
- Corrélation attaque/défense (Nmap côté attaquant ↔ Wireshark côté cible)
- Analyse de conversations et d'endpoints TCP
- Documentation d'indicateurs de compromission (IoC)

---

## 🖥️ Prérequis

- VMware Workstation / Fusion
- Kali Linux (VM)
- Ubuntu Server/Desktop (VM)
- Wireshark 4.x
- Nmap

---

## 👤 Auteur

**Nicolas Yoboué** — Étudiant Mastère Expert IT Cybersécurité @ École IRIS Paris  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/TON_PROFIL)
