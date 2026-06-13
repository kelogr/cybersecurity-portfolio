---
layout: page
title: Lab Writeups
permalink: /tabs/lab-writeups/
icon: fas fa-flask
order: 2
---

> 🎯 **Objetivo de los laboratorios:** Desarrollar, auditar y documentar de forma meticulosa escenarios prácticos de ciberseguridad, abarcando desde la identificación de amenazas y análisis de vectores de ataque hasta la implementación de contramedidas, monitoreo defensivo y hardening de entornos controlados bajo marcos de referencia del sector (MITRE ATT&CK, NIST).

En esta sección recopilo los *writeups* técnicos de máquinas CTF, entornos de infraestructura propia y desafíos de plataformas defensivas.

## Laboratorios y Auditorías Realizadas

| 🖥️ Escenario / Máquina | 🌐 Entorno / Plataforma | 📅 Fecha de Resolución |
| :--- | :--- | :--- |
{% for post in site.tags.writeup %}| [**{{ post.title }}**]({{ post.url | relative_url }}) | *Blue Team Lab* | {{ post.date | date: "%d/%m/%Y" }} |
{% else %}| *Próximamente más writeups indexados de forma automática...* | | |
{% endfor %}

---

### 🛠️ Core Stack Utilizado en Laboratorios
* **Monitoreo & Logs:** Snort IDS, Zeek, Splunk Enterprise, ELK Stack.
* **Firewalling & Hardening:** Netfilter/IPTables, FTPS, SSH Bastion Hosts.
* **Auditoría:** Nmap, Wireshark, Tshark, LinPeas.