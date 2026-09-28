# Writeup
Portswigger Authentication

# Informe de Auditoría Técnica: Autenticación
**Autor:** Aritz Elizondo Aramburu  
**Laboratorio:** PortSwigger Web Security Academy  
**Herramientas:** VM Kali Linux, Burp Suite Community, Navegador Integrado.

---

## 1. Metodología de Auditoría
Se ha seguido el orden oficial de PortSwigger para analizar los diferentes vectores de ataque en sistemas de autenticación. Para cada hallazgo se documenta el objetivo, el proceso en Burp Suite y la solución recomendada.

---

## 2. Bloque 1: Fuerza Bruta y Enumeración
### Lab 1: Username enumeration via different responses

***Objetivo:** Identificar un nombre de usuario válido analizando las discrepancias en los mensajes de error del servidor y comprometer la cuenta mediante fuerza bruta de contraseñas.

***Proceso:** 1. Interceptar una petición `POST /login` en el Proxy de Burp Suite tras realizar un intento de inicio de sesión fallido con credenciales de prueba.
2. Enviar dicha petición al módulo **Intruder** mediante el atajo `Ctrl + I`.
3. **Fase 1 (Enumeración de usuarios):** Configurar el tipo de ataque en **Sniper** y definir el marcador de posición únicamente en el parámetro del nombre de usuario (`username=§test§`). En la pestaña *Payloads*, cargar la lista de nombres de usuario proporcionada por el laboratorio y lanzar el ataque. Analizar los resultados ordenando por la columna *Length* (longitud); la fila correspondiente al usuario válido (`academico`) mostrará un tamaño diferente debido a una respuesta de error distinta ("Incorrect password" en lugar de "Invalid username").
4. **Fase 2 (Fuerza bruta de contraseñas):** Regresar a las posiciones del Intruder, sustituir el valor del usuario por el nombre válido ya descubierto de forma fija (`username=academico`) y colocar el marcador de posición únicamente en el parámetro de la contraseña (`password=§test§`). Cargar la lista de contraseñas en la configuración del payload y lanzar un nuevo ataque en modo **Sniper**. Identificar la contraseña correcta (`access`) observando un cambio en la longitud de la respuesta o un código de estado de redirección HTTP (`302 Found`).
* **Evidencia:** > 
* "Interceptación de la solicitud de autenticación POST /login y transferencia de la estructura HTTP al módulo Intruder para iniciar la parametrización de vectores de ataque."
<img width="1366" height="720" alt="image70" src="https://github.com/user-attachments/assets/dd3470f6-360f-4e4b-baff-1aae37bf97a8" />



* "Configuración del marcador de posición de tipo dinámico exclusivamente sobre el parámetro username utilizando el modo de ataque Sniper."
<img width="1366" height="720" alt="image80" src="https://github.com/user-attachments/assets/f4f392db-26e7-401e-856f-23f0e991a000" />








* "Carga exhaustiva del diccionario de usuarios predefinido en la sección de payloads del Intruder previo a la ejecución de las consultas dirigidas."
<img width="1366" height="720" alt="image48" src="https://github.com/user-attachments/assets/adc3b6f0-ce67-4660-979f-8fe6507e6dad" />






* "Identificación de un comportamiento anómalo en el servidor reflejado en una longitud de respuesta HTTP diferenciada (Length: 3214) para el payload 'academico', aislando el usuario objetivo debido a la generación de un mensaje de error específico por parte del backend."
<img width="1366" height="720" alt="image64" src="https://github.com/user-attachments/assets/13034605-e8fc-4467-9f2a-373d29c7f5d1" />



"Ejecución de la segunda fase del ataque fijando estáticamente el usuario válido identificado y parametrizando dinámicamente el campo de la contraseña."
<img width="1366" height="720" alt="image66" src="https://github.com/user-attachments/assets/cdb3f845-05f7-46a4-aa46-1c101082d975" />



* "Ejecución de la segunda fase del ataque fijando estáticamente el usuario válido identificado y parametrizando dinámicamente el campo de la contraseña."
<img width="1366" height="720" alt="image37" src="https://github.com/user-attachments/assets/b49167aa-7cf3-41fc-a723-f494a5be069a" />









* "Análisis final del ataque de fuerza bruta donde se aísla la credencial válida ('access') mediante la detección de un código de estado de redirección HTTP 302 Found, confirmando la validación del inicio de sesión."
<img width="1366" height="720" alt="image3" src="https://github.com/user-attachments/assets/556640cc-cf5e-440b-8725-4340bc7d373e" />


* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales compromised durante la auditoría."
<img width="1366" height="720" alt="image14" src="https://github.com/user-attachments/assets/ee55f9da-5fa3-45e6-88f3-fb40b1def9f2" />


