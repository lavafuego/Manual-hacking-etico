## 📑 Índice de Contenidos
- [❓ ¿Qué es hacer Fuzzing?](#qué-es-hacer-fuzzing)
- [🛠 Herramientas para Fuzzing](#herramientas-para-fuzzing)
  - [🔹 Feroxbuster](#feroxbuster)

---
<a name="qué-es-hacer-fuzzing"></a>
## ❓ ¿Qué es hacer Fuzzing?

Básicamente, **fuzzing** es utilizar un diccionario de palabras para descubrir rutas ocultas o interesantes dentro de una página web.

Este proceso puede ayudarnos a encontrar:

- Páginas de presentación  
- Paneles de registro o administración  
- Secciones de descargas  
- Rutas no documentadas o restringidas  

El fuzzing permite identificar rutas que quizás **no deberían ser visibles** o que contengan errores que podrían ser **explotables**.

---
<a name="herramientas-para-fuzzing"></a>
## 🛠 Herramientas para Fuzzing
<a name="feroxbuster"></a>
### 🔹 Feroxbuster

#### 📦 Instalación

# Instalación desde repositorios:
```bash
sudo apt update && sudo apt install -y feroxbuster
```
# Si ese método no funciona, usa el script oficial:
```bash
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | bash
```

🚀 Ventaja principal
Feroxbuster hace fuzzing en varias capas:
Si encuentra una ruta nueva, continuará explorando dentro de ella automáticamente.

⚙️ Uso básico

```bash
feroxbuster --url "http://<IP>" -w <DICCIONARIO> <OPCIONES>
```
📌 ejemplo:
```
feroxbuster --url "http://172.17.0.2/logs" -w /opt/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  -x php,txt,html,zip,log,bin   
```
🔧 Opciones útiles
Extensiones:
```bash
-x php,txt,html,zip,log,bin
```
Encabezados personalizados (por ejemplo, para APIs):
```bash
-H "Accept: application/json" -H "Authorization: Bearer {token}"
```
Para usar con Burp Suite como proxy:
```bash
feroxbuster -u http://127.0.0.1 --insecure --proxy http://127.0.0.1:8080
```
