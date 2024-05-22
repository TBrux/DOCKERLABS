# Move

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

Como tenemos abierto los puertos 22 y 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

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
Vemos que podemos descarganos un archivo de una base de datos de keepass. Nos lo descargamos y probamos a utilizarlo con **keepass2** pero tiene contraseña, así que vamos a ver que encontramos por el puerto 80.
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




