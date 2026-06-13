---
layout: post
title: Network Reconnaissance & Vulnerability Assessment with Nmap and Wireshark
date: 2026-06-13 17:00:00 +0200
categories: [academico, cybersecurity, network-security]
tags: [academic, writeup, nmap, wireshark, reconnaissanc]
---

# 🛡️ Network Reconnaissance & Vulnerability Assessment with Nmap and Wireshark

| Campo                          | Descripción                                                                                                 |
| :----------------------------- | :---------------------------------------------------------------------------------------------------------- |
| **Tecnologías / Herramientas** | Nmap, Wireshark, Kali Linux, Samba, ProFTPD, UnrealIRCd                                                                 |
| **Tags / Keywords**            | #Nmap #Wireshark #Reconnaissance #VulnerabilityAssessment #CVE #BlueTeam #NetworkSecurity                   |
| **Categorías**                 | Análisis de Amenazas / Seguridad en Redes                                                                   |
| **Autor**                      | Kevin López Granado                                                                                         |

---

## bWAPP Web Vulnerability Assessment & Defensive Analysis - [ACADEMIC / LAB - Beginner]

### 🎯 Objetivo

El objetivo de esta actividad es realizar un proceso técnico de **recopilación de información** sobre una máquina objetivo dentro de una red local, aplicando técnicas de descubrimiento de hosts, escaneo de puertos, fingerprinting de servicios y análisis de tráfico mediante Wireshark.

La práctica se centra en identificar servicios expuestos, analizar el comportamiento de distintos tipos de escaneo TCP y relacionar las versiones detectadas con posibles vulnerabilidades conocidas. Desde una perspectiva defensiva, esta actividad permite comprender cómo un atacante enumera una infraestructura y qué evidencias pueden observarse en red durante la fase de reconocimiento.

El entorno de laboratorio está compuesto por:
- **Máquina atacante:** Kali Linux.
- **Máquina objetivo:** `192.168.142.130`.
- **Red local:** `192.168.142.0/24`.
- **Herramientas principales:** Nmap y Wireshark.

---

### 🛠️ Prerrequisitos

Para reproducir esta actividad es necesario disponer de los siguientes elements:
* Una máquina atacante con **Kali Linux**.
* Una máquina objetivo vulnerable dentro de la misma red local.
* Conectividad IP entre atacante y víctima.
* Herramientas **Nmap** y **Wireshark** instaladas.
* Permisos de administración (`sudo`) para ejecutar escaneos avanzados.
* Conocimientos básicos de la pila TCP/IP, banderas de control (*Flags*), fingerprinting y vulnerabilidades CVE.

---

### 🚀 Ejecución

#### 1. Descubrimiento de hosts activos

En primer lugar, se realizó un escaneo de descubrimiento de hosts sobre el rango de red local con el objetivo de identificar dispositivos activos.

```bash
sudo nmap -sn 192.168.142.0/24
```

El resultado permitió identificar la máquina objetivo:
```text
Host is up
IP objetivo: 192.168.142.130
```

![Descubrimiento de hosts con Nmap](/assets/img/posts/net-reco-vuln-nmap/net-1.png)*Figura 1: Descubrimiento de hosts con Nmap*

Posteriormente, se verificó la conectividad entre la máquina atacante y la víctima mediante trazas ICMP:
```bash
ping -c 4 192.168.142.130
```

Salida esperada del comando:
```text
64 bytes from 192.168.142.130: icmp_seq=1 ttl=64 time=0.384 ms
64 bytes from 192.168.142.130: icmp_seq=2 ttl=64 time=0.284 ms
64 bytes from 192.168.142.130: icmp_seq=3 ttl=64 time=0.313 ms
64 bytes from 192.168.142.130: icmp_seq=4 ttl=64 time=0.399 ms

--- 192.168.142.130 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
```

![Comprobación de conectividad con PING](/assets/img/posts/net-reco-vuln-nmap/net-2.png)*Figura 2: Comprobación de conectividad con PING*

#### 2. Escaneo inicial de puertos con SYN Scan
Una vez identificada la máquina objetivo, se ejecutó un escaneo TCP SYN (*Half-Open*) para detectar puertos abiertos sin completar completamente el acuerdo de tres pasos (*Three-Way Handshake*).

