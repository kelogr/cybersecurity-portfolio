---
layout: post
title: Detección y Análisis de Vulnerabilidades en Software de Terceros
date: 2026-06-14 11:00:00 +0200
categories: [Cybersecurity, Software Security]
tags: [academic, vulnerability-analysis, owasp, dependency-check, cve, devsecops]
---

## 🛡️ Detección y Análisis de Vulnerabilidades en Software de Terceros - [ACADEMIC - Beginner]

| Campo | Descripción |
| :--- | :--- |
| **Tecnologías / Herramientas** | OWASP Dependency-Check, CVE, NVD, Apache Tomcat, XStream, Underscore.js, jQuery, jQuery-UI, jose4j |
| **Tags / Keywords** | #VulnerabilityAssessment #CVE #OWASP #DependencyCheck #SecurityAnalysis #DevSecOps #PatchManagement #SoftwareSupplyChain |
| **Categorías** | Seguridad en Aplicaciones / Análisis de Vulnerabilidades |
| **Autor** | Kevin López Granado |

---

## 🛡️ Detección y Análisis de Vulnerabilidades en Software de Terceros - [ACADEMIC - Beginner]

### 🎯 Objetivo

El objetivo de esta actividad es realizar un inventario de seguridad exhaustivo y un análisis técnico sobre las dependencias de software de terceros utilizadas en aplicaciones. Mediante el uso de la herramienta automatizada **OWASP Dependency-Check**, se identifican vulnerabilidades públicas conocidas (CVE), evaluando la existencia de parches oficiales y determinando las acciones de remediación necesarias para reducir la superficie de exposición al riesgo.

---

### 🛠️ Prerrequisitos

* Entorno de ejecución para **OWASP Dependency-Check**.
* Conectividad y acceso actualizado a las bases de datos de vulnerabilidades **NVD (National Vulnerability Database)** y **MITRE**.
* Proyecto o repositorio de muestra con dependencias de terceros (módulos Java, librerías JavaScript, servidores web, etc.).

---

### 🚀 Ejecución del Análisis y Evidencias

Se configuró y ejecutó OWASP Dependency-Check para escanear de forma automatizada las dependencias del proyecto a través de sus identificadores CPE (Common Platform Enumeration). A continuación se detalla la descripción técnica de cada vulnerabilidad detectada junto con su respectivo bloque de evidencia:

