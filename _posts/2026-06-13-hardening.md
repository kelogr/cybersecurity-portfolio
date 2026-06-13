---
layout: post
title: Hardening de Active Directory en Windows Server 2022 siguiendo Guías CCN-STIC (ENS)
date: 2026-06-10 16:00:00 +0200
categories: [Cybersecurity, Infrastructure Security]
tags: [academic, windows-server, active-directory, hardening, ccn-stic]
---

## 🛡️ Hardening de Active Directory en Windows Server 2022 siguiendo Guías CCN-STIC (ENS)

| Campo | Descripción |
| :--- | :--- |
| **Tecnologías / Herramientas** | Windows Server 2022, Active Directory, GPO, CCN-STIC, ENS, Windows 11 |
| **Tags / Keywords** | #Hardening #ActiveDirectory #CCNSTIC #ENS #SecurityCompliance #WindowsServer #BlueTeam |
| **Categorías** | Seguridad en Infraestructuras / Cumplimiento Normativo |
| **Autor** | Kevin López Granado |

---

## 🛡️ Hardening de Active Directory en Windows Server 2022 siguiendo Guías CCN-STIC (ENS)

### 🎯 Objetivo

El objetivo de esta actividad es implantar y configurar las políticas de seguridad del **Centro Criptológico Nacional (CCN)** en un entorno de dominio corporativo, a través del servicio **Active Directory** en Windows Server 2022, de acuerdo con los requerimientos del Esquema Nacional de Seguridad (ENS) para categoría básica.

Se implantarán y adaptarán las siguientes políticas específicas:
* **CCN-STIC-570A23 ENS incremental Dominio categoría básica:** Para definir la política de seguridad global del dominio.
* **CCN-STIC-570A23 ENS incremental Controladores de Dominio categoría básica:** Para establecer la seguridad del servicio controlador de dominio (DC).
* **CCN-STIC-599A23 ENS incremental clientes categoría básica:** Para establecer la seguridad de las estaciones de trabajo del dominio.

---

### 🛠️ Prerrequisitos

Para reproducir este entorno de laboratorio es necesario disponer de:
* Un servidor virtualizado con **Windows Server 2022**.
* Una máquina virtual cliente con **Windows 11**.
* Conectividad interna aislada entre ambas máquinas mediante un segmento de red local.
* Plantillas oficiales de directivas de grupo (.inf / .admx) descargadas del portal del CCN-CERT.

---

### 🚀 Ejecución

#### 1. Preparación del Servidor Local y Roles Base
El proyecto se confecciona simulando una implementación en infraestructura personal. Tras el despliegue inicial del sistema operativo base, se ejecutan las siguientes configuraciones previas indispensables:

##### 1.1. Formateo y Asignación de Nombre de Equipo
Se accede a las propiedades del servidor local para modificar el nombre genérico por un formato estandarizado e identificativo corporativo (`WXXX`).

![Asignación de nombre de equipo en servidor local](/assets/img/posts/hardening-ad/figura1.png)
_Figura 1: Asignación de nombre de equipo en servidor local_

##### 1.2. Instalación de Active Directory y Servidor DNS
Tras el reinicio obligatorio para aplicar el nombre del host, se arranca el *Asistente para agregar roles y características* del Administrador del Servidor.

Se selecciona el servidor de destino correspondiente del grupo local:
![Selección de servidor para añadir características](/assets/img/posts/hardening-ad/figura2.png)
_Figura 2: Selección de servidor para añadir características_

A continuación, se marcan para su instalación simultánea los roles de **Servicios de dominio de Active Directory (AD DS)** y **Servidor DNS**. El servicio DNS proporcionará la resolución de nombres necesaria para que los clientes locales se comuniquen con el controlador de dominio de forma transparente mediante TCP/IP.

![Selección de roles para el servidor](/assets/img/posts/hardening-ad/figura3.png)
_Figura 3: Selección de roles para el servidor_

##### 1.3. Promoción del Servidor y Configuración de Bosque
Completada la instalación de los roles básicos, se procede a promocionar el servidor a Controlador de Dominio. Se define la creación de un nuevo bosque raíz asignándole la nomenclatura corporativa oficial `dXXX.es`.