* **Mitigación:** El servidor debe devolver un mensaje genérico como "Usuario o contraseña incorrectos" con la misma longitud de respuesta para no dar pistas al atacante.
—
### Lab 2: Username enumeration via subtly different responses
***Objetivo:** Identificar un nombre de usuario válido analizando las discrepancias sutiles en las respuestas del servidor (*subtly different responses*) y eludir las restricciones de bloqueo temporal de IP por tasa de intentos (*Rate-Limiting*) mediante la alternancia estratégica de solicitudes.

***Proceso:** 1. Interceptar la petición `POST /login` con el módulo **Proxy** de Burp Suite y enviarla al **Intruder** (`Ctrl + I`), seleccionando la modalidad de ataque **Pitchfork**.
2. En la pestaña **Positions**, definir dos marcadores de posición simultáneos: el primero en el parámetro del cuerpo destinado al usuario (`username=§test§`) y el segundo en el parámetro de la clave (`password=§test§`).
3. **Payload 1 (Estrategia de Usuarios):** Configurar el primer set de payloads cargando una lista modificada que intercala de manera secuencial una solicitud con el usuario legítimo de control (`wiener`) seguido de un nombre de usuario del diccionario de víctimas.
4. **Payload 2 (Estrategia de Claves):** Configurar el segundo set de payloads de forma emparejada, alternando la contraseña válida conocida (`peter`) para reiniciar el contador de intentos fallidos del servidor, y una contraseña estática larga para los intentos de enumeración. Con esta estructura se previene el bloqueo temporal sin requerir manipulación de cabeceras de red.
5. Lanzar el ataque y evaluar las respuestas ordenando por la columna *Length* o analizando el contenido de los paquetes. La fila que presente una diferencia sutil en el tamaño o texto de la respuesta revelará el usuario válido del sistema.
6. **Segunda Fase (Fuerza Bruta):** Regresar a la pestaña **Positions**, fijar el usuario descubierto como un valor estático en la petición y trasladar el marcador exclusivamente al campo `password=§test§`, utilizando el modo **Sniper** junto con el diccionario completo de contraseñas para comprometer la cuenta tras identificar el código **HTTP 302 Found**.

Your credentials: wiener:peter
* **Evidencia:** > 


* "Evidencia de mensaje de error 'Invalid username or password'. Uso de Grep - Match incluyendo el mensaje de error para la visualización en el resultado del ataque."



* "Evidencia de ataque exitoso; la columna de advertencia (Warning) refleja una anomalía que expone el usuario válido."







* "Evidencia de segundo ataque Sniper incluyendo el usuario legítimo y la carga de la lista de contraseñas proporcionada."





* "Evidencia de ataque exitoso; la columna de advertencia indica la contraseña válida."







* "Interceptación de petición POST /login, uso de Intruder para la parametrización del usuario mediante Sniper y la carga de la lista de usuarios proporcionada."


* **Mitigación:** Normalizar de manera estricta todos los mensajes de error devueltos por los componentes de autenticación en el backend, implementando validaciones y filtros globales que depuren de forma automática cualquier discrepancia tipográfica, errata de caracteres o espacio en blanco residual (`" "`) que pudiera actuar como un oráculo de información ante un análisis comparativo de respuestas.

—

### Lab 3: Username enumeration via response timing (Con Bypass de IP)
***Objetivo:** Identificar un nombre de usuario válido analizando el tiempo de respuesta del servidor y evadiendo el bloqueo perimetral por IP.

***Proceso:** 1. Interceptar la petición `POST /login` con Burp Suite y enviarla directamente al **Intruder**, cambiando el tipo de ataque a **Pitchfork**.
2. Modificar la petición dentro del **Intruder** añadiendo manualmente la cabecera `X-Forwarded-For: §1§` en las cabeceras HTTP, marcar el parámetro `username=§2§` y escribir una contraseña fija larga.
3. **Payload 1 (IP Spoofing):** Seleccionar el tipo de payload *Numbers*, configurar un rango secuencial del 1 al 105 (con paso 1 y sin decimales) para simular un origen distinto en cada petición y evitar el bloqueo.
4. **Payload 2 (Usuarios):** Cargar el diccionario de usuarios proporcionado por PortSwigger, iniciar el ataque desde el **Intruder** y analizar los resultados ordenando de mayor a menor por la columna *Response received*.
5. Reconfiguración del **Intruder:** Regresar a la pestaña **Positions**. Mantener el marcador de la cabecera `X-Forwarded-For: §1§`, sustituir el marcador de usuario por el nombre de usuario válido descubierto (fijo y sin símbolos `§`) y añadir los marcadores al parámetro de la contraseña de la forma `password=§2§`.
6. **Payload 1 (Nuevo Rango de IP):** Mantener el tipo de payload *Numbers*, pero incrementar el rango secuencial (por ejemplo, del 106 al 210) para asegurar que se utilicen identificadores de IP limpios que no hayan sido bloqueados en la fase anterior.
7. **Payload 2 (Contraseñas):** Cambiar el tipo de payload a *Simple list*, limpiar el diccionario anterior y cargar el listado de contraseñas proporcionado por PortSwigger.
8. Análisis y Resolución: Lanzar el ataque y evaluar los resultados ordenando por la columna *Status* o *Length*. Identificar la petición que devuelve un código `302 Found` (o una longitud de respuesta diferente), iniciar sesión en la aplicación web con las credenciales válidas y completar el laboratorio.

