# Plan de auditoría · Seguridad del canal digital de la Clínica Santa Lucía

**SI-084 · Auditoría de Sistemas**

| | |
|---|---|
| **Apellidos y nombres** | HUANCA MERMA GREGORY BRANDON |
| **Código de estudiante** |2022073898 |
| **URL del repositorio** | `https://github.com/GregoBHM/si084-caso-clinica` |
| **Fecha** | 28/10/2026 |

## 1. Resultados del reconocimiento · OWASP WSTG 4.2

La Clínica Santa Lucía S.A.C. es una institución prestadora de servicios de salud (IPRESS) privada de nivel II-1, ubicada en Tacna, con 162 personas en planilla, 3 de ellas en sistemas, e ingresos de S/ 18 600 000 en 2025. La supervisan SUSALUD, por ser IPRESS, y la Autoridad Nacional de Protección de Datos Personales, por la Ley 29733. Publica en internet la reserva de citas, con 3 850 citas al mes, los resultados de laboratorio, para 12 400 pacientes con cuenta, y las postulaciones de empleo, con 214 CV en 2026. WebSur S.A.C. administra el portal desde 2021.

| Prueba | Identificador WSTG | Lo que observé, con cifras | Declaración del proveedor que contrasta | ¿Coincide? | Evidencia |
|---|---|---|---|---|---|
| E1 | WSTG-INFO-04 | Responden 3 aplicaciones (8080, 8081 y 8088) frente a 1 declarada. El puerto 8081 es un entorno de pruebas con una copia de producción restaurada el 12/09/2026. El puerto 8088 es una API que declara estar en producción, con 3 endpoints /v1/ y documentación en /swagger/. | D1 | No | `evidencias/E1_aplicaciones.txt` |
| E2 | WSTG-INFO-02 | Server: nginx/1.20.2 y X-Powered-By: PHP/7.4.3. PHP 7.4 perdió el soporte de seguridad el 28/11/2022, y nginx 1.20 pertenece a una rama estable sustituida por la 1.22 en 2022. Ninguna versión tiene soporte vigente. | D2 | No | `evidencias/E2_servidor.txt` |
| E3 | WSTG-INFO-03 | robots.txt pide no indexar 4 rutas: /admin/, /respaldos/, /resultados/ y /wp-content/plugins/wp-file-manager/. Con ello le indica a un atacante dónde mirar. security.txt devuelve HTTP 404, es decir, no existe canal de reporte. | — | — | `evidencias/E3_metarchivos.txt` |
| E4 | WSTG-INFO-05 | Se hallaron 4 comentarios en el código, 3 con información interna: el usuario admin_pruebas pendiente de eliminar, la dirección interna 10.20.0.15:8090 del sistema de laboratorio y el patrón ver.php?orden=NNNNN de los resultados. La etiqueta generator revela WordPress 5.8.1. La contraseña inicial del paciente es su fecha de nacimiento (ddmmaaaa). | D8 | No | `evidencias/E4_contenido.txt` |
| E5 | WSTG-INFO-08 | Se hallaron 5 componentes con versión visible: WordPress 5.8.1, tema medicalpro 2.1.0, Contact Form 7 5.3.1, jQuery 3.4.1 y wp-file-manager 6.8. Este último tiene el CVE-2020-25213, de ejecución remota de código sin autenticación (corregido en la 6.9). No hay señal de actualización mensual. | D3 | No | `evidencias/E5_componentes.txt` |
| E6 | WSTG-INFO-06 | Se hallaron 4 formularios, todos con action en http://, sin HTTPS. Los 4 piden datos personales (DNI, nombres, correo o credenciales). El de citas pide además datos de salud (motivo_consulta y especialidad como Psiquiatría) y los envía por el método GET. El de postulaciones recibe archivos (CV). La casilla acepto no tiene texto. | D7 | No | `evidencias/E6_entradas.txt` |
| E7 | WSTG-CONF-04 · WSTG-CONF-05 | /respaldos/ lista 4 archivos sin autenticación: 1 CSV de citas (citas_export_2026-08.csv), 2 volcados SQL comprimidos (2025-09-30 y 2026-03-31) y 1 copia de configuración (wp-config.php.bak). /admin/ responde HTTP 200 desde fuera de la red de WebSur. No se descargó ningún archivo. | D5 · D6 | No | `evidencias/E7_expuestos.txt` |
| E8 | WSTG-CONF-07 · WSTG-CONF-12 · WSTG-SESS-02 | Faltan 4 de 4 cabeceras de seguridad (HSTS, CSP, X-Frame-Options y X-Content-Type-Options). La cookie PHPSESSID no tiene Secure, HttpOnly ni SameSite (faltan 3 de 3). HTTPS en el puerto 443 devuelve HTTP 000: no hay nada escuchando. | D4 | No | `evidencias/E8_cabeceras.txt` |

