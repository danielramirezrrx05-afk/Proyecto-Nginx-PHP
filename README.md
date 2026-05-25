# Proyecto-Nginx-PHP

#Carrera: 
Ingeniería en Sistemas Computacionales

#Asignatura:
Taller de sistemas operativos

#Profesor:
Gustavo Moisés Romero González

#Integrantes:
-RAMIREZ RIVERA JOSIAS DANIEL
-VAZQUEZ BARRANCO BRAYAN DAVID
-VIELMA SANCHEZ ESMERALDA

#Grupo:
6S12
Tema:
“SERVIDOR NGINX”

#Fecha de entrega:
25/MAYO/2026


------------------------------------------------------------------------------------------------------------------

# Objetivo General

Implementar un servidor web NGINX compilado desde código fuente junto con PHP-FPM utilizando comunicación FastCGI mediante socket UNIX, configurando servicios administrados mediante SystemD para garantizar el correcto funcionamiento y autoarranque del servidor dentro de un sistema Linux.

------------------------------------------------------------------------------------------------------------------

# Objetivos Específicos

* Compilar e instalar NGINX versión 1.31.x desde código fuente.
* Configurar el prefijo de instalación en `/srv/nginx`.
* Crear usuarios y grupos específicos para los servicios NGINX y PHP.
* Compilar e instalar PHP versión 8.4.x con soporte php-fpm.
* Habilitar módulos necesarios para procesamiento de imágenes, fechas e internacionalización.
* Configurar comunicación FastCGI mediante socket UNIX.
* Implementar servicios SystemD para NGINX y PHP-FPM.
* Configurar el autoarranque de los servicios en `multi-user.target`.
* Verificar el funcionamiento mediante una página `phpinfo.php`.


------------------------------------------------------------------------------------------------------------------

# Desarrollo del Proyecto

## Instalación de Dependencias

Se instalaron las librerías necesarias para la compilación de NGINX y PHP desde código fuente.

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential gcc make wget tar unzip \
libpcre3 libpcre3-dev zlib1g zlib1g-dev \
libssl-dev libxml2-dev libsqlite3-dev \
libcurl4-openssl-dev libpng-dev libjpeg-dev \
libfreetype6-dev libonig-dev libzip-dev \
libicu-dev pkg-config -y
```


------------------------------------------------------------------------------------------------------------------


## Creación de Usuarios y Grupos

Se crearon los usuarios y grupos necesarios para ejecutar los servicios.

```bash
sudo groupadd nginx
sudo useradd -r -g nginx -s /usr/sbin/nologin nginx

sudo useradd -r -g nginx -s /usr/sbin/nologin php
```


------------------------------------------------------------------------------------------------------------------


## Descarga y Compilación de NGINX

Se descargó NGINX versión 1.31.x desde el sitio oficial.

```bash
wget https://nginx.org/download/nginx-1.31.0.tar.gz

tar -xzf nginx-1.31.0.tar.gz
cd nginx-1.31.0
```

Configuración de compilación:

```bash
./configure \
--prefix=/srv/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module
```

Compilación e instalación:

```bash
make
sudo make install
```

Verificación de versión:

```bash
/srv/nginx/sbin/nginx -v
```


------------------------------------------------------------------------------------------------------------------


## Servicio SystemD para NGINX

Archivo `/etc/systemd/system/nginx.service`

```ini
[Unit]
Description=NGINX Web Server
After=network.target

[Service]
Type=forking
ExecStart=/srv/nginx/sbin/nginx
ExecReload=/srv/nginx/sbin/nginx -s reload
ExecStop=/srv/nginx/sbin/nginx -s quit
PIDFile=/srv/nginx/logs/nginx.pid
User=nginx
Group=nginx

[Install]
WantedBy=multi-user.target
```

Habilitación del servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable nginx
sudo systemctl start nginx
```


------------------------------------------------------------------------------------------------------------------


## Descarga y Compilación de PHP

Descarga de PHP versión 8.4.x:

```bash
wget https://www.php.net/distributions/php-8.4.0.tar.gz

tar -xzf php-8.4.0.tar.gz
cd php-8.4.0
```

Configuración:

```bash
./configure \
--prefix=/srv/nginx \
--enable-fpm \
--with-fpm-user=php \
--with-fpm-group=nginx \
--with-zlib \
--with-curl \
--with-openssl \
--enable-mbstring \
--enable-intl \
--with-jpeg \
--with-freetype
```

Compilación e instalación:

```bash
make
sudo make install
```

Verificación de versión:

```bash
/srv/nginx/bin/php -v
```


------------------------------------------------------------------------------------------------------------------


## Configuración de PHP-FPM

Archivo de configuración principal:

```bash
sudo cp php.ini-production /srv/nginx/lib/php.ini
```

Configuración del pool php-fpm:

```ini
listen = /tmp/php84.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
user = php
group = nginx
```


------------------------------------------------------------------------------------------------------------------


## Servicio SystemD para PHP-FPM

Archivo `/etc/systemd/system/php-fpm8.4.service`

```ini
[Unit]
Description=PHP 8.4 FastCGI Process Manager
After=network.target

[Service]
Type=simple
ExecStart=/srv/nginx/sbin/php-fpm
ExecReload=/bin/kill -USR2 $MAINPID
User=php
Group=nginx

[Install]
WantedBy=multi-user.target
```

Habilitación del servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable php-fpm8.4
sudo systemctl start php-fpm8.4
```


------------------------------------------------------------------------------------------------------------------


## Configuración FastCGI entre NGINX y PHP-FPM

Archivo de configuración de NGINX:

```nginx
location ~ \.php$ {
    root html;
    fastcgi_pass unix:/tmp/php84.sock;
    fastcgi_index index.php;
    include fastcgi.conf;
}
```


------------------------------------------------------------------------------------------------------------------


## Creación del Archivo de Prueba PHP

```bash
cd /srv/nginx/html
sudo nano phpinfo.php
```

Contenido del archivo:

```php
<?php
phpinfo();
?>
```

Prueba en navegador:

```text
http://localhost/phpinfo.php
```


------------------------------------------------------------------------------------------------------------------


## Verificación de Servicios

Estado de NGINX:

```bash
systemctl status nginx
```

Estado de PHP-FPM:

```bash
systemctl status php-fpm8.4
```

Verificación del socket UNIX:

```bash
ls -l /tmp/php84.sock
```

Verificación de autoarranque:

```bash
systemctl is-enabled nginx
systemctl is-enabled php-fpm8.4
```

------------------------------------------------------------------------------------------------------------------



# Conclusiones

Durante el desarrollo del proyecto se logró implementar correctamente un servidor NGINX compilado desde código fuente junto con PHP-FPM utilizando comunicación FastCGI mediante socket UNIX. También se configuraron servicios administrados mediante SystemD para garantizar el arranque automático del servidor y se verificó el correcto funcionamiento mediante pruebas realizadas con el archivo `phpinfo.php`. Este proyecto permitió comprender el proceso de compilación manual de servicios web, la administración de procesos y la integración de tecnologías en sistemas Linux.

------------------------------------------------------------------------------------------------------------------
## Referencias

--NGINX. (2026). NGINX Documentation. 
https://nginx.org/

--PHP Group. (2026). PHP Documentation.
https://www.php.net/docs.php

--AlmaLinux OS Foundation. (2026). AlmaLinux Documentation. 
https://wiki.almalinux.org/