![Creación del nuevo dominio](/assets/img/posts/hardening-ad/figura4.png)
_Figura 4: Especificación del nombre de dominio raíz_

Posteriormente, el asistente valida y asigna de forma automatizada el nombre de dominio NetBIOS equivalente (`dXXX`), necesario para la resolución de nombres en sistemas heredados.

![Definición de dominio NetBIOS](/assets/img/posts/hardening-ad/figura5.png)
_Figura 5: Definición de dominio NetBIOS_

Una vez finalizada la promoción y tras el reinicio automático del sistema, se verifica la correcta ejecución del servicio y la creación de las zonas de búsqueda directa e inversa asociadas al nuevo dominio corporativo.

![Servidor DNS activo en el Administrador](/assets/img/posts/hardening-ad/figura6.png)
_Figura 6: Servidor DNS activo en el Administrador del Servidor_

![Zonas de búsqueda indexadas en el rol DNS](/assets/img/posts/hardening-ad/figura7.png)
_Figura 7: Detalle de registros de zona estáticos (SOA, NS, A) en el Administrador de DNS_

##### 1.4. Provisión de Usuarios del Dominio
Para permitir el acceso y autenticación desde estaciones de trabajo cliente, se genera la primera cuenta de usuario del dominio (ej. `uXXX`) aplicando políticas de contraseña robusta inicial.

![Creación de nuevo usuario en el dominio](/assets/img/posts/hardening-ad/figura8.png)
_Figura 8: Configuración de la cuenta de usuario del dominio_

##### 1.5. Configuración del Servidor DHCP
Con el fin de automatizar la asignación de direccionamiento IP y DNS a los puestos de trabajo finales que vayan uniéndose al entorno, se agrega el rol de **Servidor DHCP** mediante el asistente de características.

![Instalación Servidor DHCP](/assets/img/posts/hardening-ad/figura9.png)
_Figura 9: Instalación del rol Servidor DHCP_

Se inicia el asistente para Ámbito Nuevo configurando el identificador local:
![Nombre y descripción Servidor DHCP](/assets/img/posts/hardening-ad/figura10.png)
_Figura 10: Asignación de nombre y descripción al ámbito DHCP local_

Se define estrictamente el intervalo o rango de direcciones IP disponibles para distribución automática, estableciendo el mínimo en `192.168.5.3` y el máximo en `192.168.5.100`, con una máscara de subred acorde al segmento local.

![Rangos de distribución del Servidor DHCP](/assets/img/posts/hardening-ad/figura12.png)
_Figura 11: Configuración de límites IP mínimo y máximo del ámbito_

![Ámbito DHCP activo](/assets/img/posts/hardening-ad/figura11.png)
_Figura 12: Verificación del estado activo del ámbito DHCP en el servidor_

---

#### 2. Conexión de Estaciones Cliente (Windows 11)
Desde la máquina virtual de Windows 11 asignada a la estación de trabajo, se valida la conectividad de capa 3 ejecutando tazas de eco ICMP dirigidas al dominio raíz (`dXXX.es`), verificando que el direccionamiento dinámico entregado por el DHCP local resuelve correctamente contra la IP estática del servidor (`192.168.5.2`).

```powershell
ping dXXX.es
```
_Salida de trazas esperada en cliente:_
```text
Haciendo ping a dXXX.es [192.168.5.2] con 32 bytes de datos:
Respuesta desde 192.168.5.2: bytes=32 tiempo=1ms TTL=128
Respuesta desde 192.168.5.2: bytes=32 tiempo=1ms TTL=128

Estadísticas de ping para 192.168.5.2:
    Paquetes: enviados = 2, recibidos = 2, perdidos = 0 (0% perdidos)
```

Confirmado el enlace, se accede a la configuración avanzada del sistema para modificar la pertenencia del equipo, introduciendo las credenciales autorizadas del dominio para realizar la unión formal.

