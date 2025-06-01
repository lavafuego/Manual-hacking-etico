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

## 🧪 Tipos de escaneo (más comunes)

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
nmap -sU 172.17.0.2 --top-ports 100 -oN UDPscan
```

> ⚠️ **No se recomienda `-p-` con `-sU`** salvo que sea estrictamente necesario, por su lentitud.

---

✅ **Consejo final:** puedes combinar múltiples IPs, múltiples opciones de salida, escaneos mixtos TCP/UDP y más. Nmap es extremadamente potente si se domina bien.
