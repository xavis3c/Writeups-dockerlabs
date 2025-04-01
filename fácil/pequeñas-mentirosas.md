

# Máquina Pequeñas Mentirosas

![inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicio.png)


## Escaneo de puertos

Iniciamos la máquina con un escaneo Nmap para identificar puertos abiertos:

```bash
sudo nmap -p- --open -sS -sV -sC --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

El resultado muestra que tenemos dos puertos abiertos: el puerto 22 (SSH) y el puerto 80 (HTTP).

## Navegación web

Al acceder a la web por el puerto 80, encontramos una pista que menciona un usuario llamado 'a'. Con esta información, vamos a intentar ingresar al sistema utilizando **Hydra** para un ataque de fuerza bruta por el puerto 22 (SSH).

## Uso de Hydra para SSH

Usamos Hydra con el siguiente comando para atacar el servicio SSH y obtener la contraseña del usuario 'a':

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

Hydra nos devuelve la contraseña para el usuario 'a', que es: **secret**.

## Conexión SSH

Con la contraseña obtenida, nos conectamos al sistema como el usuario 'a':

```bash
ssh a@172.17.0.2
```

Ahora estamos conectados como el usuario 'a'. Para explorar el sistema, vamos a revisar los archivos del sistema.

## Exploración de archivos

Utilizamos el siguiente comando para navegar al directorio `/srv/ftp`:

```bash
cd /srv/ftp
```

Esta ruta es comúnmente utilizada en sistemas Linux para almacenar archivos relacionados con el servicio FTP. En esta carpeta, podemos encontrar archivos públicos o compartidos. Es importante prestar atención a posibles vulnerabilidades como:

- Archivos mal configurados.
- Acceso anónimo habilitado en el servicio FTP.
- Archivos sensibles olvidados o copias de seguridad.

Dentro de esta carpeta, encontramos un archivo llamado **hash_spencer.txt**, que podría contener un hash de contraseña. Lo revisamos con el siguiente comando:

```bash
cat hash_spencer.txt
```

El contenido del archivo muestra un hash MD5.

## Crackeo del hash

Para obtener la contraseña, utilizamos la página web **CrackStation** y encontramos que el hash MD5 corresponde a la contraseña: **password1**.

## Conexión como Spencer

Con la contraseña obtenida, intentamos iniciar sesión como el usuario **spencer** utilizando el comando:

```bash
su spencer
```

Ingresamos la contraseña **password1** y conseguimos acceso al usuario **spencer**.

## Comprobación de privilegios

Ahora, para determinar si tenemos privilegios elevados, ejecutamos el siguiente comando:

```bash
sudo -l
```

Observamos que tenemos permisos para ejecutar **python3** como superusuario.

## Escalada de privilegios

Utilizamos la pagina de **GTFBins** para encontrar una forma de escalar privilegios utilizando el binario de Python. GTFBins nos proporciona el siguiente comando:

```bash
sudo python -c 'import os; os.system("/bin/sh")'
```

Al ejecutarlo, obtenemos acceso a un shell con privilegios de root. Finalmente, ejecutamos el comando **whoami** para confirmar que somos **root**.

```bash
whoami
```

¡Listo! Hemos conseguido escalada de privilegios y ahora somos **root** en el sistema.


⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/
