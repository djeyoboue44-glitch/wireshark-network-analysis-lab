# wireshark-network-analysis-lab

from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import cm
from reportlab.lib import colors
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, HRFlowable, Image
from reportlab.platypus import PageBreak
from reportlab.lib.enums import TA_LEFT, TA_CENTER, TA_JUSTIFY
import os

OUTPUT = "/home/claude/projet-wireshark/rapport_detection_scan.pdf"
SCREENSHOTS = "/home/claude/projet-wireshark/screenshots"

doc = SimpleDocTemplate(
    OUTPUT,
    pagesize=A4,
    rightMargin=2*cm, leftMargin=2*cm,
    topMargin=2*cm, bottomMargin=2*cm
)

styles = getSampleStyleSheet()

# Custom styles
title_style = ParagraphStyle('CustomTitle',
    parent=styles['Title'],
    fontSize=22, textColor=colors.HexColor('#1a1a2e'),
    spaceAfter=6, alignment=TA_CENTER)

subtitle_style = ParagraphStyle('Subtitle',
    parent=styles['Normal'],
    fontSize=11, textColor=colors.HexColor('#4a4a6a'),
    spaceAfter=4, alignment=TA_CENTER)

h1_style = ParagraphStyle('H1',
    parent=styles['Heading1'],
    fontSize=14, textColor=colors.HexColor('#1a1a2e'),
    spaceBefore=16, spaceAfter=6,
    borderPad=4)

h2_style = ParagraphStyle('H2',
    parent=styles['Heading2'],
    fontSize=12, textColor=colors.HexColor('#2d5986'),
    spaceBefore=12, spaceAfter=4)

body_style = ParagraphStyle('Body',
    parent=styles['Normal'],
    fontSize=10, leading=15,
    textColor=colors.HexColor('#2c2c2c'),
    spaceAfter=6, alignment=TA_JUSTIFY)

code_style = ParagraphStyle('Code',
    parent=styles['Normal'],
    fontSize=9, fontName='Courier',
    textColor=colors.HexColor('#1e1e3f'),
    backColor=colors.HexColor('#f0f0f8'),
    borderPad=6, spaceAfter=8, spaceBefore=4,
    leftIndent=12)

caption_style = ParagraphStyle('Caption',
    parent=styles['Normal'],
    fontSize=9, textColor=colors.HexColor('#666666'),
    alignment=TA_CENTER, spaceAfter=10, spaceBefore=2,
    fontName='Helvetica-Oblique')

def add_img(filename, caption, width=16*cm):
    path = os.path.join(SCREENSHOTS, filename)
    if os.path.exists(path):
        elems = []
        try:
            img = Image(path, width=width, height=width*0.55)
            elems.append(img)
        except:
            elems.append(Paragraph(f"[Image: {caption}]", caption_style))
        elems.append(Paragraph(f"Figure : {caption}", caption_style))
        return elems
    return [Paragraph(f"[Image non trouvée: {filename}]", caption_style)]

story = []

# ─── PAGE DE GARDE ───
story.append(Spacer(1, 2*cm))
story.append(Paragraph("RAPPORT TECHNIQUE", subtitle_style))
story.append(Spacer(1, 0.3*cm))
story.append(Paragraph("Détection de Scan Réseau", title_style))
story.append(Paragraph("Wireshark & Nmap — Lab Cybersécurité", subtitle_style))
story.append(Spacer(1, 0.5*cm))
story.append(HRFlowable(width="100%", thickness=2, color=colors.HexColor('#2d5986')))
story.append(Spacer(1, 0.5*cm))

