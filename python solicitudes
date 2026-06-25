# Guía rápida: Solicitudes HTTP en Python con Requests

## Instalación

```bash
pip install requests
```

## Ejemplo completo

```python
import requests

# Crear una sesión para mantener cookies automáticamente
session = requests.Session()

# URL objetivo
url = "https://httpbin.org/post"

# Encabezados (Headers)
headers = {
    "User-Agent": "Mozilla/5.0",
    "Accept": "application/json",
    "Accept-Language": "es-ES,es;q=0.9",
    "Referer": "https://ejemplo.com",
    "Origin": "https://ejemplo.com"
}

# Cookies
cookies = {
    "sessionid": "abc123",
    "token": "xyz789"
}

# Parámetros de consulta (Query Params)
params = {
    "pagina": 1,
    "orden": "desc"
}

# Datos de formulario
form_data = {
    "usuario": "juan",
    "password": "1234"
}

# Datos JSON
json_data = {
    "nombre": "Juan",
    "edad": 25
}

# --------------------------------------------------
# GET
# --------------------------------------------------

response = session.get(
    "https://httpbin.org/get",
    headers=headers,
    cookies=cookies,
    params=params
)

print("=== GET ===")
print(response.status_code)
print(response.text)

# --------------------------------------------------
# POST FORMULARIO
# --------------------------------------------------

response = session.post(
    "https://httpbin.org/post",
    headers=headers,
    cookies=cookies,
    data=form_data
)

print("=== POST FORM ===")
print(response.status_code)
print(response.json())

# --------------------------------------------------
# POST JSON
# --------------------------------------------------

response = session.post(
    url,
    headers=headers,
    cookies=cookies,
    json=json_data
)

print("=== POST JSON ===")
print(response.status_code)
print(response.json())

# --------------------------------------------------
# COOKIES DE LA SESIÓN
# --------------------------------------------------

print(session.cookies.get_dict())

# --------------------------------------------------
# INFORMACIÓN DE LA PETICIÓN
# --------------------------------------------------

print(response.request.url)
print(response.request.method)
print(response.request.headers)
```

---

## Estructura básica de una petición

```python
response = session.post(
    "https://api.ejemplo.com/login",
    headers=headers,
    cookies=cookies,
    params=params,
    data=form_data
)

print(response.status_code)
print(response.text)
```

---

## Enviar JSON

```python
payload = {
    "email": "usuario@correo.com",
    "password": "123456"
}

response = session.post(
    "https://api.ejemplo.com/login",
    json=payload
)

print(response.json())
```

---

## Leer respuestas JSON

```python
datos = response.json()

print(datos["id"])
print(datos["nombre"])
```

---

## Obtener cookies automáticamente

```python
import requests

session = requests.Session()

session.get(
    "https://httpbin.org/cookies/set/test/123"
)

print(session.cookies.get_dict())
```

---

## Cómo copiar una petición desde Chrome

1. Abrir DevTools (`F12`).
2. Ir a la pestaña **Network**.
3. Realizar la acción deseada.
4. Seleccionar la petición.
5. Clic derecho.
6. Elegir **Copy → Copy as cURL**.

Ejemplo:

```bash
curl 'https://api.ejemplo.com/data' \
  -H 'User-Agent: Mozilla/5.0' \
  -H 'Authorization: Bearer TOKEN' \
  -b 'session=abc123'
```

A partir de ahí podrás extraer:

- URL
- Método HTTP
- Headers
- Cookies
- Query Params
- Payload
- Tokens

y reproducir la solicitud en Python usando `requests`.

---

## Qué revisar en DevTools

- Request URL
- Request Method
- Request Headers
- Cookies
- Query String Parameters
- Request Payload
- Response

Con esos elementos podrás replicar la mayoría de solicitudes web legítimas utilizando Python.