Ninguna de las ocho declaraciones del proveedor se sostiene: D1 a D8 resultaron contradichas por la evidencia, y solo E3 carecía de declaración. La comisión no depositará confianza en el inventario de WebSur y fundará el plan solo en lo observado. El hallazgo más ligado al incidente del 08/09/2026 es la dirección ver.php?orden=NNNNN de E4, que permite construir direcciones de resultados, sumada a la contraseña inicial basada en la fecha de nacimiento y a la API con el endpoint /v1/resultados de E1.

## 2. Materia de control · Sección III

La materia de control es la seguridad de los datos de salud en el canal digital de la Clínica Santa Lucía, formado por el portal, el entorno de pruebas y la API de citas, durante el periodo del 01/01/2026 al 30/09/2026. Se elige porque el reconocimiento halló 3 aplicaciones frente a 1 declarada (E1), 4 respaldos listados sin autenticación (E7) y 4 formularios sin HTTPS (E6), y porque el 08/09/2026 una paciente vio el hemograma de otra persona sin que WebSur emitiera informe.

## 3. Alcance · Sección IV

| Bloque | Contenido |
|---|---|
| Incluye | La auditoría cubrirá tres activos, y no solo el declarado. Primero, el portal institucional en 127.0.0.1:8080 (portal.clinica-santalucia.test), con sus rutas /citas/, /resultados/, /postula/, /admin/ y /respaldos/, que atiende 3 850 citas al mes, 12 400 pacientes con cuenta y 214 postulaciones con CV en 2026. Segundo, el entorno de pruebas en 127.0.0.1:8081, que contiene una copia de producción. Tercero, la API de citas en 127.0.0.1:8088, que declara estar en producción y expone datos de las mismas poblaciones. |
| Periodo auditado | El periodo auditado irá del 01/01/2026 al 30/09/2026, el que fijó la Gerencia General en su solicitud. Se elige porque abarca el incidente del 08/09/2026, la restauración de la copia de producción en el entorno de pruebas del 12/09/2026 y los respaldos de marzo y agosto de 2026, todos relacionados con la protección de los datos de salud de los pacientes. |
| Excluye expresamente | Quedan excluidos el sistema de laboratorio interno (10.20.0.15:8090), porque su dirección está fuera de 127.0.0.1, la única autorizada; la infraestructura física y el alojamiento de WebSur, porque no son del canal; la ingeniería social y la denegación de servicio, porque no están autorizadas; y la historia clínica electrónica, por estar fuera del canal digital. Tampoco se aplicará fuerza bruta ni escaneo activo hasta que la Gerencia apruebe el plan. |
| Limitaciones inherentes | La auditoría dará seguridad razonable, no absoluta. No se podrá verificar la eficacia de los controles internos de WebSur, porque el contrato no define quién responde por seguridad y respaldos. Tampoco se podrá probar la contraseña de ninguna cuenta real ni abrir los respaldos, pues están prohibidos. Las pruebas de la Unidad III se harán sobre una réplica autorizada, de modo que el comportamiento de producción podría diferir. |

## 4. Objetivos · Sección V

**Objetivo general.** Determinar si el canal digital de la Clínica Santa Lucía protege los datos de salud de sus pacientes durante el periodo del 01/01/2026 al 30/09/2026, conforme a OWASP WSTG 4.2, OWASP ASVS 4.0.3, la NTP-ISO/IEC 27001:2022 y la Ley 29733.