* **Evidencia:**















Your credentials: wiener:peter
* **Evidencia:** > 
* "Establecimiento de una línea base de tiempo de respuesta (61 ms) en entornos de autenticación exitosa mediante solicitudes controladas en el módulo Repeater."





* "Serie de pruebas realizadas de tiempo de respuesta: 1. Usuario y contraseña aleatoria: 92 ms. 2. Usuario y contraseña larga: 85 ms. 3. Wiener y contraseña larga: 92 ms."
























* "Configuración avanzada del ataque en modalidad Pitchfork inyectando dinámicamente la cabecera de enrutamiento de red X-Forwarded-For: §IP§. Esto neutraliza la defensa de bloqueo al simular peticiones concurrentes provenientes de un entorno distribuido de clientes de red."







* "Evidencia de comportamiento de la página ante un usuario existente con una contraseña larga inexistente (1880 ms)."



* "Parametrización de solicitudes y selección de modo: se muestra la pestaña Positions del módulo Intruder durante la preparación del vector de ataque. Se han definido de manera estratégica dos marcadores de posición (§): el primero en el campo del parámetro de autenticación (username) y el segundo en la cabecera HTTP inyectada (X-Forwarded-For). Se selecciona la modalidad de ataque Pitchfork para permitir la ejecución simultánea de ambos diccionarios en una única iteración."

* "Carga del primer vector de payloads (Bypass de Red): evidencia de la configuración del Payload set 1. Se procede a cargar la lista parametrizada de direcciones IP simuladas que alimentarán la cabecera X-Forwarded-For. Esta configuración asegura que Burp Suite asocie un origen de red único por cada solicitud enviada, neutralizando la contramedida perimetral de bloqueo por IP."

* "Carga del segundo vector de payloads: aislamiento del usuario legítimo 'vagrant' ordenando los resultados de forma descendente por tiempo de procesamiento de respuesta, evidenciando un retraso computacional severo en el backend al validar un usuario existente."















* "Se incluye el nombre de usuario legítimo (en este caso, 'vagrant') y se parametriza el campo de la contraseña, cargando el diccionario proporcionado por el laboratorio en el Payload set 2 para iniciar el ataque de fuerza bruta."



“Se incluye el nombre de usuario legitimo (en este caso, vagrant) y se parametriza el campo de la contraseña, cargando el diccionario proporcionado por el laboratorio en el Payload set 2 para iniciar el ataque de fuerza bruta."






* "Resultado final del ataque Pitchfork en el que se identifica la clave 'soccer' debido a la obtención de un código de estado de sesión satisfactoria (HTTP 302 Found), evadiendo por completo la restricción perimetral de la IP."




* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales comprometidas durante la auditoría."


* **Mitigación:** No utilizar cabeceras fácilmente manipulables por el cliente como `X-Forwarded-For` para aplicar restricciones de seguridad (Rate Limiting).
—
### Lab 4: Broken brute-force protection, IP block
* **Objetivo:** Sortear el bloqueo perimetral de IP intercalando de forma estricta intentos de fuerza bruta dirigidos al usuario objetivo con inicios de sesión legítimos utilizando credenciales propias para resetear el contador del servidor.

* **Proceso:** 1. Interceptar una petición `POST /login` en el Proxy tras realizar un intento de autenticación y enviarla al **Intruder**.
2. Configurar el tipo de ataque en **Sniper** y colocar el marcador de posición tanto en el parámetro de usuario como en el de contraseña simultáneamente (`username=§victim§&password=§test§`).
3. **Configuración del Diccionario:** Crear un diccionario personalizado donde se intercale de forma matemática una petición hacia la víctima y una petición legítima. El patrón debe ser estricto: por cada intento de probar una contraseña del diccionario contra la víctima, se debe introducir inmediatamente después una línea con las credenciales válidas conocidas.