| N° | Software | Versión | Vulnerabilidad (CVE) | Fecha Pub. Parche | Recomendación de Subsanación |
| :---: | :--- | :---: | :--- | :---: | :--- |
| **1** | Underscore.js | 1.10.2 | CVE-2021-23358 | SÍ (29/03/2021) | Actualizar a la versión 1.13.0-2 o 1.12.1. |
| **2** | jquery-ui | 1.10.3 | CVE-2021-41184 | Sí (26/10/2021) | Actualizar a la versión 1.13.0 o superior. |
| **3** | jose4j | 0.9.3 | CVE-2023-51775 | SÍ (28/02/2024) | Actualizar a la versión 0.9.4 o superior. |
| **4** | Apache Tomcat | 10.1.36 | CVE-2025-31651 | SÍ (28/04/2025) | No es posible identificar la versión específica debido a un error de formato web. |
| **5** | Apache Tomcat | 10.1.36 | CVE-2025-55754 | Sí (27/10/2025) | Actualizar a la versión 11.0.11, 10.1.45, 9.0.109, o superior. |
| **6** | Apache Tomcat | 10.1.36 | CVE-2025-49124 | Sí (16/06/2025) | Actualizar a la versión 11.0.8, 10.1.42, 9.0.106, o superior. |
| **7** | Apache Tomcat | 10.1.36 | CVE-2025-31650 | SÍ (28/04/2025) | Actualizar a la versión 11.0.6, 10.1.40, 9.0.104, o superior. |
| **8** | Apache Tomcat | 10.1.36 | CVE-2025-48988 | SÍ (16/06/2025) | Actualizar a la versión 11.0.8, 10.1.42 o 9.0.106, o superior. |
| **9** | Apache Tomcat | 10.1.36 | CVE-2025-48989 | si (13/08/2025) | Actualizar a la versión 11.0.10, 10.1.44 o 9.0.108, o superior. |
| **10** | XStream | 1.4.5 | CVE-2013-7285 | SÍ (15/05/2019) | Actualizar a la versión 1.4.6 o superior. |
| **11** | XStream | 1.4.5 | CVE-2021-21345 | Si (22/03/2021) | Actualizar a la versión 1.4.16 o superior. |
| **12** | XStream | 1.4.5 | CVE-2021-21344 | Si (22/03/2021) | Actualizar a la versión 1.4.16 o superior. |
| **13** | XStream | 1.4.5 | CVE-2021-21346 | Si (22/03/2021) | Actualizar a la versión 1.4.16 o superior. |
| **14** | XStream | 1.4.5 | CVE-2016-3674 | Si (17/05/2016) | Actualizar a la versión 1.4.9 o superior. |
| **15** | XStream | 1.4.5 | Si (16/06/2025) | SÍ (16/09/2022) | Actualizar a la versión 1.4.20 o superior. |
| **16** | XStream | 1.4.5 | CVE-2021-21342 | Si (23/03/2021) | Actualizar a la versión 1.4.16 o superior. |
| **17** | XStream | 1.4.5 | CVE-2021-21351 | Si (23/03/2021) | Actualizar a la versión 1.4.16 o superior. |
| **18** | XStream | 1.4.5 | Si (23/08/2021) | Si (23/08/2021) | Actualizar a la versión 1.4.18 o superior. |
| **19** | XStream | 1.4.5 | CVE-2021-39144 | Si (23/08/2021) | Actualizar a la versión 1.4.18 o superior. |
| **20** | jquery | 1.10.2 | CVE-2015-9251 | SÍ (18/01/2018) | Actualizar a la versión 3.0.0 o superior. |

#### 1. Underscore.js (v1.10.2)
* **CVE-2021-23358:** Permite la inyección de código arbitrario a través de la función de renderizado de plantillas si la aplicación procesa variables de entrada no saneadas en el motor de renderizado.
* **Evidencia:**
  ![Figura 1](/assets/img/posts/dependency-check/ilustracion1.png)
  *Figura 1. Vulnerabilidad CVE-2021-23358*

#### 2. jQuery-UI (v1.10.3)
* **CVE-2021-41184:** Fallo de Cross-Site Scripting (XSS) debido a una falta de sanitización en el parámetro `of` de la función de posicionamiento de componentes visuales (botones).
* **Evidencia:**
  ![Figura 2](/assets/img/posts/dependency-check/ilustracion2.png)
  *Figura 2. Vulnerabilidad CVE-2021-41184*

#### 3. jose4j (v0.9.3)
* **CVE-2023-51775:** Consumo excesivo y descontrolado de recursos de CPU durante la derivación de claves mediante el algoritmo PBKDF2, facilitando ataques de denegación de servicio.
* **Evidencia:**
  ![Figura 3](/assets/img/posts/dependency-check/ilustracion3.png)
  *Figura 3. Vulnerabilidad CVE-2023-51775*

#### 4. Apache Tomcat (v10.1.36)
* **CVE-2025-31651:** Inconsistencia en la interpretación del formato de peticiones entrantes específicas que puede derivar en fallos de ocultación o contrabando de peticiones HTTP (HTTP Request Smuggling).
* **Evidencia:**
  ![Figura 4](/assets/img/posts/dependency-check/ilustracion4.png)
  *Figura 4. Vulnerabilidad CVE-2025-31651*

* **CVE-2025-55754:** Neutralización inadecuada de secuencias de escape ANSI que permite la inyección de comandos visuales o manipulación de registros de auditoría si el administrador visualiza la salida en terminales vulnerables.
* **Evidencia:**
  ![Figura 5](/assets/img/posts/dependency-check/ilustracion5.png)
  *Figura 5. Vulnerabilidad CVE-2025-55754*