| N.º | Objetivo específico | Pregunta de auditoría |
|---|---|---|
| OE1 | Determinar si el canal digital expone activos, respaldos, paneles o componentes sin soporte que el proveedor no declaró, conforme a WSTG-CONF-02 y WSTG-CONF-04, ASVS V14 y A.8.8 y A.8.13. | ¿Existen activos, archivos de respaldo o componentes vulnerables accesibles desde internet que expongan datos de pacientes? |
| OE2 | Determinar si el portal impide que una cuenta consulte el resultado de laboratorio de otro titular, conforme a WSTG-ATHZ-04 y WSTG-ATHN-07, ASVS V4.2 y V2.1 y A.8.3 y A.8.5. Este objetivo responde al incidente del 08/09/2026. | ¿Puede una cuenta acceder al resultado de otro paciente y qué control lo impide o debería impedirlo? |
| OE3 | Evaluar si los datos personales y de salud se protegen en tránsito y en la sesión, y si el consentimiento es válido, conforme a WSTG-CRYP-03 y WSTG-SESS-02, ASVS V9.1 y V3.4, A.8.24 y A.8.5 y los artículos 5 y 13.6 de la Ley 29733. | ¿Viajan cifrados los datos de salud, está protegida la sesión del paciente y existe un consentimiento escrito y legible? |

## 5. Enfoque, materialidad y riesgo · Sección VI

| Elemento | Decisión | Sustento en lo que observó o en la solicitud |
|---|---|---|
| Enfoque | Sustantivo | El enfoque será sustantivo. El cuadro 1 muestra que los controles declarados por el proveedor no existen: faltan las 4 cabeceras de seguridad y no hay HTTPS (E8), /admin/ responde HTTP 200 y /respaldos/ lista 4 archivos (E7). Un control inexistente no admite prueba de eficacia operativa, por lo que la comisión no evaluará su funcionamiento en el tiempo. Probará directamente las respuestas del sistema y los datos expuestos en la réplica autorizada. |
| Materialidad cuantitativa | S/ 186 000 (1 % de los ingresos 2025 de S/ 18 600 000) | La materialidad cuantitativa será de S/ 186 000, el 1 % de los ingresos de 2025 de S/ 18 600 000, que se toman como base por ser la cifra anual declarada por la clínica. La cualitativa es independiente del monto: será material toda exposición de datos de salud de un solo paciente, porque la Ley 29733 los define como datos sensibles (artículo 2, numeral 5) y exige medidas de seguridad (artículo 9) y consentimiento escrito (artículo 13, numeral 6). |
| Materialidad cualitativa | Toda exposición de datos de salud de un solo paciente | Ley 29733, art. 2.5, art. 9 y art. 13.6. Véase el párrafo de materialidad. |
| Riesgo inherente | Alto | Datos sensibles de 12 400 pacientes e incidente del 08/09/2026. |
| Riesgo de control | Alto | 0 de 4 cabeceras (E8), /respaldos/ con 4 archivos y /admin/ HTTP 200 (E7), contraseña inicial predecible (E4). |
| Riesgo de detección | Bajo | Se revisará el 100 % de las rutas, respaldos y formularios hallados, no una muestra. |

## 6. Procedimientos · Sección VII

El programa tendrá seis procedimientos, de P-01 a P-06. P-03 y P-04 cubren el objetivo específico 1; P-01 y P-02 cubren el objetivo específico 2, y responden al incidente del resultado de laboratorio ajeno; P-05 y P-06 cubren el objetivo específico 3. Usan cinco categorías WSTG distintas de INFO: ATHZ, ATHN, CONF, CRYP y SESS. Se ejecutarán en la Unidad III sobre la réplica autorizada, con el plan ya aprobado por la Gerencia General.

### P-01 · Acceso a resultados de laboratorio de otro paciente