Your credentials: wiener:peter
Victim's username: carlos
* **Evidencia:** > 

* "Interceptación de petición POST /login con credenciales válidas."




* "Intento de evasión de bloqueos mediante rotación de IP (sin resultado exitoso)."



* "Intento de reseteo del contador mediante validación exitosa: payload el cual realiza un inicio de sesión exitoso cada 2 intentos."










“Ataque realizado sin éxito”





* "Creación de dos archivos de texto (.txt) que contienen la lista combinada de usuarios (víctima y legítimo) y la lista de posibles contraseñas con 'peter' de forma intercalada para reiniciar el contador del backend."




* "Configuración de la herramienta Intruder en modo Pitchfork utilizando el Payload 1 con el archivo de usuarios creado y el Payload 2 con el archivo de contraseñas generado, seguido de la eliminación de cookies para mantener una sesión limpia."





* "Uso de la opción 'Maximum concurrent requests' dentro del 'Resource pool' para forzar el procesamiento secuencial estricto desde la primera línea."








* "Requerimiento técnico de Burp Suite Professional para activar la opción de bucle (Loop), la cual automatiza la extracción exitosa de la clave de la víctima."




* "Ejecución de un intento de bucle manual mediante un archivo estructurado de 200 líneas incluyendo a 'carlos' y 'wiener'."







* "Carga del nuevo archivo plano para la ejecución del ataque."





* "Ataque exitoso con el diccionario generado; se identifica el código de estado HTTP 302 Found en la petición número 191 correspondiente a las credenciales 'carlos:monitor'."



* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales comprometidas durante la auditoría."


* **Mitigación:** Implementar un control de tasa (*Rate-Limiting*) estricto que asocie los intentos fallidos combinando la dirección IP real de origen con el nombre de usuario (`username`), impidiendo el reseteo del contador de bloqueos mediante la intercalación síncrona de inicios de sesión válidos de otras cuentas.

—

### Lab 5: Username enumeration via account lock

* **Objetivo:** Enumerar e identificar un nombre de usuario válido explotando una diferencia en el comportamiento del servidor (longitud de respuesta o mensaje de error) al forzar deliberadamente el bloqueo temporal de una cuenta existente tras recibir múltiples intentos de inicio de sesión fallidos consecutivos.

* **Proceso (Solución Técnica Correcta):** 1. Interceptar la petición `POST /login` en el Proxy de Burp Suite introduciendo credenciales aleatorias y enviarla al **Intruder** (`Ctrl + I`).
2. En la pestaña *Positions*, seleccionar el modo de ataque **Sniper** y configurar el marcador únicamente en el parámetro de usuario: `username=§test§&password=test`.
3. En la pestaña *Payloads*, cargar el diccionario de usuarios. Para poder forzar el bloqueo en la versión Burp Community, se debe cargar el mismo archivo de usuarios un total de 5 veces consecutivas en la lista para simular el umbral de intentos fallidos por cada cuenta antes de pasar a la siguiente.








* **Evidencia:** > 

* "Interceptación de la petición POST /login y creación del payload en modalidad Cluster Bomb."



* "Configuración del Payload 1 cargando la lista de usuarios proporcionada."









* "Configuración del Payload 2 mediante la opción de Null Payloads para repetir los intentos de inicio de sesión."


* "Evidencia de mensaje de error 'Invalid username or password'. Uso de la función Grep - Match incluyendo el mensaje de error para la visualización del estado en el resultado del ataque."



* "Nota de auditoría: el servidor del laboratorio expira de manera programada antes de que la versión Burp Community finalice el ataque completo, requiriendo optimización en el número de hilos concurrentes."


* **Mitigación:** Configurar el sistema de bloqueo de cuentas para que, al superar el umbral de intentos fallidos, responda con el mismo mensaje genérico de error y código de estado que una petición inválida ordinaria, impidiendo la enumeración de usuarios basada en el estado de bloqueo del perfil.
—-
### Lab 6: Broken brute-force protection, multiple credentials per request
* **Objetivo:** Eludir la protección de fuerza bruta enviando múltiples credenciales de forma simultánea en una única solicitud estructurada, minimizando el número de peticiones HTTP enviadas al servidor.

* **Proceso:** 1. Interceptar una petición `POST /login` en el Proxy de Burp Suite tras realizar un intento de inicio de sesión fallido, identificando si el backend acepta estructuras de datos alternativas como JSON.
2. Enviar dicha petición al módulo **Repeater** o **Intruder** para modificar la estructura del cuerpo de la solicitud HTTP.
3. **Fase de Modificación del Payload:** Reemplazar el parámetro de contraseña plano por un arreglo o lista multi-credencial (por ejemplo, `["pass1", "pass2", "pass3"...]`) dirigido contra el usuario objetivo dentro de la sintaxis del mensaje. Lanzar la petición y evaluar si el backend procesa el bloque completo de datos de manera desprotegida.