![Introducción de credenciales de Active Directory en Windows 11](/assets/img/posts/hardening-ad/figura14.png)
_Figura 13: Introducción de credenciales de Active Directory en Windows 11_

![Mensaje de bienvenida tras unirse correctamente al dominio](/assets/img/posts/hardening-ad/figura15.png)
_Figura 14: Mensaje de bienvenida tras unirse correctamente al dominio dXXX.es_

De vuelta en el Administrador de DHCP del Servidor, se constata la correcta entrega y asignación de la concesión temporal de IP hacia el nombre de host cliente correspondiente.

![Registro de la concesión activa del pool DHCP en el controlador](/assets/img/posts/hardening-ad/figura16.png)
_Figura 15: Registro de la concesión activa del pool DHCP en el controlador_

---

#### 3. Implementación de Directivas de Grupo CCN-STIC

Para iniciar el despliegue del bastionado gubernamental, se accede a la herramienta *Usuarios y equipos de Active Directory* para crear una nueva Unidad Organizativa (OU) dedicada llamada **Clientes Windows**.

Dentro de esta OU se aloja la cuenta de equipo del puesto Windows 11, mientras que el objeto del Windows Server 2022 se mantiene centralizado bajo la OU predeterminada **Domain Controllers**. Esto permite segmentar y perfilar el impacto de cada GPO.

![Estructura de la Unidad Organizativa Clientes Windows](/assets/img/posts/hardening-ad/figura17.png)
_Figura 16: Estructura de la Unidad Organizativa Clientes Windows_

![Verificación del objeto controlador en su respectiva unidad](/assets/img/posts/hardening-ad/figura18.png)
_Figura 17: Verificación del objeto controlador en su respectiva unidad_

Se procede a la importación y vinculación estratégica de las directivas descargadas desde el portal de CCN-CERT en la consola de *Administración de directivas de grupo*:
* **Raíz del Dominio (`dXXX.es`):** Vinculada a la GPO `CCN-STIC-570A23 Incremental Dominio`.
* **OU Domain Controllers:** Vinculada a la GPO `CCN-STIC-570A23 Incremental DC`.
* **OU Clientes Windows:** Vinculada a la GPO `CCN-STIC-599A23 Incremental Clientes Windows`.

![Árbol de vinculación de objetos GPO en el dominio corporativo](/assets/img/posts/hardening-ad/figura19.png)
_Figura 18: Árbol de vinculación de objetos GPO en el dominio corporativo_

---

#### 4. Adaptaciones y Ajustes de las Directivas

Con el fin de conciliar los altos niveles de seguridad exigidos por el CCN con la usabilidad operativa del entorno, se aplicaron y documentaron las siguientes modificaciones granulares sobre las plantillas base:

##### 4.1. Tipos de Cifrado Permitidos para Kerberos (Dominio y DC)
* **Configuración:** Se forzó la habilitación exclusiva de los algoritmos seguros `AES128_HMAC_SHA1` y `AES256_HMAC_SHA1`.
* **Justificación:** Se endurece el protocolo de autenticación deshabilitando cifrados obsoletos y vulnerables a ataques de degradación (como RC4 y DES), alineando la infraestructura con los estándares criptográficos vigentes del ENS.

![Directiva de seguridad de red para tipos de cifrado Kerberos](/assets/img/posts/hardening-ad/figura20.png)
_Figura 19: Directiva de seguridad de red para tipos de cifrado Kerberos_

##### 4.2. Aviso Legal Informativo de Inicio de Sesión
* **Configuración:** Se inyectó en el texto de mensaje para usuarios el párrafo legal normativo corporativo ("Está usted accediendo a un equipo propiedad de la Organización...").
* **Justificación:** Cumplimiento legal estricto orientando a informar a los usuarios de sus obligaciones, restricciones de uso y políticas de monitorización previas al acceso lógico de los activos corporativos.

![Configuración del título y texto del mensaje interactivo de inicio de sesión](/assets/img/posts/hardening-ad/figura21.png)
_Figura 20: Configuración del título y texto del mensaje interactivo de inicio de sesión_