| Columna | Contenido |
|---|---|
| Objetivo específico | OE2 |
| Origen en el reconocimiento | E4 · el patrón ver.php?orden=NNNNN permite construir direcciones de resultados · E1 · la API del puerto 8088 expone /v1/resultados · incidente del 08/09/2026. |
| Riesgo | R1 · un paciente accede al resultado de otro titular. |
| Criterio | WSTG-ATHZ-04 · OWASP ASVS 4.0.3 V4.2 y V8.2 · NTP-ISO/IEC 27001:2022 A.8.3 · Ley 29733 art. 9 |
| Tipo de prueba | Sustantiva, de autorización. |
| Población y muestra | Población: los números de orden de la réplica. Muestra: 3 cuentas de prueba creadas por la clínica y 30 órdenes consecutivas cercanas a las propias de cada cuenta. |
| Procedimiento | 1. Solicite a la jefa de Sistemas tres cuentas de prueba en la réplica, cada una con una orden propia.<br>2. Inicie sesión con la cuenta A y consulte su orden mediante ver.php?orden=NNNNN. Registre el número.<br>3. Cambie el número por 10 órdenes consecutivas que pertenezcan a la cuenta B y registre si el sistema muestra el resultado.<br>4. Repita la consulta con la cuenta C y con una sesión sin autenticar.<br>5. Revise en cada respuesta las cabeceras Cache-Control, Pragma y Expires.<br>6. Consulte /v1/resultados en la API de la réplica con la cuenta A y verifique si devuelve datos de otros titulares.<br>7. Cuente las órdenes ajenas visibles y compárelas con la definición de excepción. Registre cada una por su número, sin copiar datos del paciente. |
| Evidencia a obtener | Capturas de las solicitudes y respuestas, la lista de órdenes consultadas por cada cuenta y las cabeceras de caché de cada respuesta. |
| Definición de excepción | Toda orden de laboratorio consultable con una cuenta distinta a la del titular, y toda respuesta con resultados sin Cache-Control: no-store o private. |
| Horas estimadas | 8 |

### P-02 · Contraseña inicial y cuentas de prueba

| Columna | Contenido |
|---|---|
| Objetivo específico | OE2 |
| Origen en el reconocimiento | E4 · la contraseña inicial es la fecha de nacimiento (ddmmaaaa) · el comentario menciona el usuario admin_pruebas pendiente de eliminar. |
| Riesgo | R1 · acceso indebido a cuentas de pacientes y del entorno de pruebas. |
| Criterio | WSTG-ATHN-07 · OWASP ASVS 4.0.3 V2.1 · NTP-ISO/IEC 27001:2022 A.8.5 · Ley 29733 art. 9 |
| Tipo de prueba | Sustantiva, de autenticación, sobre la política y las cuentas de la réplica. |
| Población y muestra | Población: 12 400 cuentas de pacientes. Muestra: 25 cuentas de la réplica y las cuentas administrativas de los puertos 8080 y 8081. |
| Procedimiento | 1. Obtenga de la jefa de Sistemas el procedimiento de alta de pacientes y la política de contraseñas vigente.<br>2. Seleccione 25 cuentas de la réplica con un muestreo aleatorio y registre su fecha de creación.<br>3. Verifique en cada una si la contraseña inicial coincide con la fecha de nacimiento y si el sistema obliga a cambiarla.<br>4. Pruebe en la réplica la longitud mínima que acepta el formulario de cambio de contraseña.<br>5. Solicite la lista de cuentas administrativas y compruebe si admin_pruebas existe en cada entorno.<br>6. Compare los resultados con la definición de excepción y registre cada caso. |
| Evidencia a obtener | Texto de la política de contraseñas, lista de cuentas revisadas con su fecha de creación y capturas de las pantallas de alta y de primer acceso. |
| Definición de excepción | Toda cuenta cuya contraseña inicial se derive de un dato del titular, que no obligue a cambiarla en el primer acceso o que acepte menos de 12 caracteres. |
| Horas estimadas | 4 |

### P-03 · Respaldos y panel de administración expuestos

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E7 · /respaldos/ lista 4 archivos y /admin/ responde HTTP 200 · E3 · robots.txt revela ambas rutas. |
| Riesgo | R2 · fuga de datos de pacientes y de credenciales por respaldos y panel accesibles. |
| Criterio | WSTG-CONF-04 y WSTG-CONF-05 · OWASP ASVS 4.0.3 V14 · NTP-ISO/IEC 27001:2022 A.8.13 · Ley 29733 art. 9 |
| Tipo de prueba | Sustantiva, de configuración. |
| Población y muestra | Población: 4 archivos listados y la ruta /admin/. Muestra: el 100 % de ambos. |
| Procedimiento | 1. Solicite a WebSur el procedimiento de respaldos y la ubicación real de cada copia.<br>2. Consulte el listado de /respaldos/ en la réplica y compare cada archivo con el inventario recibido.<br>3. Verifique sin descargar el contenido si cada archivo se sirve sin autenticar, con una petición HEAD.<br>4. Consulte /admin/ desde la red interna y desde una red externa, y registre el código HTTP de cada una.<br>5. Verifique si /admin/ restringe por dirección IP, por segundo factor o por ambos.<br>6. Registre como excepción cada archivo o respuesta que cumpla la definición. |
| Evidencia a obtener | Listado de /respaldos/ con fecha, capturas de la respuesta de /admin/ desde dos redes distintas y el inventario de respaldos de WebSur. |
| Definición de excepción | Todo archivo de respaldo o configuración accesible por HTTP sin autenticación, y todo acceso a /admin/ que responda desde una dirección ajena a la red interna de WebSur. |
| Horas estimadas | 3 |