Victim's username: carlos
* **Evidencia:** > 
* "Modificación de la estructura de la solicitud HTTP debido a que el endpoint de validación múltiple carece de protecciones contra ataques de fuerza bruta concurrentes."





* "Análisis de la respuesta del servidor en segundo plano donde no se reflejan restricciones en la interfaz."









* "Generación de un enlace de sesión para visualizar y validar el estado de la autenticación directamente en el navegador integrado."




* “Link para visualizar en el navegador.”




* "Evidencia de inicio de sesión exitoso en la cuenta del usuario víctima."



* **Mitigación:**  Configurar el analizador de datos (parser) y el validador del endpoint de autenticación para que acepten estrictamente un único par de credenciales por solicitud en formato de cadena de texto (string), rechazando inmediatamente cualquier estructura que contenga arrays o múltiples valores, de modo que el sistema de protección contra fuerza bruta contabilice de forma precisa cada intento de inicio de sesión de manera unívoca. 
—-----------------------------------------------------------------------------------------------------------------------
##  Bloque 3: Multi-Factor Authentication (MFA)

###  Lab 1: 2FA simple bypass
* **Objetivo:** Acceder a la cuenta de la víctima saltándose la pantalla de validación intermedia del código de verificación de 6 dígitos mediante un fallo de control de acceso en la navegación forzada.

* **Proceso:** 1. Iniciar sesión en el formulario principal con las credenciales legítimas (`wiener:peter`) para analizar la estructura de la URL del panel interno tras pasar el MFA. Identificar que la ruta final es `/my-account?id=wiener`.
2. Cerrar sesión e iniciar una nueva autenticación con las credenciales de la víctima (`carlos`).
3. Cuando la página web se redirija a la interfaz de verificación intermedia solicitando el código PIN de seguridad de 6 dígitos, no introducir ningún dato.
4. Ir directamente a la barra de direcciones del navegador, borrar la ruta actual de validación y cambiar la URL manualmente introduciendo el endpoint objetivo modificado: `/my-account?id=carlos`.





Your credentials: wiener:peter
Victim's credentials carlos:montoya
* **Evidencia:** >

* "Buzón de correo del usuario legítimo interceptado durante el análisis de flujo."


* "Inspección en la página del servidor de explotación (Exploit Server) en busca de información técnica relevante."





* "Inicio de sesión inicial empleando el usuario legítimo de control."







* "Evidencia de requerimiento obligatorio del código de autenticación de doble factor por el canal ordinario."





* "Evidencia de funcionamiento correcto del servicio de mensajería de correo."







* "Evidencia de la transición forzada en la URL de `/login` hacia `/login2` durante el proceso de login."






* "Evidencia de redireccionamiento automático hacia la ruta `/my-account?id=wiener` tras una autenticación satisfactoria."


* "Intento de explotación de vulnerabilidad IDOR modificando el identificador del usuario por el de la víctima."



* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales comprometidas durante la auditoría."



* **Mitigación:** Implementar una arquitectura de control de acceso basada en estados secuenciales atómicos en el backend, bloqueando el acceso a directorios protegidos (como `/my-account`) hasta que la sesión refleje explícitamente la verificación y superación correcta del token de doble factor (2FA).

—-
###  Lab 2: 2FA broken logic
* **Objetivo:** Acceder a la cuenta de la víctima explotando un fallo en la lógica de negocio del servidor, el cual permite vincular y validar un código de verificación MFA contra una cuenta de usuario arbitraria sin verificar la identidad real de la sesión que lo solicita.

* **Proceso:** 1. Acceder a la sección de login e introducir las credenciales legítimas de la víctima (`carlos`). El servidor procesará la primera fase y redirigirá a la pantalla de introducción del PIN de 2FA. En este punto, el servidor genera el código y se lo envía a Carlos.
2. Sin rellenar el PIN de Carlos, abrir una nueva pestaña o utilizar el cliente de correo del laboratorio para iniciar sesión con tus propias credenciales (`wiener:peter`). Al hacerlo, el servidor te enviará tu propio código PIN temporal a tu buzón (por ejemplo: `1234`).
3. Interceptar en el Proxy de Burp Suite la petición `POST /login2` al enviar tu código PIN verídico.
4. Modificar el parámetro destinado al identificador de usuario (`username`), sustituyendo tu valor por el de la víctima (`carlos`), y enviar la solicitud para forzar la validación de tu PIN contra su cuenta.



