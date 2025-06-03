## 📑 Índice de Contenidos
- [❓ ¿Qué es hacer Fuzzing?](#qué-es-hacer-fuzzing)
- [🛠 Herramientas para Fuzzing](#herramientas-para-fuzzing)
  - [🔹 Feroxbuster](#feroxbuster)
  - [🔹 WFUZZ](#wfuzz)
    - [🔹Códigos de estado comunes](#codigos-comunes)
    - [📖wfuzz-Doble patrón](#doble-patron)
    - [📖Fuzzeo por niveles con Wfuzz](#fuzz-niveles)
    - [📖fuzzeo autentificado, opciones de cabecera](#aut-cabezera)
  - [🔹Nmap](#nmap)
---
<a name="qué-es-hacer-fuzzing"></a>
## ❓ ¿Qué es hacer Fuzzing?

Básicamente, **fuzzing** es utilizar un diccionario de palabras para descubrir rutas ocultas o interesantes dentro de una página web.

Este proceso puede ayudarnos a encontrar:

- Páginas de presentación  
- Paneles de registro o administración  
- Secciones de descargas  
- Rutas no documentadas o restringidas  

El fuzzing permite identificar rutas que quizás **no deberían ser visibles** o que contengan errores que podrían ser **explotables**.

---
<a name="herramientas-para-fuzzing"></a>
## 🛠 Herramientas para Fuzzing
<a name="feroxbuster"></a>
### 🔹 Feroxbuster

#### 📦 Instalación

# Instalación desde repositorios:
```bash
sudo apt update && sudo apt install -y feroxbuster
```
# Si ese método no funciona, usa el script oficial:
```bash
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | bash
```

🚀 Ventaja principal
Feroxbuster hace fuzzing en varias capas:
Si encuentra una ruta nueva, continuará explorando dentro de ella automáticamente.

⚙️ Uso básico

```bash
feroxbuster --url "http://<IP>" -w <DICCIONARIO> <OPCIONES>
```
📌 ejemplo:
```
feroxbuster --url "http://172.17.0.2/logs" -w /opt/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  -x php,txt,html,zip,log,bin   
```
🔧 Opciones útiles
Extensiones:
```bash
-x php,txt,html,zip,log,bin
```
Encabezados personalizados (por ejemplo, para APIs):
```bash
-H "Accept: application/json" -H "Authorization: Bearer {token}"
```
Para usar con Burp Suite como proxy:
```bash
feroxbuster -u http://127.0.0.1 --insecure --proxy http://127.0.0.1:8080
```

<a name="wfuzz"></a>

### Wfuzz

otra herramienta de las más conocidas es wfuzz, su funcionamiento es sencillo:

```bash
wfuzz -w [DICCIONARIO] [URL VICTIMA]/FUZZ
```
Básicamente lo que hace es susttuir las palabras del diccionario en la palabra FUZZ
y mediante el código de estado sabremos si existe o no

<a name="codigos-comunes"></a>
## 📄 Códigos de Estado HTTP Comunes

| Código | Significado                      | Descripción breve                                     |
|--------|---------------------------------|------------------------------------------------------|
| 200    | OK                              | La solicitud ha sido procesada correctamente.        |
| 301    | Moved Permanently               | La URL solicitada ha sido movida permanentemente.    |
| 302    | Found (Redirección temporal)   | La URL solicitada ha sido movida temporalmente.      |
| 400    | Bad Request                    | La solicitud no pudo ser entendida por el servidor.  |
| 401    | Unauthorized                   | Se requiere autenticación para acceder al recurso.   |
| 403    | Forbidden                     | El servidor entendió la solicitud, pero se niega a cumplirla. |
| 404    | Not Found                     | El recurso solicitado no se encontró en el servidor. |
| 500    | Internal Server Error          | Error interno del servidor.                           |
| 502    | Bad Gateway                   | El servidor actuó como gateway y recibió una respuesta inválida. |
| 503    | Service Unavailable           | El servidor no está disponible temporalmente.        |
| 504    | Gateway Timeout               | Tiempo de espera agotado para una respuesta del gateway. |

---

siguiendo con wfuzz aquí os dejo las opciones más comunes
## ⚙️ Opciones Comunes de Wfuzz

| Opción          | Descripción                                                       | Ejemplo                                      |
|-----------------|-------------------------------------------------------------------|----------------------------------------------|
| `-z <modo>`     | Modo de generación de payloads (por ejemplo: `file`, `range`)    | `-z file,wordlist.txt`                        |
| `-c`            | Colorea la salida para mejor visualización                        | `-c`                                          |
| `--hc <códigos>`| Oculta respuestas con códigos HTTP específicos (ej: 404, 500)    | `--hc 404,500`                                |
| `-l`            | Muestra la longitud de la respuesta                               | `-l`                                          |
| `-w <archivo>`  | Diccionario o wordlist para fuzzing                               | `-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` |
| `-u <URL>`      | URL objetivo con el punto de inyección marcado con `FUZZ`        | `-u http://example.com/FUZZ`                  |
| `-H <header>`   | Añade encabezados HTTP personalizados                             | `-H "Authorization: Bearer token123"`         |
| `--hh <longitud>` | Oculta respuestas con longitud específica                       | `--hh 100`                                    |
| `--hw <palabras>` | Oculta respuestas con número específico de palabras             | `--hw 10`                                     |
| `--hl <líneas>` | Oculta respuestas con cierta cantidad de líneas                  | `--hl 20`                                     |

---

### Ejemplo básico:

```bash
wfuzz -c --hc 404 -w wordlist.txt -u http://example.com/FUZZ
```
<a name="doble-patron"></a>
### Opción: Fuzzing con doble patrón (ruta + extensión)

Wfuzz permite hacer fuzzing en dos partes de la URL simultáneamente. Por ejemplo, puedes probar rutas y combinarlas con distintas extensiones para encontrar archivos específicos.

El comando usa dos marcadores `FUZZ` y `FUZ2Z` en la URL para indicar los dos lugares donde se aplicará fuzzing, y la opción `-z list,php-txt` indica que se usan dos listas diferentes: una para las rutas y otra para las extensiones.

```bash
wfuzz -z list,php-txt http://<URL>/FUZZ.FUZ2Z
```
-z list,php-txt: la primera lista (por ejemplo, nombres de archivos o carpetas) y la segunda lista con las extensiones (php, txt, etc.).

**FUZZ:** marcador para la primera lista (rutas).  
**FUZ2Z:** marcador para la segunda lista (extensiones).

---

Wfuzz probará combinaciones como estas:

```plaintext
http://<URL>/admin.php
http://<URL>/login.txt
http://<URL>/index.php
```
Esta técnica es muy útil para descubrir archivos con distintas extensiones en un servidor web.

<a name="fuzz-niveles"></a>
## 🔍 Fuzzeo por niveles con Wfuzz

**Wfuzz** contempla una opción de fuzzeo recursivo que permite explorar rutas encontradas durante el escaneo inicial.

Si añadimos la opción `-R <nivel>`, primero fuzzeará la página o ruta base. Si encuentra algo interesante (como un directorio accesible), profundizará automáticamente y continuará el fuzzeo dentro de esa nueva ruta.

### 🛠️ Sintaxis de la opción:
```
-R <nivel de profundidad desde 1 en adelante>
```

Puedes especificar cuántos niveles de profundidad deseas explorar. Cuanto mayor sea el número, más profunda será la exploración.

### 📌 Ejemplo:
```bash
wfuzz -c --hc 404 -w wordlist.txt -u http://example.com/FUZZ -R 3
```
Este comando hará fuzzeo en:

 - http://example.com/FUZZ

 - Y si encuentra rutas válidas como http://example.com/admin/, continuará en:

    - http://example.com/admin/FUZZ

    - http://example.com/admin/panel/FUZZ, etc., hasta 3 niveles de profundidad.

⚠️ Ten en cuenta que a mayor profundidad, mayor será la cantidad de peticiones, por lo tanto puede afectar el rendimiento o ser más detectable.

<a name="aut-cabezera"></a>
## fuzzeo autentificado, opciones de cabecera

Algunas veces solo tendremos acceso a partes de la página estando logeados como usuario, para ello debemos hacerlo desde la propia herramienta 

de fuzzin, con wfuzz,Para hacer fuzz autenticado con Wfuzz usando una cookie de sesión, simplemente debes añadir la cabecera Cookie: en la opción -H (header).

Esto es útil cuando ya te has logueado (por ejemplo, en el navegador o con Burp) y copias tu cookie de sesión para tener acceso a rutas que sólo están disponibles para usuarios autenticados.
```bash
wfuzz -c -w wordlist.txt -u http://target.com/FUZZ -H "Cookie: SESSION=valor"
```
🧪 Ejemplo real:
Supongamos que tienes esta cookie después de loguearte:
```
SESSIONID=abcd1234xyz
```
Entonces el comando sería:
```bash
-H "Cookie: SESSIONID=abcd1234xyz" -H "User-Agent: Wfuzz"
```

-Si usas Burp Suite, puedes copiar toda la cabecera de cookies directamente desde una petición ya autenticada y pegarla en Wfuzz.

-También puedes usar el flag --hh para filtrar por tamaño de respuesta (útil para detectar páginas válidas aunque devuelvan 200).

<a name="nmap"></a>
### 🔹 NMAP

Otra forma sin salir de nmap es utilizando scripts, en este caso el "http-enum"
la forma de usarlo es la siguiente:
```bash
nmap --script http-enum -p <PUERTOS> <IP>
```
ejemplo:
```
nmap --script http-enum -p 80 172.17.0.2
```
otra forma si queremos profundizar en una ruta es la siguiente:
 ```bash
nmap -p <PUERTO> <IP> --script http-enum --script-args http-enum.basepath=<RUTA>
```
ejemplo
```
nmap -p 80 172.17.0.2 --script http-enum --script-args http-enum.basepath=/login.php

```
📌 Explicación:

  -p 80: especifica el puerto HTTP.

  172.17.0.2: es la IP del objetivo.

  --script http-enum: usa el script adecuado.

  --script-args http-enum.basepath=/login.php: indica el path desde donde empezar a buscar directorios (debe ser relativo, no incluir el host/IP).