* **CVE-2025-49124:** Inclusión y carga de archivos locales a través de rutas de búsqueda no confiables (*Untrusted Search Path*) en entornos Windows que facilita el secuestro de la ejecución binaria.
* **Evidencia:**
  ![Figura 6](/assets/img/posts/dependency-check/ilustracion6.png)
  *Figura 6. Vulnerabilidad CVE-2025-49124*

* **CVE-2025-31650:** Gestión deficiente del procesamiento de errores al recibir cabeceras HTTP inválidas, provocando una fuga progresiva de memoria e hilos que induce una denegación de servicio (DoS).
* **Evidencia:**
  ![Figura 7](/assets/img/posts/dependency-check/ilustracion7.png)
  *Figura 7. Vulnerabilidad CVE-2025-31650*

* **CVE-2025-48988:** Defecto lógico en el procesamiento interno del contenedor web que permite accesos concurrentes asíncronos no autorizados a ciertos contextos del backend.
* **Evidencia:**
  ![Figura 8](/assets/img/posts/dependency-check/ilustracion8.png)
  *Figura 8. Vulnerabilidad CVE-2025-48988*

* **CVE-2025-48989:** Validación incorrecta en la serialización y persistencia temporal de las sesiones de usuario activas, abriendo la posibilidad de interceptación de identificadores de sesión.
* **Evidencia:**
  ![Figura 9](/assets/img/posts/dependency-check/ilustracion9.png)
  *Figura 9. Vulnerabilidad CVE-2025-48989*

#### 5. XStream (v1.4.5)
* **CVE-2013-7285:** Ejecución Remota de Código (RCE) causada por la deserialización de objetos XML no confiables sin restricciones de tipos ni un cortafuegos activo en el unmarshaling.
* **Evidencia:**
  ![Figura 10](/assets/img/posts/dependency-check/ilustracion10.png)
  *Figura 10. Vulnerabilidad CVE-2013-7285*

* **CVE-2021-21345:** Deserialización insegura de objetos que permite la manipulación del flujo interno del servidor logrando ejecutar comandos o archivos arbitrarios del sistema de archivos local.
* **Evidencia:**
  ![Figura 11](/assets/img/posts/dependency-check/ilustracion11.png)
  *Figura 11. Vulnerabilidad CVE-2021-21345*

* **CVE-2021-21344:** Permite a un atacante remoto forzar al motor de XStream a descargar y ejecutar código dinámico alojado externamente (Remote Code Execution).
* **Evidencia:**
  ![Figura 12](/assets/img/posts/dependency-check/ilustracion12.png)
  *Figura 12. Vulnerabilidad CVE-2021-21344*

* **CVE-2021-21346:** Fallo crítico de corrupción de datos y ejecución lógica al deserializar vectores maliciosos construidos específicamente para sobrepasar las listas negras internas.
* **Evidencia:**
  ![Figura 13](/assets/img/posts/dependency-check/ilustracion13.png)
  *Figura 13. Vulnerabilidad CVE-2021-21346*

* **CVE-2016-3674:** Vulnerabilidad de inyección de Entidades Externas XML (XXE) que permite a usuarios maliciosos leer de manera remota configuraciones críticas locales del sistema.
* **Evidencia:**
  ![Figura 14](/assets/img/posts/dependency-check/ilustracion14.png)
  *Figura 14. Vulnerabilidad CVE-2016-3674*

* **CVE-2022-40152:** Agotamiento de recursos y desbordamiento de pila en la librería XML Woodstox subyacente que bloquea los servicios provocando una caída por Denegación de Servicio (DoS).
* **Evidencia:**
  ![Figura 15](/assets/img/posts/dependency-check/ilustracion15.png)
  *Figura 15. Vulnerabilidad CVE-2022-40152*

