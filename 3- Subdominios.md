# 📑 Índice de Contenidos

- [🔍 Enumeración de Subdominios y Virtual Hosting](#-enumeración-de-subdominios-y-virtual-hosting)
  - [❓ ¿Qué es el Virtual Hosting?](#-qué-es-el-virtual-hosting)
  - [🧠 ¿Cómo detectarlo?](#-cómo-detectarlo)
    - [🔎 IP-Based](#-ip-based)
    - [🔎 Port-Based](#-port-based)
    - [🔎 Name-Based](#-name-based)
  - [🚀 Fuzzing de Subdominios](#-fuzzing-de-subdominios)
    - [🛠️ Otras opciones útiles](#-otras-opciones-útiles)
    - [💡 Alternativas con grep](#-alternativas-con-grep)
  - [📚 Diccionarios Recomendados](#-diccionarios-recomendados)
    - [directory-list-2.3-medium.txt](#directory-list-23-mediumtxt)
    - [rockyou.txt](#rockyoutxt)
    - [SecLists](#seclists)


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
## 🚀 FUZZING DE SUBDOMINIOS
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

## 🛠️ Otras opciones útiles:
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