##### 4.3. Umbral de Bloqueo de Cuentas de Usuario
* **Configuración:** Límite máximo establecido en **3 intentos de inicio de sesión no válidos**.
* **Justificación:** Control proactivo para mitigar e interceptar ataques automatizados de fuerza bruta o ataques dirigidos de diccionario sobre la base de datos de Active Directory.

![Directiva de umbral de bloqueo fijado en 3 intentos fallidos](/assets/img/posts/hardening-ad/figura22.png)
_Figura 21: Directiva de umbral de bloqueo fijado en 3 intentos fallidos_

##### 4.4. Endurecimiento de Complejidad y Vigencia de Credenciales
* **Configuración:** Longitud mínima obligatoria de **12 caracteres**, activación del filtro de complejidad, memoria de historial fijada en **10 contraseñas anteriores** y una vigencia máxima de caducidad de **30 días**.
* **Justificación:** Aumenta exponencialmente la entropía de las claves corporativas. La rotación mensual obligatoria limita el margen de explotación temporal de contraseñas potencialmente comprometidas en fugas de datos.

![Directivas de complejidad, longitud y caducidad de contraseñas](/assets/img/posts/hardening-ad/figura23.png)
_Figura 22: Directivas de complejidad, longitud y caducidad de contraseñas_

##### 4.5. Ofuscación de Identidades por Defecto (Renombrar Administrador)
* **Configuración:** Modificación del nombre de inicio de sesión de la cuenta por defecto de administración por un identificador personalizado (`Admin_XXX`).
* **Justificación:** Aplicación de seguridad por oscuridad. Al alterar la cuenta "Administrador" (objetivo mundial por defecto de scripts maliciosos), se obliga a los atacantes a tener que adivinar tanto el usuario como el password, neutralizando ataques genéricos automatizados.

![Modificación del string del nombre de la cuenta del administrador local](/assets/img/posts/hardening-ad/figura24.png)
_Figura 23: Modificación del string del nombre de la cuenta del administrador local_

##### 4.6. Mitigación de Conflictos de Perfil en Puestos de Trabajo (Clientes)
* **Configuración:** Se modificó a **Deshabilitada** la directiva de *Mostrar información acerca de inicios de sesión anteriores*.
* **Justificación:** Durante las pruebas iniciales sobre las estaciones cliente con Windows 11, esta directiva generaba fallos críticos en la interfaz ("Otro usuario"), bloqueando la sincronización lícita del perfil del dominio debido a incompatibilidades con trazas locales. Su desactivación solventó el conflicto operativo garantizando la disponibilidad del puesto.

![Configuración de directiva para inicios de sesión anteriores en clientes](/assets/img/posts/hardening-ad/figura29.png)
_Figura 24: Configuración de directiva para inicios de sesión anteriores en clientes_

##### 4.7. Control de Inactividad y Bloqueo Automatizado
* **Configuración:** Límite fijado en **600 segundos** (10 minutos).
* **Justificación:** Fuerza el cierre de sesión o bloqueo de pantalla automatizado ante descuidos o ausencias físicas del personal en sus puestos, garantizando un equilibrio óptimo entre seguridad interna y rendimiento de usuario.

![Restricción del límite de inactividad en el inicio de sesión interactivo](/assets/img/posts/hardening-ad/figura30.png)
_Figura 25: Restricción del límite de inactividad en el inicio de sesión interactivo_

##### 4.8. Gestión Centralizada de Actualizaciones de Sistema
* **Configuración:** Activación del estado *4 - Descargar automáticamente y programar la instalación*.
* **Justificación:** Asegura el despliegue sistemático de parches de seguridad críticos procedentes de Windows Update de manera programada, controlando las ventanas de mantenimiento para evitar interrupciones en jornadas laborales críticas.

![Configuración del comportamiento de las actualizaciones automáticas](/assets/img/posts/hardening-ad/figura31.png)
_Figura 26: Configuración del comportamiento de las actualizaciones automáticas_

