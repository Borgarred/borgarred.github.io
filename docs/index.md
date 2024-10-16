# Instalación y configuración de un servidor web Nginx

Primero de todo, actualizaremos e instalaremos el paquete de Nginx:

```bash
sudo apt update
sudo apt install nginx
```

![alt text](/site/assets/images/image-0.png)

### Comprobación de Nginx

Comprobaremos que Nginx está funcionando:

```bash
systemctl status nginx
```

![alt text](/site/assets/images/image-1.png)

### Creación de carpetas del sitio web

Ahora vamos a crear las carpetas que contendrán todos los archivos del sitio web. Accedemos a la carpeta /var/www y creamos la carpeta de nuestro dominio:

```bash
sudo mkdir -p /var/www/nombre_web/html`
```

Dentro de la carpeta /html, tendremos que clonar el repositorio https://github.com/cloudacademy/static-website-example.

Para poder hacerlo, debemos tener ya instalado Git.

![alt text](/site/assets/images/image-2.png)

Ahora haremos que el propietario de esta carpeta y todo lo que haya dentro sea el usuario www-data, que normalmente es el usuario del servicio web:

```bash
sudo chown -R www-data:www-data /var/www/nombre_web/html
```

Le daremos permisos para que no nos dé un error de acceso no autorizado al entrar en el sitio web:

```bash
sudo chmod -R 755 /var/www/nombre_web
```

![alt text](/site/assets/images/image-3.png)

### Comprobación del servidor

Para comprobar que el servidor está funcionando y sirviendo páginas correctamente, accedemos desde nuestro cliente a:

```
http://IP_máquina_virtual
```

Deberá aparecer algo así:
![alt text](/site/assets/images/image-4.png)

### Configuración del bloque de servidor

Para que Nginx presente el contenido de nuestra web, es necesario crear un bloque de servidor con las directivas correctas. En vez de modificar el archivo de configuración predeterminado directamente, crearemos uno nuevo en **/etc/nginx/sites-available/nombre_web:**

Con el comando:

```bash
sudo nano /etc/nginx/sites-available/vuestro_dominio
```

En el editor de configuración pondremos lo siguiente:

![alt text](/site/assets/images/image-5.png)

(Donde la ruta root será la carpeta donde clonamos el repositorio anterior en la que se encuentra nuestro archivo index.html).

Y crearemos un archivo simbólico entre este archivo y el de sitios que están habilitados, para que se dé de alta automáticamente:

```bash
sudo ln -s /etc/nginx/sites-available/nombre_web /etc/nginx/sites-enabled/
```

Reiniciamos el servidor para aplicar la configuración:

```bash
sudo systemctl restart nginx
```

### Comprobación del correcto funcionamiento

Como aún no poseemos un servidor DNS que traduzca los nombres a IPs, debemos hacerlo de forma manual. Vamos a editar el archivo /etc/hosts de nuestra **máquina anfitriona** para que asocie la IP de la máquina virtual a nuestro server_name.

En Windows acceder a:

```makefile
C:\Windows\System32\drivers\etc\hosts
```

Y deberemos añadirle la línea:

```
192.168.X.X nombre_web
```

Donde debéis sustituir la IP por la que tenga vuestra máquina virtual.

![alt text](/site/assets/images/image-6.png)

Podemos comprobar que funcione haciendo ping.

![alt text](/site/assets/images/image-7.png)

Desde el archivo **/var/log/nginx/access.log** de nuestra máquina virtual podemos consultar las peticiones a nuestra web.

![alt text](/site/assets/images/image-8.png)

Desde el archivo **/var/log/nginx/error.log** de nuestra máquina virtual podemos consultar cualquier error que se haya registrado.

![alt text](/site/assets/images/image-9.png)

### Transferir archivos desde la máquina local a la máquina virtual mediante FTP

En primer lugar, lo instalaremos desde los repositorios:

```bash
sudo apt-get update
sudo apt-get install vsftpd
```

Ahora vamos a crear una carpeta en nuestro home en Debian:

```bash
mkdir /home/nombre_usuario/ftp
```

Y ahora crearemos los certificados de seguridad necesarios para aportar la capa de cifrado a nuestra conexión (algo parecido a HTTPS):

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.pem -out /etc/ssl/private/vsftpd.pem
```

![alt text](/site/assets/images/image-10.png)

Y una vez realizados estos pasos, procedemos a realizar la configuración de vsftpd propiamente dicha:

```bash
sudo nano /etc/vsftpd.conf
```

Ahora debemos eliminar las siguientes líneas:

![alt text](/site/assets/images/image-11.png)

Y seguidamente añadiremos las siguientes líneas:

```bash
rsa_cert_file=/etc/ssl/private/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH
local_root=/home/nombre_usuario/ftp
```

![alt text](/site/assets/images/image-12.png)

Tras guardar los cambios, reiniciamos el servicio para que coja la nueva configuración:

```bash
sudo systemctl restart --now vsftpd
```

Ya podemos acceder a nuestro servidor mediante un cliente FTP como por ejemplo Filezilla. Vamos a conectarnos mediante SFTP. Debemos introducir los datos de nuestra máquina y el puerto 22.

![alt text](/site/assets/images/image-13.png)

Al pulsar conexión rápida, nos aparecerá un aviso que debemos aceptar.

A continuación, buscaremos el archivo que queremos subir a nuestro servidor en la pestaña de la izquierda; en este caso, enviaré un archivo .zip llamado “PruebaDAW”.

Para subirlo pulsamos clic derecho y “Subir”.

![alt text](/site/assets/images/image-14.png)

### Añadir certificados SSL y acceder mediante HTTPS

Lo primero será generar los certificados:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

![alt text](/site/assets/images/image-15.png)

Crear un grupo Diffie-Hellman:

```bash
sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048
```

![alt text](/site/assets/images/image-16.png)

Crear una configuración para SSL:

```bash
sudo nano /etc/nginx/snippets/self-signed.conf
```

Y poner lo siguiente:

```nginx
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```

![alt text](/site/assets/images/image-17.png)

Crear una configuración para parámetros SSL:

```bash
sudo nano /etc/nginx/snippets/ssl-params.conf
```

Y añadir lo siguiente:

```nginx
Copiar código
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
ssl_dhparam /etc/nginx/dhparam.pem;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header X-Content-Type-Options nosniff;
add_header X-Frame-Options DENY;
add_header X-XSS-Protection "1; mode=block";
```

![alt text](/site/assets/images/image-18.png)

Editar el archivo de configuración del sitio

```bash
sudo nano /etc/nginx/sites-available/nombre_web
```

![alt text](/site/assets/images/image-19.png)

Habilitar la nueva configuración y reiniciar Nginx.
Crear el enlace simbólico (si aún no está hecho):

```bash
sudo ln -s /etc/nginx/sites-available/nombre_web /etc/nginx/sites-enabled/
```

Reiniciar Nginx:

```bash
sudo systemctl restart nginx
```

Ahora ya podremos acceder a nuestra web buscando:

```arduino
https://nombre_web
```

o

```arduino
http://nombre_web
```

![alt text](/site/assets/images/image-20.png)

# Práctica 2.2 – Autenticación en Nginx

### Paquetes necesarios

Para esta práctica podemos utilizar la herramienta openssl para crear las contraseñas.

En primer lugar debemos comprobar si el paquete está instalado:

```
dpkg -l | grep openssl
```

Y si no lo estuviera, instalarlo.