info_data = [
    ["Auteur", "Yoboue Dje"],
    ["Formation", "Mastère Expert IT — Cybersécurité, Réseaux & Systèmes"],
    ["École", "École IRIS Paris"],
    ["Date", "25 Mai 2026"],
    ["Environnement", "VMware Workstation — Kali Linux + Ubuntu"],
    ["Technique MITRE ATT&CK", "T1046 — Network Service Discovery"],
]
info_table = Table(info_data, colWidths=[5*cm, 11*cm])
info_table.setStyle(TableStyle([
    ('BACKGROUND', (0,0), (0,-1), colors.HexColor('#2d5986')),
    ('TEXTCOLOR', (0,0), (0,-1), colors.white),
    ('FONTNAME', (0,0), (0,-1), 'Helvetica-Bold'),
    ('FONTSIZE', (0,0), (-1,-1), 10),
    ('BACKGROUND', (1,0), (1,-1), colors.HexColor('#f5f8ff')),
    ('ROWBACKGROUNDS', (1,0), (1,-1), [colors.HexColor('#f5f8ff'), colors.HexColor('#eef2ff')]),
    ('GRID', (0,0), (-1,-1), 0.5, colors.HexColor('#ccccdd')),
    ('VALIGN', (0,0), (-1,-1), 'MIDDLE'),
    ('TOPPADDING', (0,0), (-1,-1), 7),
    ('BOTTOMPADDING', (0,0), (-1,-1), 7),
    ('LEFTPADDING', (0,0), (-1,-1), 10),
]))
story.append(info_table)
story.append(Spacer(1, 1*cm))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#ccccdd')))
story.append(Spacer(1, 0.5*cm))

story.append(Paragraph(
    "Ce rapport documente la mise en place d'un lab de cybersécurité sous VMware permettant de simuler "
    "une attaque de reconnaissance réseau (scan Nmap) et de la détecter en temps réel via Wireshark. "
    "Il illustre des compétences Blue Team : capture de trafic, analyse de paquets TCP, identification "
    "de comportements suspects et documentation d'incidents de sécurité.",
    body_style))

story.append(PageBreak())

# ─── 1. ARCHITECTURE ───
story.append(Paragraph("1. Architecture du Lab", h1_style))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#2d5986'), spaceAfter=8))

story.append(Paragraph(
    "Le lab est composé de deux machines virtuelles déployées sous VMware Workstation, "
    "connectées sur le même réseau NAT (192.168.1.0/24) :", body_style))

arch_data = [
    ["Rôle", "OS", "IP", "Outil principal"],
    ["Attaquant", "Kali Linux 2024.3", "192.168.1.49", "Nmap 7.99"],
    ["Cible / Analyste", "Ubuntu 24.04 LTS", "192.168.1.70", "Wireshark 4.6.4"],
]
arch_table = Table(arch_data, colWidths=[4*cm, 4.5*cm, 3.5*cm, 4*cm])
arch_table.setStyle(TableStyle([
    ('BACKGROUND', (0,0), (-1,0), colors.HexColor('#1a1a2e')),
    ('TEXTCOLOR', (0,0), (-1,0), colors.white),
    ('FONTNAME', (0,0), (-1,0), 'Helvetica-Bold'),
    ('FONTSIZE', (0,0), (-1,-1), 10),
    ('ROWBACKGROUNDS', (0,1), (-1,-1), [colors.HexColor('#f5f8ff'), colors.HexColor('#eef2ff')]),
    ('GRID', (0,0), (-1,-1), 0.5, colors.HexColor('#aaaacc')),
    ('VALIGN', (0,0), (-1,-1), 'MIDDLE'),
    ('TOPPADDING', (0,0), (-1,-1), 8),
    ('BOTTOMPADDING', (0,0), (-1,-1), 8),
    ('LEFTPADDING', (0,0), (-1,-1), 10),
    ('ALIGN', (2,0), (2,-1), 'CENTER'),
]))
story.append(arch_table)
story.append(Spacer(1, 0.5*cm))

story.extend(add_img("01_wireshark_interface_ens33.png",
    "Interface Wireshark au démarrage — sélection de l'interface ens33 sur Ubuntu"))

# ─── 2. SCANS NMAP ───
story.append(Paragraph("2. Scans Nmap depuis Kali Linux", h1_style))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#2d5986'), spaceAfter=8))

story.append(Paragraph(
    "Trois types de scans ont été exécutés successivement depuis la machine Kali Linux "
    "(192.168.1.49) vers la cible Ubuntu (192.168.1.70) :", body_style))

scans = [
    ("SYN Scan (-sS)", "sudo nmap -sS 192.168.1.70",
     "Envoie uniquement des paquets TCP SYN. Furtif car n'établit pas de connexion complète. "
     "Méthode la plus utilisée en reconnaissance."),
    ("Service Detection (-sV)", "sudo nmap -sV 192.168.1.70",
     "Identifie les services et leurs versions sur les ports ouverts. "
     "Résultat : OpenSSH 10.2p1 Ubuntu sur le port 22."),
    ("Aggressive Scan (-A)", "sudo nmap -A 192.168.1.70",
     "Combine détection OS, versions, scripts NSE et traceroute. "
     "Génère un TCP/IP fingerprint pour identifier l'OS."),
]