Your credentials: wiener:peter
Victim's username: carlos
* **Evidencia:** >


* “Evidencia de requerimiento de código de autenticación.”





* "Transferencia de la petición POST /login2 al módulo Repeater para su análisis dinámico."




* "Evidencia de aceptación y funcionamiento de un código de autenticación aleatorio por parte del backend."









* "Ejecución de un script personalizado en Python para realizar fuerza bruta sobre el segundo factor de 'carlos', resultando infructuoso debido a la rotación periódica del token cada 2 minutos."




* **Mitigación:** Asociar de forma unívoca el token de verificación multifactor (2FA) al identificador de la cuenta validada estrictamente en el paso previo mediante variables de sesión del lado del servidor, impidiendo que un atacante procese un código legítimo contra el nombre de usuario de una víctima.

—

### Lab3 : 2FA bypass using a brute-force attack
### Lab: 2FA bypass using a brute-force attack

### Lab 3: 2FA bypass using a brute-force attack

* **Objetivo:** Eludir el mecanismo de autenticación de doble factor (2FA) mediante un ataque de fuerza bruta automatizado contra el código de verificación de 4 dígitos, utilizando macros de Burp Suite para mitigar la contramedida de invalidación de sesión tras intentos fallidos.

* **Proceso:** 1. Iniciar sesión con las credenciales iniciales del usuario administrador (`carlos`) e interceptar la solicitud `POST /login2` asociada al formulario de introducción del código de seguridad de 2FA.
2. **Configuración de la Macro de Persistencia:** Acceder a **Settings > Sessions** en Burp Suite. En el apartado de reglas de manejo de sesión (*Session Handling Rules*), añadir una nueva regla con alcance (*Scope*) configurado para "Include all URLs".
3. Definir una acción de regla de tipo **Run a macro** y grabar una secuencia automatizada basada en tres solicitudes consecutivas: `GET /login` para refrescar los tokens de sesión antes de cada intento.



Victim's credentials: carlos:montoya
* **Evidencia:** > 

* "Evidencia de generación del código de autenticación en la interfaz al iniciar sesión."





* "Interceptación de la petición POST /login en el módulo de tránsito (Proxy)."




* "Configuración de la regla de manejo de sesión (Session Handling Rule) aplicando el alcance a todas las URL del laboratorio."






* “Incluir todas las URL.”


* "Asignación de la acción automatizada 'Run a macro' en Burp Suite."




* "Añadido secuencial de las peticiones intermedias `/login` y `/login2` dentro de la macro."





* "Ejecución de un test de diagnóstico sobre la macro configurada con resultado satisfactorio."



* "Registro de auditoría técnico sobre las iteraciones de la macro."



* **Mitigación:** Implementar un bloqueo estricto de tasa (*Rate-Limiting*) que restrinja el número máximo de intentos fallidos (por ejemplo, 3 o 5) en el endpoint de validación del segundo factor (`/login2`), invalidando la sesión de forma inmediata ante cualquier patrón automatizado.

—------------------------------------------------------------------------------------------------------------------------
##  Bloque 4: Other-Authentication-Mechanisms
### Lab 1: Brute-forcing a stay-logged-in cookie
* **Objetivo:** Suplantar la identidad de `carlos` mediante fuerza bruta sobre su cookie de sesión persistente (`stay-logged-in`), explotando un diseño criptográfico predecible y sin aleatoriedad.

* **Proceso de Explotación:** 1. **Análisis e Ingeniería Inversa:**
   * Iniciar sesión con `wiener:peter` marcando la casilla **"Stay logged in"**.
   * Interceptar la petición HTTP en Burp Suite y decodificar la cookie `stay-logged-in` de **Base64**.
   * El resultado en texto plano es `wiener:51dc30dd...`. Se comprueba que la cadena hexadecimal corresponde al hash **MD5** de la contraseña (`peter`).
   * **Estructura identificada:** `Base64( usuario + ':' + MD5(contraseña) )` 
2. **Configuración en Burp Intruder:**
   * Cerrar sesión y enviar una petición `GET /my-account` al módulo **Intruder**.
   * Modificar el parámetro de la URL a `id=carlos`.
   * Configurar la cookie procesada como el parámetro variable para realizar el ataque.

Your credentials: wiener:peter
Victim's username: carlos
* **Evidencia:** > 
* "Interceptación de la petición GET /my-account?id=wiener, exponiendo la cookie de sesión persistente estructurada."






* "Visualización de la cadena codificada en Base64 mediante el panel del Decoder."



* "Uso de una herramienta local de descifrado de hashes (Hash Cracker) logrando revertir exitosamente el MD5."