##### 4.9. Continuidad de Servicio frente a Saturación de Registros
* **Configuración:** Se marcó como **Deshabilitada** la directiva *Auditoría: apagar el sistema de inmediato si no se pueden registrar las auditorías de seguridad*.
* **Justificación:** Se prioriza la disponibilidad operativa de la estación de trabajo frente a la integridad absoluta del log del sistema. Evita apagados repentinos accidentales de equipos de cara al usuario en caso de que los eventos de seguridad se saturen antes de ser rotados.

![Directiva para controlar el apagado por fallos de auditoría de seguridad](/assets/img/posts/hardening-ad/figura32.png)
_Figura 27: Directiva para controlar el apagado por fallos de auditoría de seguridad_

---

### 🧪 Evidencias de Funcionamiento

Las comprobaciones en caliente demuestran la propagación efectiva de las políticas en todas las capas del entorno:

#### Prueba 1: Despliegue del Banner Legal en el Servidor
Al intentar acceder local o remotamente al Controlador de Dominio, el sistema intercepta el flujo forzando la visualización del aviso legal redactado en las directivas previas antes de habilitar el panel de introducción de credenciales.

![Interfaz del banner de seguridad desplegado con éxito en el Servidor](/assets/img/posts/hardening-ad/figura27.png)
_Figura 28: Interfaz del banner de seguridad desplegado con éxito en el Servidor_

#### Prueba 2: Detección de la Directiva de Bloqueo de Cuentas
Tras inyectar intencionadamente 3 contraseñas erróneas en el prompt de acceso de la estación de trabajo, la GPO del dominio responde bloqueando temporalmente el inicio de sesión, impidiendo ataques continuos de fuerza bruta.

![Mensaje de bloqueo emitido por el sistema tras agotar el umbral de errores](/assets/img/posts/hardening-ad/figura28.png)
_Figura 29: Mensaje de bloqueo emitido por el sistema tras agotar el umbral de errores_

#### Prueba 3: Coherencia de Políticas en el Extremo Cliente (Windows 11)
El cliente Windows 11 hereda correctamente las configuraciones de la directiva de la OU asignada, replicando el banner legal corporativo mapeado en los objetos aplicados, manteniendo la coherencia de cumplimiento normativo en toda la organización.

![Visualización del Aviso de Seguridad e implicaciones legales en Windows 11](/assets/img/posts/hardening-ad/figura34.png)
_Figura 30: Visualización del Aviso de Seguridad e implicaciones legales en Windows 11_

---

### 📝 Resumen Defensivo

Este laboratorio consolida las bases de la administración de sistemas orientada a la seguridad corporativa de nivel Blue Team. El bastionado perimetral o a nivel de red resulta estéril si el corazón de la autenticación de la compañía (Active Directory) mantiene configuraciones por defecto permisivas.

> **Análisis Forense y Visibilidad:** El endurecimiento masivo implementado a través de las plantillas CCN-STIC garantiza que cualquier intento de violación del flujo normal de accesos (como ataques de diccionario o accesos fuera de hora) quede registrado obligatoriamente en los eventos de Windows (Event Viewer) con IDs específicos, facilitando su recolección por agentes SIEM de un SOC para una detección temprana antes de la fase de compromiso.

---

### 💡 Conclusiones

* **Defensa Multi-Capa Orientada al Cumplimiento:** La correcta integración de las directivas del Centro Criptológico Nacional permite elevar el nivel de madurez tecnológica de la organización, facilitando la auditoría y certificación en el Esquema Nacional de Seguridad (ENS).
* **Equilibrio entre Seguridad y Disponibilidad:** Como se evidenció en la mitigación del error de perfiles en clientes Windows 11, un analista de seguridad no debe limitarse a aplicar plantillas de forma ciega; es indispensable testear, depurar e introducir excepciones para salvaguardar la usabilidad de los entornos de trabajo del usuario final.
* **Recomendación de Robustez:** Para dar continuidad a la robustez de este dominio, se aconseja complementar este endurecimiento implementando arquitecturas de **Tiering de Active Directory** (separando credenciales de administración de servidores de las de puestos de usuario) y desplegando soluciones **LAPS** para atomizar y rotar de forma aleatoria la contraseña del administrador local de cada Endpoint.