### P-04 · Entornos no declarados y componentes sin soporte

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E1 · 3 aplicaciones frente a 1 declarada, con una copia de producción en pruebas · E2 y E5 · PHP 7.4.3, nginx 1.20.2, WordPress 5.8.1 y wp-file-manager 6.8 con el CVE-2020-25213. |
| Riesgo | R3 y R4 · explotación de componentes conocidos y datos reales en un entorno de pruebas. |
| Criterio | WSTG-CONF-02 · OWASP ASVS 4.0.3 V14.2 · NTP-ISO/IEC 27001:2022 A.8.8 y A.8.33 · Ley 29733 art. 9 |
| Tipo de prueba | Sustantiva, de configuración y de inventario. |
| Población y muestra | Población: 3 aplicaciones y 5 componentes con versión visible. Muestra: el 100 %. |
| Procedimiento | 1. Elabore una tabla con cada versión de E2 y E5 y consulte su fecha de fin de soporte en la fuente del fabricante.<br>2. Consulte en la base del NIST (NVD) los CVE de cada componente y anote su código y su puntaje CVSS.<br>3. Verifique si el CVE-2020-25213 afecta a wp-file-manager 6.8 en la réplica, sin ejecutar ningún exploit.<br>4. Solicite a la jefa de Sistemas la lista de tablas del entorno 8081 y compare cuáles contienen datos reales.<br>5. Consulte con WebSur la razón de que la API del puerto 8088 se declare en producción.<br>6. Registre como excepción cada caso que cumpla la definición. |
| Evidencia a obtener | Tabla de versiones con su fecha de fin de soporte, los CVE consultados con su fuente y la lista de tablas del entorno de pruebas con datos reales. |
| Definición de excepción | Todo componente sin soporte del fabricante o con un CVE de severidad CVSS 7,0 o más, y todo entorno no productivo que contenga datos reales de pacientes. |
| Horas estimadas | 6 |

### P-05 · Cifrado en tránsito y envío de datos de salud

| Columna | Contenido |
|---|---|
| Objetivo específico | OE3 |
| Origen en el reconocimiento | E6 · 4 formularios con action http://, y el de citas envía datos de salud por GET con una casilla acepto sin texto · E8 · HTTPS sin servicio en el puerto 443. |
| Riesgo | R5 · interceptación de datos de salud y consentimiento inválido. |
| Criterio | WSTG-CRYP-03 · OWASP ASVS 4.0.3 V9.1 · NTP-ISO/IEC 27001:2022 A.8.24 · Ley 29733 arts. 5 y 13.6 |
| Tipo de prueba | Sustantiva, de cifrado y de legalidad. |
| Población y muestra | Población: 4 formularios y los 3 servicios de la clínica. Muestra: el 100 %. |
| Procedimiento | 1. Liste los 4 formularios y registre el método y el protocolo de su action.<br>2. Intente cargar cada ruta por HTTPS en la réplica y registre si hay certificado y qué versión de TLS usa.<br>3. Capture el tráfico de una cita de prueba y verifique si nombres, DNI y motivo de consulta viajan en la URL.<br>4. Verifique si el servidor redirige de HTTP a HTTPS y si envía la cabecera Strict-Transport-Security.<br>5. Registre el texto asociado a la casilla acepto y compárelo con el artículo 13.6 de la Ley 29733.<br>6. Compare cada resultado con la definición de excepción. |
| Evidencia a obtener | Capturas del tráfico de cada formulario en la réplica, la configuración TLS, el texto de la casilla acepto y la política de privacidad vigente. |
| Definición de excepción | Todo formulario o ruta que envíe o reciba datos personales por HTTP, todo dato de salud enviado por GET, y toda casilla de consentimiento sin texto escrito. |
| Horas estimadas | 5 |

