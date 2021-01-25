# Ejercicios con docker

## Ejercicio 1: Nuestros primeros contenedores

### Nuestro primer contenedor "Hola Mundo"

    $ docker run ubuntu /bin/echo 'Hello world'
    Unable to find image 'ubuntu:latest' locally
    latest: Pulling from library/ubuntu
    8387d9ff0016: Pull complete 
    ...
    Status: Downloaded newer image for ubuntu:latest
    Hello world

Con el comando `run` vamos a crear un contenedor donde vamos a ejecutar un comando, en este caso vamos a crear el contenedor a partir de una imagen ubuntu. Como todavía no hemos descargado ninguna imagen del registro docker hub, es necesario que se descargue la  imagen. Si la tenemos ya en nuestro ordenador no será necesario la descarga. 

* Comprueba si hay algún contenedor ejecutándose (`docker ps`).
* Comprueba los contenedores que ya se han ejecutado y ahora están parados (`docker ps -a`).
* Compruebas las imágenes de nuestro registro local (`docker images`)

### Ejecutando un contenedor interactivo

En este caso usamos la opción `-i` para abrir una sesión interactiva, `-t` nos permite crear un pseoude-terminal que nos va a permitir interaccionar con el contenedor, indicamos un nombre del contenedor con la opción `--name`, y la imagen que vamos a utilizar para crearlo, en este caso “ubuntu”,  y por último el comando que vamos a ejecutar, en este caso `/bin/bash`, que lanzará una sesión bash en el contenedor:

    $  docker run -it --name contenedor1 ubuntu /bin/bash 
    oot@2bfa404bace0:/#

El contenedor se para cuando salimos de él. Para volver a conectarnos a él:

    $ docker start contendor1
    contendor1
    $ docker attach contendor1
    root@2bfa404bace0:/#

Si el contenedor se está ejecutando podemos ejecutar comando en él con el subcomando `exec`:

    $ docker start contendor1
    contendor1
    $ docker exec contenedor1 ls -al

### Creando un contenedor demonio

En esta ocasión hemos utilizado la opción `-d` del comando `run`, para la ejecución el comando en el contenedor se haga en segundo plano.

    $ docker run -d --name contenedor2 ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
    7b6c3b1c0d650445b35a1107ac54610b65a03eda7e4b730ae33bf240982bba08

* Comprueba que el contenedor se está ejecutando
* Comprueba lo que está haciendo el contenedor (`docker logs contenedor2`)

Por último podemos parar el contenedor y borrarlo con las siguientes instrucciones:

    $ docker stop contenedor2
    $ docker rm contenedor2

### Creando un contenedor con un servidor web

Tenemos muchas imágenes en el registro público **docker hub**, por ejemplo podemos crear un servidor web con apache 2.4:

    $ docker run -d --name my-apache-app -p 8080:80 httpd:2.4

Vemos que el contenedor se está ejecutando, además con la opción `-p` mapeamos un puerto del equipo donde tenemos instalado el docker, con un puerto del contenedor.  Para probarlo accede desde un navegador a la ip del servidor con docker y al puerto 8080.

Para acceder al log del contenedor podemos ejecutar:

    $ docker logs my-apache-app

## Ejercicio 2: Creando nuestras imágenes con Dockerfile

Vamos a desplegar una página estática utilizando un servidor apache 2.4, para ello vamos a crear un directorio `mi_pagina`.Dentro creamos un directorio `public_html` donde vamos a guardar nuestra página:

    $ cd public_html
    echo "<h1>Prueba</h1>" > index.html

En el directorio `mi_pagina` creamos un fichero `Dockerfile` con el siguiente contenido:

    FROM debian
    RUN apt-get update -y && apt-get install -y \
            apache2 \
            && apt-get clean && rm -rf /var/lib/apt/lists/*
    COPY ./public_html /var/www/html/
    ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

Creamos la imagen que hemos definido en el fichero `Dockerfile`:

    ~/mi_pagina$ docker build -t josedom24/aplicacionweb:v1 .

* Comprueba que hemos creado una nueva imagen

Finalmente creamos un contenedor a partir de nuestra nueva imagen:

    $ docker run --name aplweb -d -p 80:80 josedom24/aplicacionweb:v1

## Ejercicio 3: Instalación de wordpress

Lo primero que vamos a hacer es crear un contenedor desde la imagen mariadb con el nombre `servidor_mariadb`, siguiendo las instrucción del <a href="https://hub.docker.com/_/mariadb/">repositorio</a> de docker hub:

    $ docker run --name servidor_mariadb -e MYSQL_ROOT_PASSWORD=asdasd -e MYSQL_DATABASE=wordpress -d mariadb

Hemos indicado la variable de entrono <em>MYSQL_ROOT_PASSWORD</em>, que es obligatoria, indicando la contraseña del usuario root y `MYSQL_DATABASE` donde indicamos el nombre de la base de datos que debe crear. Si seguimos las instrucciones de docker hub podemos observar que podríamos haber creado más variables, por ejemplo: `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_ALLOW_EMPTY_PASSWORD`.

A continuación vamos a crear un nuevo contenedor, con el nombre _servidor_wp_, con el servidor web a partir de la imagen wordpress, enlazado con el contenedor anterior.

    $ docker run --name servidor_wp -p 8000:80 --link servidor_mariadb:mariadb -d wordpress

Para realizar la asociación entre contenedores hemos utilizado el parámetro `--link`, donde se indica el nombre del contenedor enlazado y un alias por el que nos podemos referir a él.