for title, cmd, desc in scans:
    story.append(Paragraph(title, h2_style))
    story.append(Paragraph(cmd, code_style))
    story.append(Paragraph(desc, body_style))

story.append(Paragraph("Résultat consolidé des scans :", h2_style))
story.append(Paragraph(
    "PORT    STATE  SERVICE  VERSION<br/>"
    "22/tcp  open   ssh      OpenSSH 10.2p1 Ubuntu 2ubuntu3.2 (Ubuntu Linux; protocol 2.0)<br/>"
    "999 ports closed (reset)<br/>"
    "MAC Address: 00:0C:29:3E:39:64 (VMware)<br/>"
    "Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel",
    code_style))

story.extend(add_img("10_nmap_commandes_kali.png",
    "Résultats des 3 scans Nmap (-sS, -sV, -A) exécutés depuis Kali Linux"))

story.append(PageBreak())

# ─── 3. ANALYSE WIRESHARK ───
story.append(Paragraph("3. Analyse Wireshark — Détection du scan", h1_style))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#2d5986'), spaceAfter=8))

story.append(Paragraph("3.1 Trafic réseau normal (avant le scan)", h2_style))
story.append(Paragraph(
    "Avant le lancement des scans, Wireshark capture un trafic réseau normal : "
    "requêtes DNS, paquets ARP, échanges TLS vers des services externes. "
    "Ce trafic sert de baseline pour identifier les anomalies.", body_style))
story.extend(add_img("02_capture_trafic_normal.png",
    "Trafic réseau normal capturé sur ens33 — DNS, ARP, TLS (baseline)"))

story.append(Paragraph("3.2 Isolation du trafic Kali → Ubuntu", h2_style))
story.append(Paragraph("Filtre appliqué :", body_style))
story.append(Paragraph("ip.src == 192.168.1.49", code_style))
story.append(Paragraph(
    "Ce filtre isole tout le trafic provenant de la machine attaquante. "
    "On observe une transition nette : des requêtes DNS normales au début, "
    "puis une explosion de paquets TCP [SYN] vers des dizaines de ports différents — "
    "signature caractéristique d'un scan automatisé.", body_style))
story.extend(add_img("05_trafic_kali_vers_ubuntu.png",
    "Trafic de Kali (192.168.1.49) vers Ubuntu — SYN vers ports 143, 5900, 587, 110, 139..."))

story.append(PageBreak())

story.append(Paragraph("3.3 Signature du SYN Scan — 1000 paquets", h2_style))
story.append(Paragraph("Filtre appliqué :", body_style))
story.append(Paragraph("ip.src == 192.168.1.49 and tcp.flags.syn == 1 and tcp.flags.ack == 0", code_style))
story.append(Paragraph(
    "Ce filtre révèle la signature caractéristique du SYN Scan : 1000 paquets TCP SYN "
    "envoyés depuis le port source fixe 35914 de Kali, vers 1000 ports différents d'Ubuntu, "
    "en moins d'une seconde. Chaque paquet pèse exactement 60 bytes, confirmant l'absence "
    "de handshake TCP complet (technique furtive).", body_style))
story.extend(add_img("07_SYN_scan_1000_ports.png",
    "SYN Scan : 1000 paquets TCP [SYN] de 192.168.1.49 → 192.168.1.70 en < 1 seconde"))

story.append(Paragraph("3.4 Détection des ports fermés — Réponses RST", h2_style))
story.append(Paragraph("Filtre appliqué :", body_style))
story.append(Paragraph("ip.src == 192.168.1.49 and tcp.flags.reset == 1", code_style))
story.append(Paragraph(
    "Ubuntu répond avec des paquets TCP [RST] ou [RST, ACK] sur les ports fermés. "
    "Ces réponses confirment que les paquets SYN ont bien atteint la cible et "
    "permettent à Nmap de cartographier les ports ouverts vs fermés :", body_style))