```bash
sudo nmap -sS 192.168.142.130
```

Este tipo de escaneo permite identificar servicios expuestos de forma eficiente y es comúnmente utilizado durante las fases iniciales de reconocimiento.

Servicios relevantes detectados:
```text
21/tcp    open   ftp
22/tcp    open   ssh
445/tcp   open   microsoft-ds
3000/tcp  open   ppp
3306/tcp  open   mysql
6697/tcp  open   ircs-u
8080/tcp  open   http-proxy
8181/tcp  open   intermapper
```

![Escaneo de puertos (-sS)](/assets/img/posts/net-reco-vuln-nmap/net-3.png)*Figura 4: Escaneo de puertos (-sS)*

#### 3. Escaneo completo de puertos y detección de versiones
A continuación, se realizó un escaneo completo de todos los puertos TCP junto con detección de versiones para identificar con precisión los demonios activos.

```bash
sudo nmap -sV -p- 192.168.142.130
```

Resultados destacados del fingerprinting:
```text
21/tcp    open   ftp        ProFTPD 1.3.5
22/tcp    open   ssh        OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13
445/tcp   open   netbios-ssn Samba smbd 3.X - 4.X
3306/tcp  open   mysql      MySQL unauthorised
6697/tcp  open   irc        UnrealIRCd
8080/tcp  open   http       Jetty 1.7.x
```
La detección de versiones es una fase crítica, ya que permite correlacionar los servicios identificados con vulnerabilidades públicas conocidas.

![Escaneo completo (-p-))](/assets/img/posts/net-reco-vuln-nmap/net-4.png)*Figura 4: Escaneo completo (-p-)*

#### 4. Escaneos TCP NULL, FIN y XMAS
Se ejecutaron escaneos TCP avanzados para analizar cómo responde la máquina objetivo ante diferentes combinaciones anómalas de flags TCP.

##### 4.1. NULL Scan
El escaneo NULL envía paquetes TCP sin ningún flag activado.
```bash
sudo nmap -sN 192.168.142.130
```

Resultado observado:
```text
Not shown: 998 open|filtered tcp ports (no-response)
3000/tcp closed ppp
8181/tcp closed intermapper
```

![Escaneo NULL](/assets/img/posts/net-reco-vuln-nmap/net-5.png)*Figura 5: Escaneo NULL*

##### 4.2. FIN Scan
El escaneo FIN envía paquetes TCP únicamente con la bandera FIN activada.
```bash
sudo nmap -sF 192.168.142.130
```

Resultado observado:
```text
Not shown: 998 open|filtered tcp ports (no-response)
3000/tcp closed ppp
8181/tcp closed intermapper
```

![Escaneo FIN](/assets/img/posts/net-reco-vuln-nmap/net-6.png)*Figura 6: Escaneo FIN*

##### 4.3. XMAS Scan
El escaneo XMAS envía paquetes TCP con las banderas FIN, PSH y URG activadas de forma simultánea.
```bash
sudo nmap -sX 192.168.142.130
```

Resultado observado:
```text
Not shown: 998 open|filtered tcp ports (no-response)
3000/tcp closed ppp
8181/tcp closed intermapper
```
Estos escaneos permiten observar respuestas anómalas o la ausencia de respuesta ante paquetes TCP no convencionales, lo que ayuda a inferir el estado de determinados puertos bajo entornos con sistemas de filtrado o cortafuegos.

![Escaneo XMAS](/assets/img/posts/net-reco-vuln-nmap/net-7.png)*Figura 7: Escaneo XMAS*

#### 5. Análisis de tráfico con Wireshark
Durante los escaneos se capturó tráfico de red con Wireshark para analizar las anomalías estructurales generadas por cada técnica de inyección.

* **NULL Scan:** Se observaron paquetes dirigidos al host carentes de flags activos (valor hexadecimal `0x000`).
  Filtro aplicado:
  ```text
  ip.addr == 192.168.142.130 && tcp && tcp.flags == 0x000
  ```

