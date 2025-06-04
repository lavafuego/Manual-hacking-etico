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
## ## 🚀 Fuzzing de Subdominios con WFUZZ

Una vez que sospechamos que un servidor web podría estar manejando múltiples **Virtual Hosts**, podemos usar herramientas de fuzzing como **Wfuzz** para descubrir **subdominios ocultos o virtual hosts mal configurados**.

### 📌 Comando básico

```bash
wfuzz -c --hc=404 -w <DICCIONARIO> -H "Host: FUZZ.DOMINIO" http://DOMINIO | tee dominios
```
| Parámetro                 | Descripción                                                                             |                                                                                    |
| ------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `-c`                      | Colorea la salida para facilitar la lectura                                             |                                                                                    |
| `--hc=404`                | Oculta todas las respuestas con código HTTP `404 Not Found`                             |                                                                                    |
| `-w <DICCIONARIO>`        | Wordlist con posibles nombres de subdominios (por ejemplo: `admin`, `dev`, `api`, etc.) |                                                                                    |
| `-H "Host: FUZZ.DOMINIO"` | Inserta `FUZZ` en la cabecera `Host:` para probar con cada entrada del diccionario      |                                                                                    |
| `http://DOMINIO`          | URL base del servidor al que se envían las peticiones                                   |                                                                                    |
| \`                        | tee dominios\`                                                                          | Guarda la salida en un archivo llamado `dominios` y también la muestra en pantalla |
```
wfuzz -c --hc=404 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.realgob.dl" http://realgob.dl | tee subdominios.txt

```
💡 Nota: Asegúrate de que realgob.dl resuelva a la IP correcta (usa /etc/hosts si es necesario):
```
127.0.0.1 realgob.dl

```
🛠️ Otras opciones útiles

| Opción     | Descripción                                                                           |
| ---------- | ------------------------------------------------------------------------------------- |
| `-L`       | Sigue redirecciones (`Location:` headers)                                             |
| `--hl=<n>` | Oculta respuestas con `<n>` líneas exactas (útil para filtrar páginas tipo plantilla) |
| `--hh=<n>` | Oculta respuestas con `<n>` bytes de cuerpo (body size)                               |
| `--hw=<n>` | Oculta respuestas con `<n>` palabras                                                  |

📁 Diccionarios recomendados (SecLists)
```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
```
✅ Buenas prácticas
  Usa --hc, --hl, --hh o --hw para reducir ruido y falsos positivos.

  Valida que el servidor diferencie bien por Host; si no, todos los subdominios devolverán lo mismo.

  Si no ves resultados útiles, prueba cambiando el código a excluir (--hc=200, por ejemplo), ya que algunos servidores devuelven 200 OK incluso cuando el contenido no existe.





<a name="gobuster"></a>
## 🚀 Fuzzing de Subdominios con Gobuster

Otra herramienta que gusta por su rapidez es Gobuster, la forma de buscar subdominios con esta herramienta es la siguiente:

### 🛠️ Sintaxis básica
```
gobuster vhost -u http://<dominio> -w <DICCIONARIO> <OPCIONES>
```
- **vhost**: Subcomando para enumerar virtual hosts.

- **-u / --url**: URL base del objetivo (debe incluir el protocolo).

- **-w / --wordlist**: Diccionario de posibles subdominios.

- **[opciones]**: Parámetros opcionales como número de hilos, filtros, etc.

### Ejemplo
```
gobuster vhost -u http://ejemplo.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```
---

### ⚙️ Recomendaciones

Editar `/etc/hosts` si estás trabajando con entornos locales o staging:
```
192.168.1.10 ejemplo.com
```
Combina con Burp o DNS resolvers si necesitas más precisión o bypass.

Usa diccionarios específicos para subdominios como los de SecLists.

---

### 📁 Diccionarios recomendados
```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
```
---

### 🛠️ Otras opciones útiles

Añadir la opción `--append-domain`, ejemplo:
```
gobuster vhost -u pl0t.nyx -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
Ocultar códigos de estado con `-b` y ocultar errores con `--no-error`, ejemplo:
```
gobuster dir -u http://realgob.dl -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php,txt,html,py,db,js,png,jpg -t 200 -b 404,403 --no-error
```
En caso de que `--exclude-status` o `-b` fallen, puedes usar un `grep -v` como alternativa:
```
gobuster vhost -u hackzones.hl -w <diccionario> --append-domain | grep -v "400\|404"
```
Si la página tiene login usar `-U` y `-P` para usuario y contraseña respectivamente:
```
gobuster vhost -u http://www.example.com/ -w /path/to/dictionary -U username -P password
```
Gobuster puede utilizar indistintamente el método GET o POST, aunque por defecto usa GET:
```
gobuster vhost -u http://www.example.com/ -w /path/to/dictionary -m POST
```
Cuando estás enumerando directorios en un sitio con certificado TLS autofirmado o inválido y no quieres que falle la verificación SSL, usa `-k`:
```
gobuster dir -u https://target.local -w /usr/share/wordlists/dirb/common.txt -k
```
---

### 📚 Tabla completa de opciones comunes

| Opción                         | Descripción                                                                  |
|-------------------------------|------------------------------------------------------------------------------|
| `-c`                          | Cookies que se usarán en las solicitudes                                     |
| `-x`                          | Extensión(es) de archivo a buscar (ej: `php`, `txt`, `html`)                 |
| `-H`                          | Especificar encabezados HTTP, por ejemplo: `-H "Header: valor"`              |
| `-k`                          | Omitir la verificación del certificado TLS (útil con certificados autofirmados) |
| `-n`                          | No imprimir códigos de estado en la salida                                  |
| `-P`                          | Contraseña para autenticación básica                                         |
| `-s`                          | Lista de códigos de estado HTTP a considerar como positivos                 |
| `-b`                          | Lista de códigos de estado HTTP a excluir (blacklist)                       |
| `-U`                          | Usuario para autenticación básica                                            |



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

