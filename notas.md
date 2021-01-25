# Taller Docker - Notas

## Intro

- Pero porque queremos Docker. Compilar aplicaciones con las mismas características,
  versiones de dependencias incompatibles y que sea fácil desplegar una aplicación.

- Un **contenedor** es un agrupado de procesos los cuales son aislados de
  otros procesos y pueden tener sus propios:
  - usuarios
  - red _namespace_ 
  - _id_ de procesos
  - sistemas de archivos
  - límites de CPU/memoria

- Una **máquina virtual** es una computadora falsa, cada máquina tiene su 
  propio _kernel_

- Algunas ventajas de contenedores: usan menos memoria RAM y HD e incian mucho más rápido 
  que las MV

- Desventajas: son más inseguras, ya que no están aisladas de la máquina host,
  aún cuando puedes pensarlas como una computadora no puedes hacer todo lo que
  harías en una computadora

## Instalación

- WSL2

## Imágenes y contenedores de Docker

- Diferenciar entre estos dos conceptos

## Docker

- Visitar hub.docker.com para ver ngnix

```
docker pull nginx
docker image
docker run nginx:latest
docker container ls
docker run -d nginx:latest
```

<ctrl+c> y después `ls`
otra vez lanzarlo y comparar nombres

- Empezar con el mapeo de puertos
```
docker run -d -p 8080:80 nginx:latest
docker stop ID
docker run -d -p 3000:80 nginx:latest
docker stop
docker run -d -p 3000:80 -p 8080:80 nginx:latest
```

- Docker rm, ps, stop
```
docker ps
docker stop NAME
docker start NAME
docker stop ID
docker ps 
docker ps --help
docker ps -a 
docker rm ID
docker ps -aq
docker rm $(docker -aq)
```
  - `docker ps` solo muestra los contenedores que estan en ejecucion
  - `docker ps -aq` solo muestra los ID
  - Es importante notar que `docker ps -a` muestra tanto contenedores
    corriendo como detenidos
  - No se puede borrar contenedores que se estan ejecutando, pero con
    la opción `-f` es posible.

- Nombrar contenedores
```
docker run -d --name website -p 3000:80 -p 8080:80 nginx:latest
docker stop website
docker start website
```

- Docker ps
```
docker ps
docker run -d --name website-2 -p 9000:80 nginx:latest
```
"ID\t{{.ID}}\nNOMBRE\t{{.Names}}\nIMAGEN\t{{.Image}}\nPUERTOS\t{{.Ports}}\nCOMANDO\t{{.Command}}\nCREADO\t{{.CreatedAt}}\nESTADO\t{{.Status}}\n"
```
docker ps --format ---
export FORMAT="ID\t{{.ID}}\nNOMBRE\t{{.Names}}\nIMAGEN\t{{.Image}}\nPUERTOS\t{{.Ports}}\nCOMANDO\t{{.Command}}\nCREADO\t{{.CreatedAt}}\nESTADO\t{{.Status}}\n"
docker ps --format $FORMAT
```

- Compartir archivos entre host y container

Crear folder y crear archivo `index.html`. Cambiarme al folder de trabajo.

```
docker run -d --name website -p 8080:80 -v $(pwd):/user/share/nginx/html:ro nginx:latest
```

Verificar actualizacion en el _browser_

```
docker exec -ti website bash
```

Analizar archivos, tratar de crear un archivo

```
docker stop website 
docker rm website
docker run -d --name website -p 8080:80 -v $(pwd):/user/share/nginx/html:rw nginx:latest
docker exec -ti website bash
```

Crear archivo about en `/usr/share/nginx/html`.

## Dockerfile

Bajar https://github.com/startbootstrap/startbootstrap-sb-admin-2

```
FROM nginx:latest
ADD . /usr/share/nginx/latest
```

```
docker build --tag jbarrios/website:latest .

docker stop website
docker rm
docker images
docker run --name website -p 8080:80 -d jmbarrios/website:latest
```

## Escribir una api PHP

- Instalar php  y composer

```
composer create-project --prefer-dist laravel/lumen mi_api
```

Editar `routes/web.php`

```
$router->get('/', function () use ($router) {
    return response()->json('Bienvenido a mi API');
});

$router->get('/users/', function() use ($router) {
    return response()->json([
        [
            'name'  => 'Juan',
            'email' => 'juan@example.com'
        ],
        [
            'name'  => 'Pedro',
            'email' => 'pedro@mail.com'
        ]
    ]);
});
```
Se incia con 
```
php -S 0.0.0.0:3000 -t public
```

Buscar documentacion sobre imagen php 
https://hub.docker.com/_/php

Como vamos a usar Apache tenemos que hacer un archivo para
el rewrite de rutas y que las resuelva el `public/index.php`

.htaccess
```
RewriteEngine On
RewriteRule ^(.*)$ public/$1 [L]
```

Dockerfile
```
FROM php:7.4-apache

RUN a2enmod rewrite

COPY mi_api/ /var/www/html/
```

## Instalar composer y utilizar `.gitignore`

Crear el archivo `install-composer.sh` usando las instrucciones de
instalación de la página de composer.

```
#!/bin/bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
```

Agregar a `Dockerfile` las líneas 

```
# Install composer 
ADD install-composer.sh /root/
RUN /root/install-composer.sh
```
Crear el archivo `.dockerignore`

```
mi_api/vendor
```

Correr el contenedor. FALLA

Agregar el paso `composer install` al `Dockerfile`, pero composer necesitará 
`git` y `unzip` por lo que hay que instalarlo.

```
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
        unzip \
        git; \
    rm -rf /var/lib/apt/lists/*
RUN composer install
```

# Usar cache

Nuestro `Dockerfile` se ve así

```
FROM php:7.4-apache

# Install composer 
ADD install-composer.sh /root/
RUN /root/install-composer.sh

COPY mi_api/ /var/www/html/
COPY .htaccess /var/www/html/

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
        unzip \
        git; \
    rm -rf /var/lib/apt/lists/*
RUN composer install

RUN a2enmod rewrite
```

Cambiar `routes/web.php` agregar nuevos nombres. 

Acomodar cosas para que la instalación sea más rápida.

## _Tags_, versionamiento y etiquetado

Discutir que usamos `7.4-apache` aunque si usaramos `apache` tendríamos
php 8.

Cada vez que se construye una imagen con el mismo tag la otra se invalida 
y queda sin name y tag

```
docker tag jmbarrios/mi_api:latest jmbarrios/mi_api:1
```

## Algunos comandos de Docker

- `docker inspect`
- `docker logs`
- `docker exec`
