
# BreakMySSH

![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/dockerInicio.png)



## Escaneo Inicial

Iniciamos la máquina con un ping para verificar que está activa:

```bash
ping -c 1 172.17.0.2
```

![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/pingBreak.png)



Vemos que la máquina responde correctamente.

### Escaneo de Puertos con Nmap

Utilizamos `nmap` para identificar los puertos abiertos:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```
![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/nmapBreak.png)



#### Explicación de los parámetros:
- `sudo` → Ejecuta con permisos de superusuario.
- `nmap` → Herramienta para escanear redes y puertos.
- `-p-` → Escanea todos los puertos (1-65535).
- `--open` → Muestra solo los puertos abiertos.
- `-sS` → Escaneo SYN (sigiloso y rápido).
- `--min-rate 5000` → Envía al menos 5000 paquetes por segundo (acelera el escaneo).
- `-vvv` → Muestra más detalles en la salida.
- `-n` → No resuelve nombres de dominio (más rápido).
- `-Pn` → No hace ping, asume que el host está activo.
- `172.17.0.2` → IP del objetivo.

El escaneo revela que solo el puerto `22` (SSH) está abierto.

## Ataque de Fuerza Bruta con Hydra

Como solo tenemos SSH, intentaremos un ataque de fuerza bruta con `Hydra`.

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```


![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/hydraBreak.png)



#### Explicación de los parámetros:
- `hydra` → Ejecuta la herramienta Hydra.
- `-l root` → Usa `root` como usuario.
- `-P /usr/share/wordlists/rockyou.txt` → Usa el diccionario `rockyou.txt`.
- `ssh://172.17.0.2` → Ataca el servicio SSH en la IP objetivo.
- `-t 4` → Usa 4 intentos simultáneos.

Si no tienes el usuario, puedes probar con nombres comunes como `admin`, `root`, `user`, `test`.

El ataque nos devuelve la contraseña: **estrella**.

## Acceso al Sistema

Nos conectamos con `ssh` usando la contraseña obtenida:

```bash
ssh root@172.17.0.2
```

Ingresamos la contraseña y logramos acceso como `root`.


![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/rootBreak.png)


Hemos logrado acceso explotando un SSH con credenciales débiles.

### Recomendaciones de Seguridad:
- Usar autenticación con clave pública y deshabilitar el acceso por contraseña.
- Restringir el acceso a SSH solo desde IPs autorizadas.
- Implementar mecanismos de detección de intentos de fuerza bruta (fail2ban, SSHGuard).
- Usar puertos no convencionales para SSH(22).

--- 
