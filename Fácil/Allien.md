
# Allien
![image](https://github.com/user-attachments/assets/7fcabe7e-bd29-4322-916d-3478628d8599)


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
sudo nmap -sSCV -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
```
```
- `-sSCV`: Realiza un escaneo SYN, de versiones y vulnerabilidades.
- `-p-`: Escanea todos los puertos.
- `--open`: Muestra solo los puertos abiertos.
- `--min-rate 5000`: Establece la velocidad mínima de envío de paquetes a 5000 por segundo.
- `-vvv`: Genera una salida muy detallada y verbosa.
- `-n`: Evita la resolución de DNS inversa.
- `-Pn`: Omite la detección de hosts.
- `-oG allPorts`: Guarda los resultados en un archivo llamado "allPorts".
- `172.17.0.2`: La dirección IP del host objetivo.
```


```bash
/bin/bash -p
```
![image](https://github.com/user-attachments/assets/e42da890-b5cf-497b-aa4c-70011c2c296e)
