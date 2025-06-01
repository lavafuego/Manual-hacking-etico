# 📑 Índice de Contenidos

- [🔍 Puertos Abiertos y Servicios Asociados](#-puertos-abiertos-y-servicios-asociados)
  - [🛠 Herramienta principal: Nmap](#-herramienta-principal-nmap)
  - [🔰 Uso básico](#-uso-básico)
  - [⚠️ Nota sobre puertos](#️-nota-sobre-puertos)
  - [🔍 Estructura del comando Nmap](#-estructura-del-comando-nmap)
  - [🧪 tipos de escaneo comunes](#-tipos-de-escaneo-comunes)
  - [🎯 Rango de puertos](#-rango-de-puertos)
  - [⚙️ Opciones útiles](#️-opciones-útiles)
  - [🔌 Detección de servicios](#-detección-de-servicios)
  - [💾 Guardar la salida](#-guardar-la-salida)
  - [✅ Comando recomendado (CTF tested)](#-comando-recomendado-ctf-tested)
  - [🌐 Escaneo UDP con Nmap](#-escaneo-udp-con-nmap)
  - [🔌 Puertos más comunes y sus servicios](#-puertos-más-comunes-y-sus-servicios)
  - [🛠 Script alternativo en bash](#-si-no-funciona-el-scan-por-udp-ni-tcpip-podemos-usar-este-script)

 - [🔍 Nmap: Uso de Scripts para Detectar Vulnerabilidades](#-nmap-uso-de-scripts-para-detectar-vulnerabilidades)
   - [🔎 Escaneo básico](#escaneo-básico-con-scripts-de-vulnerabilidades)
   - [📋 Escaneo avanzado](#escaneo-avanzado-todos-los-puertos--scripts--detección-de-so-y-versiones)
   - [Selección de scripts específicos](#selección-de-scripts-específicos)
   - [🔍 NMAP: Uso de Scripts para Detectar Vulnerabilidades](#-🔍-nmap-uso-de-scripts-para-detectar-vulnerabilidades)
   - [Escaneo básico con scripts de vulnerabilidades](#escaneo-básico-con-scripts-de-vulnerabilidades)
   - [Escaneo avanzado: todos los puertos + scripts + detección de SO y versiones](#escaneo-avanzado-todos-los-puertos--scripts--detección-de-so-y-versiones)
   - [Consultar scripts disponibles](#consultar-scripts-disponibles)
   - [Escaneo detallado con verbosity máxima](#escaneo-detallado-con-verbosity-máxima)





# 🔍 PUERTOS ABIERTOS Y SERVICIOS ASOCIADOS

Para poder acceder a una máquina vulnerable, es fundamental conocer qué puertos tiene abiertos y qué servicios están corriendo en ellos. Esto nos permite identificar posibles puntos de entrada aprovechando vulnerabilidades.

## 🛠 Herramienta principal: Nmap

`nmap` es una herramienta muy versátil que permite desde el descubrimiento de hosts hasta la detección de servicios y sistemas operativos.

### 🔰 Uso básico:

```bash
nmap <IP o HOSTNAME>
```

#### 📌 Ejemplo:

```bash
nmap 172.17.0.2
```

Este escaneo básico revisa **los 1000 puertos TCP más comunes**.

> ⚠️ **Nota:** En total hay **65,536 puertos** (del 0 al 65535), tanto TCP como UDP.

---

## 🔍 Estructura del comando `nmap`

```bash
nmap [tipo de escaneo] [opciones] {objetivo(s)}
```

---

## 🔄 Negociación de tres pasos (TCP 3-Way Handshake)

1. El cliente envía un paquete **SYN**.
2. El servidor responde con un **SYN/ACK** si el puerto está abierto.
3. El cliente responde con un **ACK** para establecer la conexión.

---

## 🧪 tipos de escaneo comunes

| Opción | Descripción |
|--------|-------------|
| `-sS`  | Escaneo SYN (rápido y sigiloso) |
| `-sT`  | Escaneo TCP completo (menos sigiloso) |
| `-PA`  | Escaneo con paquetes TCP ACK |
| `-PS`  | Escaneo con paquetes TCP SYN a puertos específicos |
| `-PR`  | Ping ARP (efectivo en redes locales) |
| `-sA`  | Detectar si un puerto está filtrado (firewall) |

---

## 🎯 Rango de puertos

| Opción         | Descripción                       |
|----------------|-----------------------------------|
| `-p-`          | Escanea **todos los puertos (0-65535)** |
| `-p 21-400`    | Escanea puertos desde el 21 al 400 |
| `-p 21,22,80`  | Escanea una lista específica de puertos |

---

## ⚙️ Opciones útiles

| Opción        | Descripción |
|---------------|-------------|
| `--open`      | Solo muestra puertos abiertos |
| `-v` / `-vvv` | Verbose (más detalles) |
| `-O`          | Detecta sistema operativo |
| `-A`          | Sistema operativo + traceroute + servicios |
| `-F`          | Escaneo rápido |
| `-r`          | Escaneo de puertos en orden ascendente |

---

## 🔌 Detección de servicios

| Opción  | Descripción                             |
|---------|-----------------------------------------|
| `-sC`   | Usa scripts por defecto (detección básica de servicios) |
| `-sV`   | Detecta versiones de servicios          |
| `-sCV` | Combina ambos                           |

---

## 💾 Guardar la salida

Puedes guardar los resultados en un archivo de varias formas:

```bash
nmap 172.17.0.2 > scan.txt       # Redirección simple
nmap 172.17.0.2 -oN salida.txt   # Formato legible tipo Nmap
nmap 172.17.0.2 -oX salida.xml   # Formato XML
nmap 172.17.0.2 -oG salida.grep  # Formato grepeable
```

---

## ✅ Comando recomendado (CTF tested)

Después de varios años haciendo CTFs, este comando me resulta el más completo y efectivo:

```bash
sudo nmap -sS -sCV -Pn --min-rate 5000 -p- -vvv --open <IP> -oN <archivo_de_salida>
```

#### 🧪 Ejemplo:

```bash
sudo nmap -sS -sCV -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN puertosYservicios
```

---

## 🌐 Escaneo UDP con Nmap

Los escaneos UDP son **mucho más lentos** que los TCP y pueden dar muchos falsos negativos si el servicio no responde.

### 🔧 Comando básico UDP:

```bash
nmap -sU <IP> <puertos>
```

#### 📌 Ejemplo:

```bash
nmap -sU 172.17.0.2 --top-ports=100 -oN UDPscan
```

> ⚠️ **No se recomienda `-p-` con `-sU`** salvo que sea estrictamente necesario, por su lentitud.

---

✅ **Consejo final:** puedes combinar múltiples IPs, múltiples opciones de salida, escaneos mixtos TCP/UDP y más. Nmap es extremadamente potente si se domina bien.



# 🔌 PUERTOS MÁS COMUNES Y SUS SERVICIOS

A continuación se muestran los puertos más comunes que suelen estar abiertos en sistemas y redes, junto con los servicios típicos que corren en ellos.

| Puerto | Protocolo | Servicio común        | Descripción                          |
|--------|-----------|----------------------|------------------------------------|
| 22     | TCP       | SSH                  | Secure Shell (acceso remoto seguro)|
| 21     | TCP       | FTP                  | File Transfer Protocol             |
| 23     | TCP       | Telnet               | Terminal remoto no seguro           |
| 25     | TCP       | SMTP                 | Protocolo para envío de correo     |
| 53     | TCP/UDP   | DNS                  | Resolución de nombres de dominio   |
| 67     | UDP       | DHCP Server           | Asignación dinámica de IP           |
| 68     | UDP       | DHCP Client           | Cliente DHCP                       |
| 80     | TCP       | HTTP                 | Protocolo web sin cifrado          |
| 110    | TCP       | POP3                 | Protocolo de correo para recibir   |
| 119    | TCP       | NNTP                 | Protocolo para grupos de noticias  |
| 123    | UDP       | NTP                  | Sincronización de tiempo           |
| 143    | TCP       | IMAP                 | Protocolo de correo para recibir   |
| 161    | UDP       | SNMP                 | Monitorización y gestión de red    |
| 194    | TCP       | IRC                  | Chat en tiempo real                 |
| 443    | TCP       | HTTPS                | HTTP seguro con TLS/SSL             |
| 445    | TCP       | SMB                  | Compartición de archivos Windows   |
| 465    | TCP       | SMTP (SSL)            | SMTP con cifrado SSL/TLS            |
| 514    | UDP       | Syslog               | Envío de mensajes de sistema       |
| 587    | TCP       | SMTP (Submission)     | Envío de correo con autenticación  |
| 993    | TCP       | IMAPS                | IMAP con cifrado SSL/TLS            |
| 995    | TCP       | POP3S                | POP3 con cifrado SSL/TLS            |
| 3306   | TCP       | MySQL                | Base de datos MySQL                |
| 3389   | TCP       | RDP                  | Escritorio remoto Windows          |
| 5432   | TCP       | PostgreSQL           | Base de datos PostgreSQL           |
| 5900   | TCP       | VNC                  | Control remoto de escritorios      |
| 8080   | TCP       | HTTP-Alt             | Servidor HTTP alternativo           |

---

**Nota:** Esta lista no es exhaustiva, pero cubre la mayoría de servicios usados en redes típicas.


## 🛠 Si no funciona el scan por UDP ni TCP/IP podemos usar este script:

```bash
#!/bin/bash
# Escaneo básico de TODOS los puertos TCP (1-65535) usando /dev/tcp
# Uso: ./scan_all_ports.sh <IP>

trap ctrl_c INT

function ctrl_c() {
  echo -e "\n[!] Escaneo interrumpido por el usuario."
  exit 1
}

if [ "$#" -ne 1 ]; then
  echo "Uso: $0 <IP>"
  exit 1
fi

IP=$1

echo "Escaneando todos los puertos TCP en $IP (1-65535)... (Ctrl+C para detener)"

for ((port=1; port<=65535; port++)); do
  (echo > /dev/tcp/$IP/$port) >/dev/null 2>&1
  if [ $? -eq 0 ]; then
    echo "Puerto $port está abierto"
  fi
done

echo "Escaneo completado."
```


# 🔍 NMAP: Uso de Scripts para Detectar Vulnerabilidades

## Escaneo básico con scripts de vulnerabilidades
nmap --script vuln IP_O_HOST
# --script vuln: ejecuta todos los scripts relacionados con vulnerabilidades conocidas.
# IP_O_HOST: dirección IP o nombre del host objetivo.

# Ejemplo:
nmap --script vuln 172.17.0.2

# Escaneo avanzado: todos los puertos + scripts + detección de SO y versiones
sudo nmap -p- -sV -O --script vuln IP_O_HOST -oN archivo_de_salida
-p-: escanea todos los 65535 puertos.

-sV: detecta versiones de servicios.

-O: detecta sistema operativo.

--script vuln: ejecuta scripts de vulnerabilidades.

-oN archivo_de_salida: guarda la salida en un archivo.

# Selección de scripts específicos
```bash
nmap --script smb-vuln* IP_O_HOST
```
# Ejemplo para buscar vulnerabilidades SMB.

 Scripts comunes para vulnerabilidades:
http-vuln*   Busca vulnerabilidades en servicios HTTP

smb-vuln*    Escanea vulnerabilidades SMB

ftp-vuln*    Escanea vulnerabilidades FTP

ssl*        Escanea problemas en configuraciones SSL/TLS

# Consultar scripts disponibles
```bash
ls /usr/share/nmap/scripts/
```
 O para filtrar scripts relacionados con vulnerabilidades:
```bash
ls /usr/share/nmap/scripts/ | grep vuln
```
# Escaneo detallado con verbosity máxima
sudo nmap -p- -sV -O --script vuln -vvv IP_O_HOST
# -vvv: nivel máximo de verbosity para más detalles en la salida.