rst_data = [
    ["Paquet", "Source", "Destination", "Flags", "Interprétation"],
    ["#4070", "192.168.1.49:35914", "192.168.1.70:22", "[RST]", "Port 22 — réinitialisé"],
    ["#6051", "192.168.1.49:35914", "192.168.1.70:9200", "[RST, ACK]", "Port 9200 — fermé"],
]
rst_table = Table(rst_data, colWidths=[1.5*cm, 3.5*cm, 3.5*cm, 2*cm, 5.5*cm])
rst_table.setStyle(TableStyle([
    ('BACKGROUND', (0,0), (-1,0), colors.HexColor('#c0392b')),
    ('TEXTCOLOR', (0,0), (-1,0), colors.white),
    ('FONTNAME', (0,0), (-1,0), 'Helvetica-Bold'),
    ('FONTSIZE', (0,0), (-1,-1), 9),
    ('ROWBACKGROUNDS', (0,1), (-1,-1), [colors.HexColor('#fff5f5'), colors.HexColor('#ffe8e8')]),
    ('GRID', (0,0), (-1,-1), 0.5, colors.HexColor('#ddaaaa')),
    ('VALIGN', (0,0), (-1,-1), 'MIDDLE'),
    ('TOPPADDING', (0,0), (-1,-1), 6),
    ('BOTTOMPADDING', (0,0), (-1,-1), 6),
    ('LEFTPADDING', (0,0), (-1,-1), 6),
]))
story.append(rst_table)
story.append(Spacer(1, 0.3*cm))
story.extend(add_img("06_RST_ports_fermes.png",
    "Réponses RST d'Ubuntu : confirmation des ports fermés suite au scan Kali"))

story.append(PageBreak())

story.append(Paragraph("3.5 Vue Conversations TCP — 1000 connexions", h2_style))
story.append(Paragraph(
    "La vue Statistics → Conversations → TCP révèle 1000 entrées, toutes issues de "
    "192.168.1.49:35914 vers 192.168.1.70. Chaque ligne correspond à une tentative "
    "de connexion sur un port différent. La taille uniforme de 60 bytes par échange "
    "confirme qu'aucune connexion TCP complète n'a été établie.", body_style))
story.extend(add_img("08_conversations_TCP_1000.png",
    "Conversations TCP · 1000 entrées — toutes de Kali (35914) vers Ubuntu"))

story.append(Paragraph("3.6 Vue Endpoints TCP — 1001 ports touchés", h2_style))
story.append(Paragraph(
    "La vue Statistics → Endpoints → TCP confirme que 192.168.1.49 a émis 1000 paquets "
    "TCP (60 kB au total) vers 1001 endpoints distincts sur Ubuntu. Chaque port reçoit "
    "exactement 1 paquet de 60 bytes. C'est la preuve irréfutable d'un scan automatisé "
    "couvrant l'ensemble des 1000 ports top Nmap.", body_style))
story.extend(add_img("09_endpoints_TCP_1001.png",
    "Endpoints TCP · 1001 ports touchés sur 192.168.1.70 — 60 kB émis par Kali"))

story.append(PageBreak())

# ─── 4. RÉSUMÉ ───
story.append(Paragraph("4. Résumé de la détection", h1_style))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#2d5986'), spaceAfter=8))

summary_data = [
    ["Indicateur", "Valeur observée", "Interprétation"],
    ["IP source suspecte", "192.168.1.49", "Machine attaquante (Kali Linux)"],
    ["Paquets SYN envoyés", "1 000", "Scan de 1000 ports TCP"],
    ["Durée du scan", "< 400ms", "Scan automatisé (Nmap)"],
    ["Taille des paquets", "60 bytes", "SYN sans handshake complet"],
    ["Port source constant", "35914", "Source unique = scanner"],
    ["Port ouvert détecté", "22/tcp (SSH)", "Service OpenSSH 10.2p1 exposé"],
    ["Réponses RST", "2 confirmées", "Ports fermés détectés"],
    ["Technique MITRE", "T1046", "Network Service Discovery"],
]
sum_table = Table(summary_data, colWidths=[5*cm, 4*cm, 7*cm])
sum_table.setStyle(TableStyle([
    ('BACKGROUND', (0,0), (-1,0), colors.HexColor('#1a1a2e')),
    ('TEXTCOLOR', (0,0), (-1,0), colors.white),
    ('FONTNAME', (0,0), (-1,0), 'Helvetica-Bold'),
    ('FONTSIZE', (0,0), (-1,-1), 9.5),
    ('ROWBACKGROUNDS', (0,1), (-1,-1), [colors.HexColor('#f0f5ff'), colors.HexColor('#e8eeff')]),
    ('GRID', (0,0), (-1,-1), 0.5, colors.HexColor('#aaaacc')),
    ('VALIGN', (0,0), (-1,-1), 'MIDDLE'),
    ('TOPPADDING', (0,0), (-1,-1), 7),
    ('BOTTOMPADDING', (0,0), (-1,-1), 7),
    ('LEFTPADDING', (0,0), (-1,-1), 8),
    ('BACKGROUND', (1,5), (1,5), colors.HexColor('#d4edda')),
    ('TEXTCOLOR', (1,5), (1,5), colors.HexColor('#155724')),
    ('FONTNAME', (1,5), (1,5), 'Helvetica-Bold'),
]))
story.append(sum_table)

