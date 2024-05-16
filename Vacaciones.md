# Vacaciones

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

Como tenemos abierto los puerto 22 y 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

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
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 41:16:eb:54:64:34:d1:69:ee:dc:d9:21:9c:72:a5:c1 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzT6jdfo9QUX+9zCmyJQNTcAJXdhXByneCfqA9I7cXPBFGDGgxNAfQdoiqH3EMiTjf+maPlCNyVHGFl+sClQa5sJwdrbWZiJPxfxGkCtWiSrRdKKUKt/7rCMKMOy79bFRvurgss+57tsglfXkE9FPkZGd3mLruXt5Lyb+8uhFWpW58Df6ZUoSsJi7n0bkXNpEzJAzYHNmRRtv0RsGDFosi/t5KUCMPX67jbM8jsApIVwFIQBTiwzwGQn33G2ZoAJy/NYZ9dkuN2cKM2uItovo25daA+0/SxEfHqAHGquvoMKSj8pcX3qZVD7cGWlsn9c5QNzHRC2DZUSHrK7UIaG0r
|   256 f0:c4:2b:02:50:3a:49:a7:a2:34:b8:09:61:fd:2c:6d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMD2Z/ZotorXbs6zP9Sg9XenjSX0HIjYjoEH2cAV7aDoQXZKrssz5AJ98j8b4ntOPGfVehrcRv9X7lKswOea9HM=
|   256 df:e9:46:31:9a:ef:0d:81:31:1f:77:e4:29:f5:c9:88 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK/0ZadHoPSGKg31xFAhPaX854MMS09s5JgdzqmD3jCl
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Como no tenemos ningún usuario todavía vamos a ver qué tenemos en la página http://172.17.0.2 y a simple vista no tenemos nada pero si miramos el código fuente vemos un mensaje.
```
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```
Ya tenemos dos posibles usuarios, como parece que somos Camilo vamos a aprovechar el puerto 22 (SSH) para con hydra y fuerza bruta intentar descubrir la contraseña.
```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
