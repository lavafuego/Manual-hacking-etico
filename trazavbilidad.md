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

para nosotros lo importante es que aparezca
```
1 packets transmitted, 1 received
```
y no
```
1 packets transmitted, 0 received
```
ademas el ttl porque dependiendo de su valor sabremos que sistema opetativo corre:
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