* **CVE-2021-21342:** Permite ataques de Server-Side Request Forgery (SSRF), manipulando la deserialización para forzar al servidor a realizar peticiones HTTP no autorizadas hacia la infraestructura de red interna.
* **Evidencia:**
  ![Figura 16](/assets/img/posts/dependency-check/ilustracion16.png)
  *Figura 16. Vulnerabilidad CVE-2021-21342*

* **CVE-2021-21351:** Ejecución arbitraria de código mediante la inyección y manipulación de instancias serializadas basadas en tipos de datos nativos desprotegidos.
* **Evidencia:**
  ![Figura 17](/assets/img/posts/dependency-check/ilustracion17.png)
  *Figura 17. Vulnerabilidad CVE-2021-21351*

* **CVE-2021-39141:** Evasión de las defensas y controles internos del filtro de XStream, logrando deserializar clases arbitrarias no permitidas con impacto RCE directo.
* **Evidencia:**
  ![Figura 18](/assets/img/posts/dependency-check/ilustracion18.png)
  *Figura 18. Vulnerabilidad CVE-2021-39141*

* **CVE-2021-39144:** Inyección de payloads XML preparados para corromper las referencias dinámicas internas y ejecutar código ajeno al contexto seguro de la aplicación.
* **Evidencia:**
  ![Figura 19](/assets/img/posts/dependency-check/ilustracion19.png)
  *Figura 19. Vulnerabilidad CVE-2021-39144*

#### 6. jQuery (v1.10.2)
* **CVE-2015-9251:** Vulnerabilidad de Cross-Site Scripting (XSS) en la función encargada de realizar peticiones Ajax entre diferentes dominios sin forzar la comprobación estricta del parámetro de tipo de dato (*dataType*).
* **Evidencia:**
  ![Figura 20](/assets/img/posts/dependency-check/ilustracion20.png)
  *Figura 20. Vulnerabilidad CVE-2015-9251*

### 📂 Entregables del Análisis

Para profundizar en los detalles técnicos, resultados brutos y las gráficas de severidad generadas durante el escaneo, puedes descargar el informe técnico original exportado por la herramienta:

* **[Descargar Reporte Completo (HTML)](/assets/pdf/dependency-check/Dependency-Check%20Report.pdf)**

Este archivo contiene el inventario completo detectado por **OWASP Dependency-Check**, incluyendo las rutas de los archivos afectados, los niveles de confianza de la herramienta y los enlaces directos a las referencias de la NVD para cada una de las 20 vulnerabilidades analizadas en este ejercicio académico.

---

### 📝 Resumen Defensivo

El análisis automatizado de dependencias mediante herramientas SCA (Software Composition Analysis) constituye un pilar fundamental en la estrategia de seguridad **Shift-Left**. La infraestructura moderna depende de un tejido complejo de componentes open-source de terceros; ignorar el ciclo de vida de estas dependencias es equivalente a dejar vectores de ataque expuestos y documentados públicamente.

El valor real del análisis SCA radica en su integración continua. Automatizar OWASP Dependency-Check dentro de las etapas del pipeline de integración (CI/CD) previene de manera activa el despliegue de paquetes vulnerables hacia los entornos productivos, rompiendo la construcción del código si se superan los umbrales de riesgo aceptables.

---

### 💡 Conclusiones

* **Seguridad en la Cadena de Suministro (Supply Chain):** Desarrollar código seguro en la lógica de negocio propia no es suficiente si el entorno subyacente o las librerías transversales importadas poseen fallos críticos explotables públicamente.
* **Mantenimiento del SBOM (Software Bill of Materials):** Contar con un inventario preciso y transparente de las dependencias agiliza los tiempos de respuesta ante incidentes cuando se publican nuevas vulnerabilidades de día cero (*0-day*).
* **Remediación Dirigida por Riesgo:** El reporte en formato HTML generado por la herramienta proporciona métricas estandarizadas (CVSS Base Score y vectores de ataque). Esto facilita al equipo defensivo clasificar el nivel de severidad técnica reales y priorizar las ventanas de mantenimiento de forma alineada con los requisitos organizativos.