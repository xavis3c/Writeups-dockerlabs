
# BorazuwarahCTF

![inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicioBora.png)


## Escaneo de Puertos con Nmap

Lo primero que hacemos es escanear todos los puertos con Nmap para ver qué servicios están corriendo en la máquina.

```bash
sudo nmap -p- --open -sS --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

El escaneo nos muestra que hay dos puertos abiertos:

- **22** (SSH)
- **80** (HTTP)

## Exploración del Sitio Web

Abrimos el navegador y vamos al puerto 80. Lo primero que vemos es una imagen de un huevo Kinder Sorpresa. Esto nos da la idea de que algo puede estar oculto dentro de la imagen.

Descargamos la imagen.

## Análisis de Metadatos con ExifTool

Para ver si la imagen tiene información oculta, usamos `exiftool`:

```bash
exiftool imagen.jpg
```

Esto nos devuelve un dato interesante: un nombre de usuario **borazuwarah**.

## Ataque de Fuerza Bruta con Hydra

Ahora que tenemos un usuario, intentamos obtener la contraseña con `hydra` usando el diccionario `rockyou.txt`.

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

Después de unos segundos, obtenemos la contraseña: **123456** (Hydra la encontro rapido).

## Acceso por SSH

Nos conectamos usando las credenciales obtenidas:

```bash
ssh borazuwarah@172.17.0.2
```

Ingresamos la contraseña y ya estamos dentro.

## Escalada de Privilegios

Revisamos los permisos con:

```bash
sudo -l
```

Vemos que el usuario tiene permisos para ejecutar cualquier comando como root, así que simplemente hacemos:

```bash
sudo bash
whoami
```

¡Y listo! Ahora somos `root` y tenemos control total de la máquina. 

---

⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/
