
# Maquina Tproot

![inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicioTProot.png)


## Iniciamos la máquina

Una vez desplegada la máquina, comenzamos con un escaneo de puertos utilizando **Nmap**:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

### Analizando versiones de servicios

Ejecutamos otro escaneo más detallado para identificar las versiones de los servicios:

```bash
nmap -p21,80 -sC -sV 172.17.0.2
```

### Explicación de las opciones:

- **`-p21,80`** → Escanea solo los puertos **21 (FTP)** y **80 (HTTP)**.
- **`-sC`** → Ejecuta scripts básicos de Nmap (`--script=default`) para obtener más información.
- **`-sV`** → Detecta versiones de los servicios en los puertos abiertos.
- **`172.17.0.2`** → IP del objetivo.

Nos damos cuenta de que los puertos **21 (FTP)** y **80 (HTTP)** están abiertos, por lo que comenzamos revisando el puerto **80**.

Al ingresar la IP en el navegador, encontramos la página por defecto de **Apache2**, lo que no nos aporta mucha información. Ahora nos enfocamos en el **puerto 21 (FTP)**.

---

## Buscando exploits para FTP

Utilizamos `searchsploit` para buscar vulnerabilidades en el servicio **vsftpd 2.3.4**:

```bash
searchsploit vsftpd 2.3.4
```

Entre los resultados, encontramos dos opciones: un exploit en **Python** (`.py`) y otro en **Ruby** (`.rb`). En este caso, elegimos el de **Python**.

Copiamos el exploit a nuestro directorio de trabajo con:

```bash
searchsploit -m unix/remote/49757.py
```

### Explicación:

- **`searchsploit`**: Herramienta para buscar exploits en la base de datos de **Exploit-DB**.
- **`-m`**: Copia el exploit encontrado al directorio actual.
- **`unix/remote/49757.py`**: ID del exploit **49757**, diseñado para sistemas **Unix/Linux**, permitiendo ejecución remota.

Ahora que tenemos el exploit en nuestro directorio, lo ejecutamos contra la máquina objetivo.

---

## Ejecutando el exploit

```bash
python3 49757.py 172.17.0.2
```

### Explicación:

- **`python3`**: Ejecuta el script con Python 3.
- **`49757.py`**: Archivo del exploit copiado anteriormente.
- **`172.17.0.2`**: IP del objetivo.

Dependiendo del exploit, este puede permitirnos ejecutar comandos de manera remota en la máquina objetivo.

Para verificar si tenemos acceso, ejecutamos:

```bash
whoami
```

Ahora somos **root** en la máquina objetivo.

---

### Consideraciones finales

Este ejercicio demuestra cómo se puede identificar y explotar una vulnerabilidad en un sistema desactualizado. Sin embargo, es importante recordar que la explotación de vulnerabilidades sin autorización es **ilegal**. Utiliza estos conocimientos únicamente en entornos de pruebas o con permiso explícito.

Nos vemos en la próxima.

⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/