![Wireshark Escaneo NULL](/assets/img/posts/net-reco-vuln-nmap/net-8.png)*Figura 8: Wireshark Escaneo NULL*

* **FIN Scan:** Los paquetes capturados mostraban únicamente la bandera FIN activa, sin el acompañamiento de ACK.
  Filtro aplicado:
  ```text
  ip.addr == 192.168.142.130 && tcp && tcp.flags.fin == 1 && tcp.flags.ack == 0
  ```

![ Wireshark Escaneo FIN](/assets/img/posts/net-reco-vuln-nmap/net-9.png)*Figura 9: Wireshark Escaneo FIN*

* **XMAS Scan:** Las tramas presentaban la coincidencia inusual de las banderas FIN, PSH y URG encendidas a la vez.
  Filtro aplicado:
  ```text
  ip.addr == 192.168.142.130 && tcp && tcp.flags.fin == 1 && tcp.flags.push == 1 && tcp.flags.urg == 1
  ```

![ Wireshark Escaneo XMAS](/assets/img/posts/net-reco-vuln-nmap/net-10.png)*Figura 10: Wireshark Escaneo XMAS*

El análisis con Wireshark permitió validar de forma visual el comportamiento de las técnicas ofensivas y comprender cómo se representan estas firmas de red en una captura real de tráfico.

#### 6. Identificación de vulnerabilidades asociadas
A partir del fingerprinting de servicios, se seleccionaron e investigaron varias vulnerabilidades públicas críticas asociadas a las versiones descubiertas:

![Fingerprinting de los servicios, por ejemplo, Samba](/assets/img/posts/net-reco-vuln-nmap/net-11.png)*Figura 11: Fingerprinting de los servicios, por ejemplo, Samba*

##### 6.1. Samba - CVE-2017-7494 (SambaCry)
El servicio Samba fue detectado en el puerto `445/tcp` con una versión identificada como `Samba smbd 3.X - 4.X`. La vulnerabilidad afecta a protocolos SMB y permite la ejecución remota de código mediante la carga de una biblioteca compartida maliciosa en un recurso accesible con permisos de escritura.
* **Vector de ataque:** Es remoto a través de la exposición del servicio en el puerto `445/tcp`. Si existe un recurso compartido con permisos de escritura configurados, un atacante podría subir una librería `.so` maliciosa y forzar su carga remota mediante Samba.
* **Impacto potencial:** Ejecución remota de código con máximos privilegios (`root`), compromiso completo del sistema, exfiltración de credenciales y uso del activo como pivote dentro de la red interna.

![Vulnerabilidad NIST CVE-2017-7494](/assets/img/posts/net-reco-vuln-nmap/net-12.png)*Figura 12: Vulnerabilidad NIST CVE-2017-7494*

##### 6.2. ProFTPD 1.3.5 - CVE-2015-3306
El servicio FTP se localiza en el puerto `21/tcp` corriendo `ProFTPD 1.3.5`. El fallo afecta directamente al módulo interno `mod_copy`, permitiendo a atacantes remotos leer y escribir archivos del sistema sin autorización previa mediante el abuso de comandos FTP de copia de datos.
* **Vector de ataque:** Remoto a través del servicio FTP expuesto en el puerto `21/tcp`. Un atacante con conectividad podría enviar comandos de control de `mod_copy`, como `SITE CPFR` (origen) y `SITE CPTO` (destino).
* **Ejemplo conceptual de comandos explotados:**
  ```text
  SITE CPFR /etc/passwd
  SITE CPTO /tmp/passwd.copy
  ```
* **Impacto potencial:** Manipulación o lectura no autorizada de archivos confidenciales. Si el entorno dispone de un servicio web activo, se podría escribir una *webshell* ejecutable en el Webroot (`/var/www/html/`) para consolidar una ejecución remota de comandos (RCE).

![Vulnerabilidad NIST CVE-2015-3306](/assets/img/posts/net-reco-vuln-nmap/net-13.png)*Figura 13: Vulnerabilidad NIST CVE-2015-3306*