* "Configuración del módulo Intruder para orquestar un ataque dirigido en modalidad Sniper."







* "Carga del diccionario de contraseñas proporcionado por el laboratorio para alimentar el vector de ataque."




* "Ataque ejecutado con éxito; se identifica la solicitud válida gracias a una longitud de respuesta (Length) diferenciada del resto del conjunto."







* "Evidencia de la falsificación final de la cookie cifrada en Base64 perteneciente a la víctima."



* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales comprometidas durante la auditoría."



* **Mitigación:** Reemplazar el uso de cookies persistentes basadas en hashes predecibles de datos estáticos por identificadores de sesión pseudoaleatorios de alta entropía (*Session Tokens*) de un solo uso, configurando estrictamente sus atributos protectores `HttpOnly`, `Secure` y `SameSite`.

—
### Lab 2: Offline password cracking
* **Objetivo:** Suplantar la identidad del usuario administrador explotando una vulnerabilidad de XSS almacenado para secuestrar su cookie de sesión persistente, descifrando posteriormente su contenido de forma externa.

* **Proceso (Basado en tus evidencias):** 1. **Fase de Análisis:** Iniciar sesión en el portal con las credenciales asignadas (`wiener:peter`). Interceptar la navegación en el Proxy de Burp Suite y analizar la estructura de la cookie `stay-logged-in`. Al decodificarla de Base64, se determina que almacena el formato predecible `usuario:hash_md5(contraseña)`.
2. **Fase de Inyección (Stored XSS):** Dirigirse a la sección de comentarios del blog del laboratorio e inyectar un payload malicioso de JavaScript estructurado para leer las cookies de sesión activa de cualquier usuario que visite dicha entrada.
3. **Fase de Explotación:** Monitorizar el *Access Log* del servidor del laboratorio. Cuando el usuario administrador (`carlos`) visite la entrada comprometida, su navegador enviará de fondo la cookie de sesión persistente hacia tu receptor HTTP externo.




Your credentials: wiener:peter
Victim's username: carlos
* **Evidencia:** > 

* "Navegación e inspección preliminar de las funcionalidades expuestas de la aplicación web."




* "Evidencia de la inicialización del Exploit Server y obtención de su URL única para la recepción de exfiltraciones."








* "Confirmación de presencia de una vulnerabilidad de Cross-Site Scripting (XSS) almacenado en los formularios de comentarios."






* "Análisis del registro de accesos (Access Log) del Exploit Server, donde se expone la captura de las credenciales codificadas en Base64 de la cuenta de la víctima."



* "Uso de una utilidad de descifrado criptográfico (Hash Cracker) identificando con éxito la contraseña en texto plano."




* "Evidencia de inicio de sesión administrativo y acceso al panel de control de borrado de cuentas."




* "Validación del requerimiento de contraseña para la ejecución de acciones críticas en el perfil."




* "Confirmación de la eliminación exitosa del usuario objetivo, cumpliendo los objetivos planteados."


* **Mitigación:** Implementar una función de derivación de claves robusta y computacionalmente costosa (*Key Derivation Function*), como `Argon2id` o `bcrypt`, aplicando un factor de coste elevado y un valor de sal (*salt*) pseudoaleatorio y único por usuario para ralentizar drásticamente cualquier intento de descifrado local de hashes.


—

### Lab 3: Password reset broken logic

* **Objetivo:** Explotar una vulnerabilidad en la lógica de negocio del proceso de restablecimiento de contraseña para cambiar la clave del usuario víctima (`carlos`) sin necesidad de conocer su enlace ni su token de recuperación.

* **Proceso (Solución Técnico-Correcta):** 1. **Fase de Análisis:** Dirigirse a la sección de inicio de sesión y hacer clic en *"Forgot password?"*. Introducir tu propio usuario legítimo (`wiener`) para recibir el enlace de recuperación y analizar la estructura de los parámetros transmitidos en el formulario de cambio de clave.
2. Interceptar la petición de cambio definitivo de contraseña en Burp Suite.
3. Modificar el parámetro `username` en el cuerpo de la petición HTTP, cambiando el valor de `wiener` a `carlos`. Eliminar por completo el parámetro del token temporal (`temp-forgot-password-token`) y enviar la petición desde el módulo **Repeater** para comprobar si el backend omite la validación del token al no estar presente.



Your credentials: wiener:peter
Victim's username: carlos

* **Evidencia:** > 

* "Interceptación de la solicitud HTTP GET /forgot-password."


* "Evidencia de acceso al buzón de correo virtual del usuario para recuperar el flujo de restablecimiento."









* "Evidencia de funcionamiento correcto del módulo de recuperación de credenciales."