story.append(Spacer(1, 1*cm))

# ─── 5. IOC ───
story.append(Paragraph("5. Indicateurs de Compromission (IOC)", h1_style))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#2d5986'), spaceAfter=8))

story.append(Paragraph(
    "Type     : Network Reconnaissance Scan<br/>"
    "Source   : 192.168.1.49 (Kali Linux / VMware MAC: 00:0C:29:1D:AE:B2)<br/>"
    "Cible    : 192.168.1.70 (Ubuntu ens33 / VMware MAC: 00:0C:29:3E:39:64)<br/>"
    "Date     : 25/05/2026 — 18:17 UTC<br/>"
    "Outil    : Nmap 7.99 (SYN Scan -sS / -sV / -A)<br/>"
    "Volume   : 1000 paquets SYN / 60 kB<br/>"
    "Durée    : ~400ms<br/>"
    "MITRE    : T1046 — Network Service Discovery",
    code_style))

# ─── 6. COMPÉTENCES ───
story.append(Paragraph("6. Compétences démontrées", h1_style))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#2d5986'), spaceAfter=8))

skills_data = [
    ["Domaine", "Compétence"],
    ["Blue Team", "Détection de scan réseau en temps réel avec Wireshark"],
    ["Analyse réseau", "Filtres avancés : TCP flags, IP source, Statistics"],
    ["Protocoles TCP/IP", "Compréhension SYN/RST/ACK, handshake, comportement Nmap"],
    ["MITRE ATT&CK", "Identification technique T1046 (Network Service Discovery)"],
    ["Lab sécurité", "Déploiement d'un environnement attaquant/cible sous VMware"],
    ["Documentation", "Rédaction rapport d'incident structuré avec IOC"],
    ["Pentest (Red Team)", "Utilisation Nmap -sS/-sV/-A, interprétation des résultats"],
]
skills_table = Table(skills_data, colWidths=[4.5*cm, 11.5*cm])
skills_table.setStyle(TableStyle([
    ('BACKGROUND', (0,0), (-1,0), colors.HexColor('#2d5986')),
    ('TEXTCOLOR', (0,0), (-1,0), colors.white),
    ('FONTNAME', (0,0), (-1,0), 'Helvetica-Bold'),
    ('FONTSIZE', (0,0), (-1,-1), 10),
    ('ROWBACKGROUNDS', (0,1), (-1,-1), [colors.HexColor('#f5f8ff'), colors.HexColor('#eef2ff')]),
    ('GRID', (0,0), (-1,-1), 0.5, colors.HexColor('#aaaacc')),
    ('VALIGN', (0,0), (-1,-1), 'MIDDLE'),
    ('TOPPADDING', (0,0), (-1,-1), 7),
    ('BOTTOMPADDING', (0,0), (-1,-1), 7),
    ('LEFTPADDING', (0,0), (-1,-1), 8),
    ('BACKGROUND', (0,1), (0,-1), colors.HexColor('#e8f0fe')),
    ('FONTNAME', (0,1), (0,-1), 'Helvetica-Bold'),
    ('TEXTCOLOR', (0,1), (0,-1), colors.HexColor('#2d5986')),
]))
story.append(skills_table)

story.append(Spacer(1, 1*cm))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#ccccdd')))
story.append(Spacer(1, 0.3*cm))
story.append(Paragraph(
    "Yoboue Dje — Mastère Expert IT Cybersécurité — École IRIS Paris — Mai 2026<br/>"
    "linkedin.com/in/yoboue-dje | github.com/djeyoboue44-glitch",
    caption_style))

doc.build(story)
print("PDF généré avec succès !")