##### 6.3. UnrealIRCd - CVE-2010-2075
Durante el escaneo de red se identificó el servicio IRC expuesto en el puerto `6697/tcp` ejecutando `UnrealIRCd`. La vulnerabilidad crítica afecta a la compilación `3.2.8.1` y está asociada a una puerta trasera (*backdoor*) inyectada de forma maliciosa en los repositorios oficiales de descarga del software.
* **Vector de ataque:** Es remoto y directo a través del puerto de escucha `6697/tcp`. Un atacante podría enviar una cadena de control específica integrada por los caracteres `"AB"` seguida de comandos del sistema operativo para forzar su ejecución inmediata.
* **Impacto potencial:** Ejecución remota de comandos y compromiso total inmediato de la máquina objetivo, sin requerir ningún tipo de credencial o autenticación previa.

![Vulnerabilidad NIST CVE-2010-2075](/assets/img/posts/net-reco-vuln-nmap/net-14.png)*Figura 14: Vulnerabilidad NIST CVE-2010-2075*

---

### 🧪 Validación (Proof of Concept)

La validación de la actividad se realizó mediante la correlación directa entre los resultados estructurados por Nmap y las capturas analizadas en la telemetría de red.

* **Validación de conectividad:** Confirmación mediante ICMP directo (`0% packet loss`), garantizando la estabilidad del canal.
* **Validación de servicios expuestos:** Verificación manual de banners críticos de software expuesto:
  - `21/tcp open ftp ProFTPD 1.3.5` -> Valida análisis de **CVE-2015-3306**.
  - `445/tcp open netbios-ssn Samba smbd 3.X - 4.X` -> Valida análisis de **CVE-2017-7494**.
  - `6697/tcp open irc UnrealIRCd` -> Valida análisis de **CVE-2010-2075**.
* **Validación de flags TCP con Wireshark:** Las firmas capturadas en red corroboran el envío exacto de las cabeceras malformadas e inválidas inyectadas por Nmap durante los análisis avanzados (`tcp.flags == 0x000`, `tcp.flags.fin == 1`, etc.).

---

### 📝 Resumen Defensivo

En esta actividad se ha realizado un proceso completo de reconocimiento sobre una máquina objetivo en red local, combinando técnicas de descubrimiento de hosts, escaneo de puertos, detección de versiones y análisis de tráfico con Wireshark. El laboratorio permite comprender tanto la metodología ofensiva de enumeración como las evidencias defensivas que pueden utilizarse para detectar este tipo de actividad en una infraestructura real.

> **Indicadores de Reconocimiento Detectables (IoCs):** Desde la perspectiva del analista de un SOC, las actividades de escaneo dejan trazas medibles en los sistemas de monitoreo técnico. Se deben generar reglas y alertas de correlación ante múltiples intentos de conexión concurrentes desde una única IP origen, barridos de puertos secuenciales y el tránsito de paquetes TCP inválidos que violen las directivas del estándar RFC.

---

### 💡 Conclusiones

* **Defensa en Profundidad:** La exposición de servicios obsoletos o desactualizados como FTP, SMB o IRC incrementa críticamente la superficie de ataque de una compañía. Queda demostrada la necesidad imperativa de aplicar controles multi-capa, incluyendo una estricta segmentación de red mediante DMZs para mitigar movimientos laterales rápidos, hardening de aplicaciones y monitorización continua de eventos.
* **Optimización y Rendimiento:** La combinación técnica de Nmap y Wireshark otorga una visibilidad completa del ecosistema. Nmap facilita la identificación veloz de activos, puertos y versiones de software, mientras que Wireshark valida la veracidad del tráfico inyectado, permitiendo el análisis fino de flags en firmas TCP malformadas (NULL, FIN, XMAS).
* **Mitigación y Buenas Prácticas:** Para contener los riesgos detectados se recomienda actualizar los servicios vulnerables a versiones estables y vigentes, deshabilitar módulos innecesarios (como la directiva `mod_copy` en ProFTPD), restringir los accesos perimetrales a SMB/FTP por cortafuegos, aplicar mínimos privilegios sobre los procesos del sistema y desplegar sistemas de detección de intrusos (*IDS/IPS*, como Snort o Suricata) conectados a un SIEM centralizado para aislar tempranamente patrones de reconocimiento anómalos.