

## ¿QUÉ ES UN LOG?

Un log en **Apache** es un archivo donde el servidor web **Apache HTTP Server** registra información sobre su actividad.

Estos registros son útiles para:

- monitorear el tráfico del sitio web
- diagnosticar errores
- analizar el rendimiento
- detectar posibles ataques o comportamientos anómalos

## Ubicación de los logs

La ubicación depende de la configuración y del sistema operativo, pero comúnmente:

- **Linux:** `/var/log/apache2/` o `/var/log/httpd/`
- **Windows:** dentro del directorio de instalación de Apache

## Configuración

Los logs se configuran en el archivo de configuración de Apache (`httpd.conf` o `apache2.conf`) mediante las siguientes directivas:

```apache
CustomLog "/var/log/apache2/access.log" combined
ErrorLog "/var/log/apache2/error.log"
```

## Tipos de logs en Apache

### 1️⃣ Access log (`access.log`)

El *Access log* registra **cada solicitud que recibe el servidor**.

Incluye información como:

- dirección IP del visitante  
- fecha y hora de la solicitud  
- página o recurso solicitado  
- código de estado HTTP (200, 404, 500, etc.)  
- navegador del cliente (User-Agent)  

**Ejemplo de línea típica:**  
```192.168.1.1 - - [06/Jun/2025:12:34:56 +0000] "GET /index.html HTTP/1.1" 200 1024```

### 2️⃣ Error log (`error.log`)

El *Error log* registra **mensajes de error y advertencias del servidor**.

Incluye:

- errores de configuración  
- errores de permisos  
- scripts que fallan (por ejemplo, en PHP)  
- problemas de red o hardware  

**Ejemplo de línea típica:**  
```[Fri Jun 06 12:34:56.789012 2025] [core:error] [pid 1234] [client 192.168.1.1:54321] File does not exist: /var/www/html/missing-page.html```
```
