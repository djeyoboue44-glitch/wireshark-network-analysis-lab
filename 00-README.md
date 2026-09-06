<div align="center">

# 🔍 Détection d'un Scan de Ports SYN  Wireshark

### Analyse réseau défensive  Capture • Filtrage • Corrélation avec l'attaquant

![Wireshark](https://img.shields.io/badge/Wireshark-4.6.4-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-attaquant-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Nmap](https://img.shields.io/badge/Nmap-7.99-00A8E1?style=for-the-badge&logo=nmap&logoColor=white)
![VMware](https://img.shields.io/badge/VMware-Workstation-607078?style=for-the-badge&logo=vmware&logoColor=white)
![TCP/IP](https://img.shields.io/badge/TCP%2FIP-analyse-red?style=for-the-badge)

</div>

---

## 📖 Description

Laboratoire d'analyse réseau réalisé sous **VMware**, mettant en scène une machine cible (**192.168.1.70**) et une machine **Kali Linux** attaquante (**192.168.1.49**). L'objectif est de capturer avec **Wireshark** un scan de ports **SYN (half-open scan)** lancé via **Nmap**, puis de reconstituer la chronologie de l'attaque à partir des seuls paquets capturés  avant de confirmer les résultats côté attaquant.

---

## 🎯 Objectifs

- 🖧 Capturer du trafic réseau en temps réel avec Wireshark
- 🕵️ Identifier la signature d'un scan de ports SYN dans une capture brute
- 🎯 Isoler le trafic malveillant à l'aide de filtres d'affichage ciblés
- 🔗 Corréler les paquets capturés avec la commande Nmap réellement exécutée côté attaquant
- 📊 Analyser les conversations et endpoints TCP générés par le scan

---

## 🧰 Environnement

| Machine | Rôle | Adresse IP |
|---|---|---|
| 🖥️ **Cible** | VM Linux analysée (interface `ens33`) | `192.168.1.70` |
| 🐉 **Kali Linux** | Machine d'attaque | `192.168.1.49` |

---

## 🖼️ Aperçu du projet

### 🏗️ Mise en place de la capture

**Interface Wireshark — sélection de `ens33`**
![Interface Wireshark](1-INTERFACE%20WIRESHARK.PNG)

**Capture en direct sur ens33**
![Capture en direct sur ens33](2-CAPTURE%20EN%20DIRECT%20SUR%20ENS33.PNG)

### 🕵️ Isolation du trafic SYN

**Filtre SYN uniquement**
![Filtre SYN uniquement](3-FILTRE%20SYN%20UNIQUEMENT.PNG)

**Filtre par IP source (Kali)**
![Filtre source 192.168.1.49](4-FILTRE%20SOURCE%20192.168.1.49.PNG)

**Filtre RST renvoyés par la cible**
![Filtre RST depuis Kali](5-FILTRE%20RST%20DEPUIS%20KALI.PNG)

### 🎯 Signature du scan de ports

**Vue complète du scan SYN**
![Port scan SYN - vue complète](6-%20PORT%20SCAN%20SYN%20-%20VUE%20COMPLETE.PNG)

**Analyse des conversations TCP**
![Conversation TCP](7-%20CONVERSATION%20TCP.PNG)

**Endpoints TCP**
![Endpoints TCP](9-%20ENDPOINTS%20TCP.PNG)

### 💻 Confirmation côté attaquant

**Scan Nmap exécuté depuis Kali**
![Scan Nmap côté attaquant](8-%20SCAN%20NMAP%20COTE%20ATTAQUANT.PNG)

```bash
sudo nmap -sS 192.168.1.70   # Scan SYN furtif
sudo nmap -sV 192.168.1.70   # Détection de version de service
sudo nmap -A 192.168.1.70    # Scan agressif (OS, version, scripts)
```

Résultat : seul le port **22/tcp (SSH  OpenSSH 10.2p1 Ubuntu)** est ouvert, tous les autres sont fermés (RST reçus).

---

## 📊 Indicateurs de compromission (IoC)

- 🚨 Volume anormal de paquets **SYN** vers de nombreux ports depuis une seule IP source
- 🔁 Port source **constant** (35914) réutilisé sur toutes les tentatives
- ❌ Réponses **RST** quasi systématiques (ports fermés), sauf sur le port 22
- ⏱️ Intervalle de temps très court entre chaque tentative (scan automatisé)

---

## ✅ Conclusion

La corrélation entre la capture Wireshark et les commandes Nmap exécutées côté attaquant confirme sans ambiguïté qu'un **scan de ports SYN (half-open scan)** a été mené contre la machine cible. Ce type de scan est furtif car il n'établit jamais de connexion TCP complète (pas de 3-way handshake terminé), ce qui le rend historiquement plus difficile à logguer côté application  d'où l'intérêt de l'analyse au niveau paquet avec Wireshark.

---

<div align="center">

*Projet réalisé dans le cadre du Mastère Expert IT — spécialisation Cybersécurité, Réseaux & Systèmes.*

</div>
