## TRAZABILIDAD

Lo primero es comprobar la trazabilidad de nuestra máquina atacante con la victima
para ello utilizaremos herramientas como "ping"
```bash
ping -c 1 <IP>
```
ping manda paquetes ICMP con "-c 1" manda un único paquete sin esta opción manda continuamente hasta que abortemos

ejemplo de uso:
```bash
ping -c 1 172.17.0.2 
```
salida del comando:
```
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.112 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.112/0.112/0.112/0.000 ms
```
-¿Cómo leer los resultados?
```
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
```
-En esta parte te dice que manda un paquete ICMP de 56 bytes que con encabezado se convierte en 84
```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.112 ms
```
Recibiste una respuesta desde esa IP.

icmp_seq=1: Es el número de secuencia del paquete ICMP (como un ID).

ttl=64: "Time to Live", evita bucles infinitos. El valor 64 es típico de sistemas Linux.

time=0.112 ms: El tiempo que tardó en ir y volver el paquete (muy rápido: 0.112 milisegundos, lo que indica conexión local o de red muy cercana).
```bash
--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.112/0.112/0.112/0.000 ms
```
1 transmitted, 1 received: El paquete fue enviado y recibido correctamente.

0% packet loss: No hubo pérdida de paquetes, lo cual es perfecto.

rtt: Round Trip Time (tiempo ida y vuelta).

min/avg/max: Todos iguales (0.112 ms), porque solo hiciste 1 intento.

mdev: Desviación estándar (0.000 porque solo se envió un paquete).

## ------PARA NOSOTROS LO IMPORTANTE ES:---- 
que aparezca
```
1 packets transmitted, 1 received
```
y no
```
1 packets transmitted, 0 received
```
En esta segunda vemos 0 recived y eso nos indica que hemos mandado un paquete y no hemos
recibido la respuesta, luego no tenemos trazabilidad con la máquina

ademas también nos importa el valor del ttl porque dependiendo de su valor sabremos que sistema opetativo corre:
```bash
| Sistema Operativo      | TTL Inicial Típico | TTL que Sueles Ver en Ping |
|------------------------|--------------------|-----------------------------|
| Linux / Unix           | 64                 | 64                          |
| macOS                  | 64                 | 64                          |
| Windows (todas)        | 128                | 128                         |
| Cisco (routers/switch) | 255                | 255                         |
| Solaris                | 255                | 255                         |
| FreeBSD / OpenBSD      | 64                 | 64                          |
| AIX                    | 64                 | 64                          |
| HP-UX                  | 64                 | 64                          |
```

## SI FALLA PING

Hay otras opciones a ping, para ello clonamos el repositorio de tcping:
```bash
git clone https://github.com/cloverstd/tcping.git
```
entramos en la capeta crada
```bash
cd tcping
```
compilamos:
```bash
 go build -ldflags "-s -w" .
```
comprimimos
```bash
 upx brute tcping
```
uso:
```bash
./tcping <ip>
```

EJEMPLO:
```bash
./tcping 172.17.0.2 
```
```
Ping tcp://172.17.0.2:80(172.17.0.2:80) connected - time=215.809µs dns=0s
Ping tcp://172.17.0.2:80(172.17.0.2:80) connected - time=747.234µs dns=0s
Ping tcp://172.17.0.2:80(172.17.0.2:80) connected - time=579.401µs dns=0s
Ping tcp://172.17.0.2:80(172.17.0.2:80) connected - time=257.58µs dns=0s

Ping statistics tcp://172.17.0.2:80
        4 probes sent.
        4 successful, 0 failed.
Approximate trip times:
        Minimum = 215.809µs, Maximum = 747.234µs, Average = 450.006µs%
```

## OTRAS FORMAS
-traceroute:

```bash
traceroute <ip>
```
ejemplo:
```bash
traceroute 172.17.0.2
```
```                                                                                                                                   
traceroute to 172.17.0.2 (172.17.0.2), 30 hops max, 60 byte packets
 1  172.17.0.2 (172.17.0.2)  0.156 ms  0.049 ms  0.037 ms
```

-nmap

```bash
nmap -Pn <IP>
```
ejemplo:
```bash
nmap -Pn 172.17.0.2
```
```
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
-Pn evita que nmap use ping (ideal si ICMP está bloqueado).

-También puedes usar nmap -sT para hacer escaneo TCP.