* "Captura del token temporal dinámico generado por la aplicación para el cambio de clave."


* "Modificación controlada del parámetro identificador sustituyéndolo por la cuenta de la víctima ('carlos')."







* "Uso del módulo Repeater para la supresión completa del token temporal en la estructura de la solicitud."



* "Evidencia de respuesta HTTP 302 Found emitida por el servidor, confirmando la alteración legítima de la contraseña del usuario víctima."







* **Mitigación:** Validar de manera estricta y obligatoria en el backend que el token de restablecimiento de contraseña adjunto en la solicitud corresponda unívocamente con el nombre de usuario (`username`) que intenta realizar la acción, ignorando o rechazando cualquier parámetro secundario que pretenda alterar la identidad del destinatario.


—

### Lab 4: Password reset poisoning via middleware

* **Objetivo:** Secuestrar el enlace de recuperación de contraseña de `carlos` manipulando cabeceras HTTP para desviar el token de seguridad hacia un servidor controlado por el atacante.

* **Proceso de Explotación:** 1. **Fase de Análisis:**
   * Solicitar una recuperación de contraseña para el usuario propio (`wiener`).
   * Interceptar la petición `POST /forgot-password` en Burp Suite y enviarla al **Repeater**.
   * Añadir la cabecera `X-Forwarded-Host: tu-exploit-server.net`. Enviar la petición y verificar en el cliente de correo que el dominio del enlace ha sido envenenado con éxito.
2. **Fase de Ataque:**
   * En el Repeater, cambiar el valor del parámetro en el cuerpo a `username=carlos`. Asegurarse de que la cabecera `X-Forwarded-Host` apunte a la dirección de tu servidor de explotación y enviar la petición HTTP para dirigir el token de Carlos hacia tus registros.



Your credentials: wiener:peter
Victim's username: carlos
* **Evidencia:** > 

* "Interceptación de la solicitud POST /forgot-password mediante el Proxy."





* "Evidencia del servidor de explotación (Exploit Server) expuesto en la red."






* "Evidencia del registro de accesos (Access Log) donde se monitorizan las peticiones entrantes."




* "Intento de elusión de autenticación mediante análisis dinámico (verificación colateral de vectores como JWT)."








* "Evidencia de acceso al formulario de restablecimiento de contraseña sin credenciales previas."




* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales comprometidas durante la auditoría."



* **Mitigación:** Configurar el backend para que construya los enlaces de restablecimiento utilizando exclusivamente un dominio estático y absoluto definido en un archivo de configuración seguro del servidor, ignorando por completo cabeceras HTTP dinámicas y manipulables como `X-Forwarded-Host`.
—



### Lab 5: Password brute-force via password change

* **Objetivo:** Acceder a la cuenta de `carlos` realizando un ataque de fuerza bruta contra el formulario de cambio de contraseña interna, aprovechando una validación deficiente de la sesión o de las credenciales actuales.

* **Proceso de Explotación:** 1. **Fase de Análisis:**
   * Iniciar sesión con las credenciales de prueba (`wiener:peter`) e ir a la sección de cambio de contraseña.
   * Rellenar el formulario e interceptar la petición `POST /change-password` en Burp Suite.
   * Enviar la solicitud a **Burp Intruder**.






Your credentials: wiener:peter
Victim's username: carlos
* **Evidencia:** > 

* "Interceptación de la petición POST /my-account/change-password."




* "Evidencia del comportamiento de la aplicación web ante un cambio de contraseña exitoso."





* "Evidencia del comportamiento del sistema ante un intento de cambio de contraseña fallido."




* "Carga de un diccionario de contraseñas candidato utilizando la modalidad de ataque Sniper desde el Intruder."



* "Evidencia de la obtención del mensaje de error 'Current password is incorrect'. Uso de la directiva Grep - Extract incluyendo la alerta para su visualización tabular en los resultados del ataque."




* "Evidencia de ataque exitoso detectado mediante el cambio de estado en la columna de advertencias (Warnings)."




* "Acceso legítimo y consolidación de sesión en la interfaz privada de la cuenta corporativa empleando las credenciales comprometidas durante la auditoría."




* **Mitigación:** Exigir siempre la validación obligatoria de la contraseña actual del usuario antes de procesar cualquier solicitud de cambio de clave, e implementar un mecanismo estricto de límite de tasa (*Rate-Limiting*) que bloquee temporalmente las peticiones concurrentes o excesivas hacia dicho endpoint.



—------------------------------------------------------------------------------------------------------------------------




## 5. Conclusiones Generales
Los errores de lógica de negocio son tan peligrosos como los fallos de software. Una autenticación robusta requiere validación constante de la sesión en cada paso del flujo.

---
