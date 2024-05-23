# PyRed ![pyred](https://github.com/TBrux/DOCKERLABS/assets/168732212/581d6970-5f9f-4693-915d-8e2cd06b14cb)


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
5000/tcp open  upnp    syn-ack ttl 64
```

Como tenemos abierto el puertos 5000, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p5000 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p5000`: Escaneo de los puertos 5000.
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```
PORT     STATE SERVICE REASON         VERSION
5000/tcp open  upnp?   syn-ack ttl 64
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.2 Python/3.12.2
```
