# 📑 Índice de Contenidos

- [🔍 Trazabilidad](#-trazabilidad)
  - [📡 Usando ping](#-usando-ping)
  - [📖 Interpretación de la salida](#-interpretación-de-la-salida)
  - [🧠 Deducción del sistema operativo por TTL](#-deducción-del-sistema-operativo-por-ttl)
  - [❌ ¿Ping no funciona?](#-ping-no-funciona)
    - [🔁 tcping](#-tcping)
  - [🌐 Otras herramientas](#-otras-herramientas)
    - [🔍 traceroute](#-traceroute)
    - [🛠️ nmap](#️-nmap)

# 🔍 TRAZABILIDAD

El primer paso es comprobar la **conectividad (trazabilidad)** entre nuestra máquina atacante y la víctima. Esto nos asegura que existe comunicación entre ambas.

## 📡 Usando `ping`

`ping` envía paquetes ICMP para comprobar si un host está accesible. El parámetro `-c 1` indica que solo se enviará un paquete:

```bash
ping -c 1 <IP>
```

### 📌 Ejemplo:

```bash
ping -c 1 172.17.0.2
```

### ✅ Resultado esperado:

```bash
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.112 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.112/0.112/0.112/0.000 ms
```

## 📖 Interpretación de la salida

- `56(84) bytes of data`: Se envían 56 bytes de datos; con encabezado IP e ICMP, son 84.
- `64 bytes from ...`: Se recibió respuesta desde la IP.
- `icmp_seq=1`: Número de secuencia del paquete (útil para identificar paquetes).
- `ttl=64`: "Time to Live". Evita bucles infinitos. El valor también da pistas sobre el sistema operativo.
- `time=0.112 ms`: Tiempo ida y vuelta (RTT), muy bajo en redes locales.

### ✅ Lo importante:

Nos interesa ver:

```
1 packets transmitted, 1 received
```

Y **NO**:

```
1 packets transmitted, 0 received
```

Lo segundo indica que el paquete no tuvo respuesta, por tanto **no hay trazabilidad**.

---

## 🧠 Deducción del sistema operativo por TTL

| Sistema Operativo      | TTL Típico |
|------------------------|------------|
| Linux / macOS / BSD    | 64         |
| Windows                | 128        |
| Cisco / Solaris        | 255        |

---

## ❌ ¿Ping no funciona?

Si ICMP está bloqueado, hay alternativas:

### 🔁 `tcping`

Permite realizar "ping" a través de TCP (por defecto al puerto 80).

### 🔧 Instalación:

```bash
git clone https://github.com/cloverstd/tcping.git
cd tcping
go build -ldflags "-s -w" .
upx brute tcping
```

### ▶️ Uso:

```bash
./tcping <IP>
```

### 📌 Ejemplo:

```bash
./tcping 172.17.0.2
```

### 📈 Resultado:

```bash
Ping tcp://172.17.0.2:80 connected - time=215.809µs dns=0s
Ping tcp://172.17.0.2:80 connected - time=747.234µs dns=0s
Ping tcp://172.17.0.2:80 connected - time=579.401µs dns=0s
Ping tcp://172.17.0.2:80 connected - time=257.580µs dns=0s

Ping statistics tcp://172.17.0.2:80
        4 probes sent.
        4 successful, 0 failed.
Approximate trip times:
        Minimum = 215.809µs, Maximum = 747.234µs, Average = 450.006µs
```

---

## 🌐 Otras herramientas

### 🔍 `traceroute`

Muestra los saltos que da un paquete hasta su destino:

```bash
traceroute <IP>
```

#### Ejemplo:

```bash
traceroute 172.17.0.2
```

```bash
traceroute to 172.17.0.2 (172.17.0.2), 30 hops max, 60 byte packets
 1  172.17.0.2 (172.17.0.2)  0.156 ms  0.049 ms  0.037 ms
```

---

### 🛠️ `nmap`

Detecta si el host está activo y qué puertos tiene abiertos. La opción `-Pn` evita que se use ICMP:

```bash
nmap -Pn <IP>
```

#### Ejemplo:

```bash
nmap -Pn 172.17.0.2
```

```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-31 13:05 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.37 seconds
```



