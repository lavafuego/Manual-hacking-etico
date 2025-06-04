# 📑 Índice de Contenidos

- [🔍 Enumeración de Subdominios y Virtual Hosting](#-enumeración-de-subdominios-y-virtual-hosting)
  - [❓ ¿Qué es el Virtual Hosting?](#-qué-es-el-virtual-hosting)
  - [🧠 ¿Cómo detectarlo?](#-cómo-detectarlo)
    - [🔎 IP-Based](#-ip-based)
    - [🔎 Port-Based](#-port-based)
    - [🔎 Name-Based](#-name-based)
  - [🚀 Fuzzing de Subdominios con Wfuzz](#wfuzz)
  - [🚀 Fuzzing de Subdominios con Gobuster](#gobuster)
    - [🛠️ Otras opciones útiles](#otras-opciones)
- [📚 DICCIONARIOS QUE RECOMIENDO](#-diccionarios-que-recomiendo)
  - [1. directory-list-2.3-medium.txt](#1-directory-list-23-mediumtxt)
  - [2. rockyou.txt](#2-rockyoutxt)
  - [3. SecLists](#3-seclists)
- [🛠⚠️⚠️⚠️ archivo /etc/hosts](#etc-hosts)





## 🔍 ENUMERACIÓN DE SUBDOMINIOS Y VIRTUAL HOSTING

Durante la fase de enumeración, una parte crítica es la búsqueda de **subdominios**, ya que pueden revelar servicios ocultos o vías alternativas de intrusión. Para ello, debemos entender primero qué es el virtual hosting y cómo detectarlo.

---

### ❓ ¿Qué es el Virtual Hosting?

El **virtual hosting** permite a un único servidor web alojar múltiples sitios web desde la **misma dirección IP**.

#### Tipos de Virtual Hosting:

| Tipo                      | Explicación                                                                                   | Pros / Contras                                        |
|--------------------------|-----------------------------------------------------------------------------------------------|-------------------------------------------------------|
| **Name-Based**           | Se diferencia por el **nombre del dominio** enviado en la cabecera `Host`.                   | ✅ Más común / ✅ Solo una IP / 🔒 Requiere SNI        |
| **IP-Based**             | Cada sitio tiene una **IP diferente**.                                                       | ✅ Ideal para certificados / ❌ Requiere más IPs       |
| **Port-Based**           | Cada sitio responde en **puertos distintos**.                                                 | ✅ Útil para pruebas / ❌ No apto para producción      |

---

### 🧠 ¿Cómo detectarlo?

#### 🔎 IP-Based:
No lo analizaremos, ya que se detecta simplemente por tener IPs distintas.

#### 🔎 Port-Based:
Se detecta fácilmente al escanear puertos abiertos (por ejemplo con `nmap`) y ver servicios HTTP en diferentes puertos.

#### 🔎 Name-Based:
Más difícil. Se alojan en la misma IP y puerto (`80` o `443`), diferenciándose solo por el **subdominio**. Ejemplo en Apache:

```apache
<VirtualHost *:80>
    ServerName www.ejemplo1.com
    DocumentRoot /var/www/ejemplo1
</VirtualHost>

<VirtualHost *:80>
    ServerName www.ejemplo2.com
    DocumentRoot /var/www/ejemplo2
</VirtualHost>
```
<a name="wfuzz"></a>
## 🚀 FUZZING DE SUBDOMINIOS CON WFUZZ
Una vez que sospechamos que hay Virtual Hosts, podemos detectarlos con herramientas de fuzzing de subdominios como wfuzz y gobuster.
```bash
wfuzz -c --hc=404 -w <DICCIONARIO> -H "Host: FUZZ.DOMINIO" http://DOMINIO | tee dominios
```
| Parámetro                    | Descripción                                                                   |                                                                 |
| ---------------------------- | ----------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `-c`                         | Colorea la salida                                                             |                                                                 |
| `--hc=404`                   | Oculta respuestas con código HTTP 404 (Not Found)                             |                                                                 |
| `-w`                         | Diccionario con posibles subdominios                                          |                                                                 |
| `-H "Host: FUZZ.realgob.dl"` | FUZZ se reemplaza por cada entrada del diccionario (`admin.realgob.dl`, etc.) |                                                                 |
| `http://realgob.dl`          | Dominio base que responde al virtual host                                     |                                                                 |
| \`                           | tee dominios\`                                                                | Guarda la salida en un archivo y también la muestra en pantalla |

<a name="otras-opciones"></a>
### 🛠️ Otras opciones útiles


-L → Sigue redirecciones

--hl=<líneas> → Oculta respuestas con cierta cantidad de líneas (ej: --hl=140)

--hh=<bytes> → Oculta respuestas con cierto número de caracteres

--hw=<palabras> → Oculta respuestas con número específico de palabras

| Parámetro          | Descripción                                                     |
| ------------------ | --------------------------------------------------------------- |
| `vhost`            | Modo de búsqueda de subdominios usando cabecera `Host`          |
| `-u hackzones.hl`  | Dominio objetivo                                                |
| `-w`               | Diccionario con posibles subdominios                            |
| `--append-domain`  | Añade automáticamente el dominio a cada entrada del diccionario |
| `--exclude-status` | Oculta códigos HTTP específicos (como 400, 404)                 |
💡 En caso de que --exclude-status falle, puedes usar un grep -v como alternativa:
```bash
gobuster vhost -u hackzones.hl -w <diccionario> --append-domain | grep -v "400\|404"
```
<a name="gobuuster"></a>
## 🚀 FUZZING DE SUBDOMINIOS CON GOBUSTER
Otra herramienta que gusta por su rapidez es Gobuster, la forma de buscr subdominios con esta herramienta es la siguiente:
  # 🛠️ Sintaxis básica
  ```bash
  gobuster vhost -u http://<dominio> -w <DICCIONARIO> <OPCIONES>
  ```
  vhost → subcomando para enumerar virtual hosts.

  -u / --url → URL base del objetivo (debe incluir el protocolo).

  -w / --wordlist → diccionario de posibles subdominios.

  [opciones] → parámetros opcionales como número de hilos, filtros, etc.
  EJEMPLO:
  ```
  gobuster vhost -u http://ejemplo.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
  ```
⚙️ Recomendaciones
Editar /etc/hosts si estás trabajando con entornos locales o staging:
```bash
192.168.1.10 ejemplo.com
```
Combina con Burp o DNS resolvers si necesitas más precisión o bypass.

Usa diccionarios específicos para subdominios como los de SecLists.

📁 Diccionarios recomendados
```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
```

condideraciones finales

añdir la opcion --append-domain, ejemplo:
```bash
gobuster vhost -u pl0t.nyx -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
ocultar codigos de estado con con -b y ocultar estados con --no-error, ejemplo:
| Parte                                  | Significado                                                                                                                  |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `gobuster dir`                         | Usa el subcomando `dir`, para buscar directorios y archivos en un sitio web.                                                 |
| `-u http://realgob.dl`                 | URL objetivo donde se buscarán rutas. Puede ser IP o dominio.                                                                |
| `-w /usr/share/wordlists/seclists/...` | Diccionario de nombres de archivos/directorios a probar. En este caso, una lista de palabras en minúsculas con tamaño medio. |
| `-x php,txt,html,py,db,js,png,jpg`     | Probará cada palabra del diccionario con estas extensiones. Ej: `login.php`, `admin.html`, `config.py`, etc.                 |
| `-t 200`                               | Usa 200 hilos (conexiones paralelas) para máxima velocidad. ⚠️ Puede ser demasiado para servidores frágiles.                 |
| `-b 404,403`                           | Oculta resultados con códigos de estado HTTP 404 (no encontrado) y 403 (prohibido). Útil para reducir "ruido".               |
| `--no-error`                           | Oculta errores de red (timeouts, etc.) para que la salida sea más limpia.                                                    |





# 📚 DICCIONARIOS QUE RECOMIENDO

---

### 1. directory-list-2.3-medium.txt

Este es un diccionario muy utilizado para fuzzing de directorios y archivos.

**Ruta típica:**

```bash
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
### 2. rockyou.txt
Este es uno de los diccionarios más famosos para ataques de fuerza bruta sobre contraseñas.

Ruta típica (comprimido):
```bash
/usr/share/wordlists/rockyou.txt.gz
```
Para poder usar rockyou.txt, primero debes descomprimirlo:
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```
### 3. SecLists
SecLists es un repositorio muy completo de diccionarios para múltiples propósitos (subdominios, directorios, contraseñas, fuzzing, etc.).

Para descargarlo:
```bash
git clone https://github.com/danielmiessler/SecLists.git
```
<a name="etc-hosts"></a>
## 🛠️ Redirección de subdominios usando /etc/hosts

📌 ¿Por qué modificar /etc/hosts para subdominios?  
Cuando trabajas con Git y entornos divididos por subdominios, puede ser útil redirigir esos subdominios manualmente, por ejemplo:

- dev.git.miempresa.com → entorno de desarrollo  
- staging.git.miempresa.com → entorno de pruebas  
- ci.git.miempresa.com → servidor de integración continua  

Para lo que a nosotros nos importa es que cuando tenemos una IP víctima y detectamos un dominio, lo
primero es añadirlo al /etc/hosts para que cuando apuntemos al subdominio nos abra correctamente la 
página entre otras cosas.

📎 Esto también es útil en pruebas de pentesting o laboratorios cuando el dominio no resuelve por DNS.

---

🛠️ **¿Cómo modificarlo?**

lo abriremos con nano o un editor de texto:
```bash
sudo nano /etc/hosts
```
y añadiremos una línea al final de la siguiente forma:
```bash
<IP>  <DOMINIO>
```
posteriormente cuando vayamos encontrando subdominios los iremos añadiendo consecutivamente:
```bash
<IP> <DOMINIO> <SUBDOMINIO> <SUBDOMINIO>
```
📌 Ejemplo:
```
192.168.1.100 git.miempresa.com dev.git.miempresa.com staging.git.miempresa.com ci.git.miempresa.com
```
⚠️ Recuerda que /etc/hosts no acepta comodines como *.git.miempresa.com, así que cada subdominio debe añadirse manualmente.

### ⚠️⚠️⚠️ Importante en un CTF añadir dominios y subdominios al /etc/hosts para que resuelva correctamente cuando apuntamos a esas páginas

