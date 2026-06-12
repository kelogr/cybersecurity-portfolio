# Hardening Perimetral y Gestión de Reglas de Estado con Linux Iptables en Arquitecturas DMZ

| Campo | Descripción |
| :--- | :--- |
| **Tecnologías / Herramientas** | [cite_start]Linux Iptables, NETinVM, Netcat (`nc`), KWrite  |
| **Tags / Keywords** | #FirewallHardening, #Iptables, #DMZ-Isolation, #NetworkSecurity, #BlueTeam |
| **Categorías** | Análisis de Amenazas |
| **Autor** | [cite_start]Kevin López Granado  |

---

## Hardening Perimetral en Redes Multizona - [ACADEMIC - Intermediate]

### 🎯 Objetivo

[cite_start]El propósito de esta actividad es reforzar los conocimientos prácticos sobre topologías de defensa perimetral mediante el diseño y la implementación de un conjunto de reglas de filtrado y traducción de direcciones en un cortafuegos (`fw`) basado en **Iptables**[cite: 4, 10]. [cite_start]Se busca aislar y proteger tres segmentos de red diferenciados de la empresa *Example*: una red exterior que simula Internet ($10.5.0.0/24$), una Zona Desmilitarizada o DMZ ($10.5.1.0/24$) y una red interna corporativa ($10.5.2.0/24$)[cite: 9, 238].

---

### 🚀 Ejecución

### 🏢 Topología y Direccionamiento de Interfaces

[cite_start]El cortafuegos central (`fw`) interconecta los segmentos utilizando tres interfaces de red específicas[cite: 10, 11]:
* [cite_start]**`eth0`**: Interfaz conectada a la red externa / Internet (IP: `10.5.0.254`)[cite: 11].
* [cite_start]**`eth1`**: Interfaz conectada a la DMZ (IP: `10.5.1.254`)[cite: 11].
* [cite_start]**`eth2`**: Interfaz conectada a la red interna corporativa (IP: `10.5.2.254`)[cite: 11].

![alt text](images/image.png)

---

### 📊 Matriz de Reglas Iptables

| # | Acción / Requerimiento | Reglas de Iptables |
| --- | --- | --- |
| **1** | [cite_start]Listar las reglas de la tabla filter en modo detallado [cite: 263] | [cite_start]`iptables -t filter -L -v` [cite: 317] |
| **2** | [cite_start]Borrar todas las reglas de la tabla filter [cite: 268] | [cite_start]`iptables -t filter -F` [cite: 317] |
| **3** | [cite_start]Establecer una política restrictiva en la cadena que falta [cite: 270] | [cite_start]`iptables -P INPUT DROPiptables -P FORWARD DROPiptables -P OUTPUT DROP` [cite: 317] |
| **4** | [cite_start]Permitir el tráfico de conexiones ya establecidas en todas las cadenas de la tabla filter [cite: 272] | [cite_start]`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPTiptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPTiptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT` [cite: 317, 322] |
| **5** | [cite_start]Permitir las nuevas conexiones salientes desde la red local a los servidores DNS (UDP) públicos de Cloudflare (IP: 1.1.1.1 y 1.0.0.1) que se encuentran en internet [cite: 275] | [cite_start]`iptables -A FORWARD -i eth2 -o eth0 -s 10.5.2.0/24 -d 1.1.1.1 -p udp --dport 53 -m state --state NEW -j ACCEPTiptables -A FORWARD -i eth2 -o eth0 -s 10.5.2.0/24 -d 1.0.0.1 -p udp --dport 53 -m state --state NEW -j ACCEPT` [cite: 322] |
| **6** | [cite_start]Reasigne como destino al servidor web de la DMZ todas las nuevas conexiones desde la red local a servidores web seguros en internet [cite: 277] | [cite_start]`iptables -t nat -A PREROUTING -i eth2 -s 10.5.2.0/24 -p tcp --dport 443 -j DNAT --to-destination 10.5.1.10` [cite: 322] |
| **7** | [cite_start]Permitir las nuevas conexiones HTTPS desde el servidor web en la DMZ a servidores web de internet [cite: 279] | [cite_start]`iptables -A FORWARD -i eth1 -o eth0 -s 10.5.1.10 -p tcp --dport 443 -m state --state NEW -j ACCEPT` [cite: 322] |
| **8** | [cite_start]Permitir las nuevas conexiones SSH desde el PC exta al servidor web [cite: 281] | [cite_start]`iptables -A FORWARD -i eth0 -o eth1 -s 10.5.0.10 -d 10.5.1.10 -p tcp --dport 22 -m state --state NEW -j ACCEPT` [cite: 322] |
| **9** | [cite_start]Permitir las nuevas conexiones HTTPS y SFTP desde el cortafuegos al servidor de actualizaciones de Debian en España (IP: 82.194.78.250) que se encuentra en internet [cite: 283] | [cite_start]`iptables -A OUTPUT -o eth0 -d 82.194.78.250 -p tcp --dport 443 -m state --state NEW -j ACCEPTiptables -A OUTPUT -o eth0 -d 82.194.78.250 -p tcp --dport 22 -m state --state NEW -j ACCEPT` [cite: 322] |
| **10** | [cite_start]Permitir las nuevas conexiones desde la red local al futuro servidor de escritorio remoto (RDP) que se encontrará en la DMZ (10.5.1.89) [cite: 288] | [cite_start]`iptables -A FORWARD -i eth2 -o eth1 -s 10.5.2.0/24 -d 10.5.1.89 -p tcp --dport 3389 -m state --state NEW -j ACCEPT` [cite: 327] |

---

### 📝 Resumen

En esta actividad se ha diseñado un modelo de seguridad perimetral basado en el principio de **menor privilegio** empleando Linux Iptables. [cite_start]Se eliminó la configuración laxa por defecto y se sustituyó por una política de denegación estricta (`DROP`) en los flujos `INPUT`, `FORWARD` y `OUTPUT`[cite: 268, 270].

[cite_start]A partir de este estado de *Zero Trust*, se definieron reglas con inspección de estado (Stateful) minuciosas, limitando el tráfico según interfaces de entrada/salida, direcciones IP de origen/destino y puertos de capa de transporte específicos[cite: 272, 296]. [cite_start]Entre las soluciones implementadas destaca la integración de técnicas de **DNAT** para forzar el paso del tráfico web interno a través de la DMZ y el aislamiento del tráfico de administración (SSH) a un único origen legítimo en el exterior (`exta`)[cite: 277, 281].

---

### 💡 Conclusiones

- **La Denegación por Defecto como Pilar:** Establecer políticas `DROP` como base del cortafuegos garantiza que cualquier vector de ataque o protocolo no contemplado explícitamente quede completamente neutralizado de forma nativa.
- [cite_start]**Eficiencia mediante Inspección de Estado:** El uso del módulo `state` (`ESTABLISHED,RELATED`) simplifica radicalmente el tablón de reglas[cite: 272]. Permite que el firewall valide automáticamente el tráfico de retorno de las conexiones legítimas sin necesidad de abrir puertos bidireccionales de manera permanente.
- [cite_start]**Defensa en Profundidad con DMZ:** Forzar a la red interna a interactuar con Internet a través de un intermediario en la DMZ (mediante DNAT) evita la exposición directa de los hosts internos, añadiendo una capa de inspección y anonimato dentro de la infraestructura corporativa[cite: 277].
- [cite_start]**Segmentación Estricta de la Administración:** Limitar el acceso SSH a un único direccionamiento IP externo (`exta`) mitiga los ataques de fuerza bruta generalizados y reduce drásticamente la superficie de exposición de los activos críticos de la DMZ[cite: 281].