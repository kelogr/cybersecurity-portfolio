# 🛡️ BTJA: Network Analysis Framework (Wireshark, tcpdump & Capstone)

## 📝 Introducción y Objetivos del Repositorio

Este módulo técnico recopila de forma unificada la investigación forense, el análisis de protocolos y la resolución de incidentes correspondientes al bloque de **Análisis de Tráfico de Red** del curso **Blue Team Junior Analyst (BTJA)**. 

El objetivo de este documento es doble: servir como bitácora de aprendizaje metodológico y consolidar un portafolio técnico enfocado en habilidades reales de un **Analista de SOC (Security Operations Center)**. A lo largo de los laboratorios, se despliegan destrezas críticas en tres áreas fundamentales:

1. **Análisis Visual y Forense (Wireshark):** Aislamiento de conversaciones, reconstrucción de flujos (*TCP/HTTP Streams*), análisis de payloads y detección de anomalías en entornos interactivos.
2. **Filtrado Avanzado por CLI (tcpdump):** Uso del motor BPF (*Berkeley Packet Filter*) para optimizar la recolección de paquetes, manipulación y cálculo binario de flags TCP, inspección de offsets de memoria (TTL, Checksums) y automatización con herramientas nativas de Linux (`grep`, `wc`, `unzip`).
3. **Respuesta a Incidentes (Capstone Project):** Correlación de eventos complejos para identificar vectores de ataque reales como movimientos laterales, ataques de denegación de servicio, exfiltración de datos comprometidos y envenenamiento de red (*ARP Spoofing / MitM*).

---

## 🦈 Sección 1: Wireshark Labs (PCAP 1 & PCAP 2)
*Investigación inicial de protocolos, análisis de volumen de datos y extracción de credenciales mediante inspección de flujos.*

### 🧩 Preguntas y Respuestas

#### 1. Which protocol was used over port 3942?
* **Metodología:** Filtrado del puerto específico utilizando la sintaxis nativa de visualización para aislar el protocolo de la capa de aplicación.
* **Evidencia:** `[Inserta aquí la referencia a tu imagen: Paquete o flujo correspondiente]`
* **Resultado:** `[Tu Respuesta Aquí]`

#### 2. What is the IP address of the host that was pinged twice?
* **Metodología:** Filtrado por protocolo ICMP de tipo solicitud de eco (*Echo Request* o *Type 8*) para auditar los hosts objetivos.
* **Evidencia:** `[Imagen: Paquetes ICMP filtrados]`
* **Resultado:** `8.8.4.4`

#### 3. How many DNS query response packets were captured?
* **Metodología:** Aplicación de filtros booleanos avanzados para aislar únicamente las respuestas DNS (`dns.flags.response == 1`) descartando las peticiones de origen.
* **Evidencia:** `[Imagen: Contador de paquetes en la barra de estado]`
* **Resultado:** `[Tu Respuesta Aquí]`

#### 4. What is the IP address of the host which sent the most number of bytes?
* **Metodología:** Análisis a través del panel estadístico de conversaciones (*Statistics > Conversations > IPv4*) ordenando los endpoints por volumen de bytes transmitidos en sentido de salida.
* **Evidencia:** `[Imagen: Panel de conversaciones de Wireshark]`
* **Resultado:** `115.178.9.18`

#### 5. What is the WebAdmin password?
* **Metodología:** Localización de paneles de gestión web HTTP y seguimiento del flujo TCP (*Follow TCP Stream*) para capturar el payload del método POST con las credenciales en texto plano.
* **Evidencia:** `[Imagen: Credenciales capturadas en el stream]`
* **Resultado:** `[Tu Respuesta Aquí]`

#### 6. What is the version number of the attacker’s FTP server?
* **Metodología:** Inspección del banner de bienvenida (*FTP Response: 220*) transmitido en los primeros paquetes del establecimiento de la sesión FTP.
* **Evidencia:** `[Texto del banner extraído]`
* **Resultado:** `pyftpdlib 1.5.5 ready.`

#### 7. Which port was used to gain access to the victim Windows host?
* **Metodología:** Correlación del tráfico buscando el establecimiento de la sesión de explotación. Se analizan conexiones sospechosas con combinaciones específicas de flags `[SYN, ECE, CWR]` para identificar accesos persistentes post-explotación del servidor web.
* **Resultado:** `[Tu Respuesta Aquí]`

#### 8. What is the name of a confidential file on the Windows host?
* **Metodología:** Inspección de transferencias de archivos en la sesión comprometida o listado de comandos ejecutados de forma remota.
* **Evidencia:** `[Imagen: Nombre del archivo confidencial]`
* **Resultado:** `[Tu Respuesta Aquí]`

#### 9. What is the name of the log file that was created at 4:51 AM on the Windows host?
* **Metodología:** Correlación temporal analizando las marcas de tiempo de los paquetes TCP que involucran la creación o transferencia de artefactos ejecutables (`malware.exe`).
* **Evidencia:** `[Imagen: Paquete con marca de tiempo exacta]`
* **Resultado:** `[Tu Respuesta Aquí]`

---

## 🐚 Sección 2: tcpdump & Linux CLI (PCAP 4)
*Filtrado avanzado de paquetes a nivel de bits, análisis de rendimiento y manipulación de cabeceras de red desde la consola.*

### 🧩 Preguntas y Respuestas

#### 1. How many UDP packets have been captured?
* **Metodología:** Filtrado directo por protocolo UDP en la lectura de la captura y encadenamiento mediante *pipe* (`|`) hacia el contador de líneas de la terminal (`wc -l`) para automatizar el cálculo exacto de paquetes en el output.
* **Comando Ejecutado:**
  ```bash
  tcpdump udp -r SBT-PCAP4.pcap | wc -l