### P-06 · Cookie de sesión y cabeceras de seguridad

| Columna | Contenido |
|---|---|
| Objetivo específico | OE3 |
| Origen en el reconocimiento | E8 · la cookie PHPSESSID no tiene Secure, HttpOnly ni SameSite y faltan las 4 cabeceras de seguridad. |
| Riesgo | R5 · robo o fijación de la sesión del paciente. |
| Criterio | WSTG-SESS-02 · OWASP ASVS 4.0.3 V3.4 · NTP-ISO/IEC 27001:2022 A.8.5 · Ley 29733 art. 9 |
| Tipo de prueba | Sustantiva, de sesión. |
| Población y muestra | Población: las cookies y cabeceras de /, /citas/, /resultados/, /postula/ y /admin/. Muestra: el 100 % de las 5 rutas. |
| Procedimiento | 1. Consulte con curl -I cada una de las 5 rutas de la réplica y guarde las cabeceras.<br>2. Inicie sesión con una cuenta de prueba y registre los atributos de la cookie PHPSESSID.<br>3. Verifique si el identificador de sesión cambia tras iniciar sesión.<br>4. Cierre la sesión y confirme que la cookie se invalida en el servidor.<br>5. Cuente las cabeceras ausentes por ruta y compárelas con la definición de excepción. |
| Evidencia a obtener | Cabeceras de respuesta de cada ruta antes y después de iniciar sesión, y capturas de la cookie en el navegador. |
| Definición de excepción | Toda cookie de sesión que carezca de Secure, HttpOnly o SameSite, y toda ruta sin alguna de las 4 cabeceras de seguridad. |
| Horas estimadas | 3 |

## 7. Anexo A · Matriz de riesgos

| N.º | Riesgo | Causa, con su evidencia | Impacto | Nivel | Control existente | Control esperado |
|---|---|---|---|---|---|---|
| R1 | Acceso a resultados de laboratorio de otro paciente | Dirección de resultados predecible (ver.php?orden=NNNNN) y contraseña inicial igual a la fecha de nacimiento (E4); API con /v1/resultados (E1). | Divulgación de datos sensibles, sanción de la Autoridad de Datos Personales y daño reputacional. | Alto | Ninguno evidenciado; WebSur cerró el incidente sin informe. | Autorización por titular en cada consulta, identificadores no predecibles y caché desactivada. |
| R2 | Fuga de datos por respaldos y panel de administración expuestos | /respaldos/ lista 4 archivos sin autenticación y /admin/ responde HTTP 200 (E7); robots.txt los revela (E3). | Pérdida de datos de pacientes y de credenciales de base de datos. | Alto | Ninguno; D5 y D6 no se sostienen. | Respaldos fuera del servidor web y panel solo desde la red interna. |
| R3 | Explotación de componentes sin soporte | PHP 7.4.3 y nginx 1.20.2 sin soporte (E2); WordPress 5.8.1 y wp-file-manager 6.8 con el CVE-2020-25213 (E5). | Toma de control del portal y acceso a la base de datos. | Alto | Ninguno evidenciado; D3 no se sostiene. | Gestión de parches mensual y retiro de complementos innecesarios. |
| R4 | Datos reales y activos fuera de inventario | 3 aplicaciones frente a 1 declarada, con copia de producción en pruebas (E1) y usuario admin_pruebas (E4); el contrato no define la responsabilidad de seguridad. | Filtración desde un entorno sin controles y responsabilidad indefinida con el proveedor. | Medio | Ninguno; D1 no se sostiene. | Inventario único, datos enmascarados en pruebas y cláusulas de seguridad en el contrato. |
| R5 | Datos de salud sin cifrar y consentimiento inválido | 4 formularios con action http://, citas por GET y casilla acepto sin texto (E6); 0 de 4 cabeceras, cookie sin atributos y sin HTTPS (E8). | Interceptación de datos sensibles e incumplimiento de los artículos 5 y 13.6 de la Ley 29733. | Alto | Ninguno; D4 no se sostiene. | HTTPS obligatorio con HSTS, envío por POST y consentimiento escrito. |
