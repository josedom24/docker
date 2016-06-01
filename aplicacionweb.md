---
layout: index
prev: << Primeros pasos con Docker
prev_url: /docker/primerospasos
---

<p style="text-align: justify;"><a class="thumbnail" href="http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/engine.png" rel="attachment wp-att-1601"><img class="wp-image-1601 alignright" src="http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/engine.png" alt="engine" width="327" height="206" srcset="http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/engine.png 540w, http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/engine-300x189.png 300w" sizes="(max-width: 327px) 100vw, 327px" /></a></p>
<p style="text-align: justify;">Seguimos profundizando en el uso de contenedores con docker. En la <a href="/docker/premerospasos">pasada entrada</a>, hicimos una introducción al uso de docker creando nuestros primeros contenedores, en esta entrada vamos a profundizar en la gestión de imágenes docker y en la creación de un contenedor que nos ofrezca un servicio, más concretamente que ejecute un servidor web y que nos ofrezcan una página web estática.</p>
<h2>Trabajando con imágenes</h2>
<p style="text-align: justify;">Vamos a descargar una imagen del sistema operativo GNU/Linux Debian del registro público <strong>docker hub</strong>. Normalmente el nombre de las imágenes tienen la forma <em>usuario/nombre:etiqueta</em>, si no indicamos la etiqueta será <strong><em>latest</em></strong>. Por ejemplo el nombre de una imagen puede ser <strong>nuagebec/ubuntu:15.04</strong></p>
<pre>$ docker pull debian
 Using default tag: latest
 latest: Pulling from library/debian
 03e1855d4f31: Pull complete
 a3ed95caeb02: Pull complete
 Digest: sha256:d2ea9df44c61c1e3042c20dd42bf57a86bd48bb428e154bdd1d1003fad6810a4
 Status: Downloaded newer image for debian:latest</pre>
<p style="text-align: justify;">Como podemos ver cada imagen está formado por distintas capas, que corresponden a parte del sistema de fichero que forman la imagen. Las capas que son comunes a varias imágenes se comparten entre ellas y por lo tanto no es necesario bajarlas. Vamos a bajar otra imagen:</p>
<pre>$ docker pull ubuntu:14.04</pre>
<p>Y podemos ver todas las imágenes que tenemos en nuestro equipo local con:</p>
<pre>$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 debian              latest              9a02f494bef8        12 days ago         125.1 MB
 ubuntu              14.04               3876b81b5a81        2 weeks ago         187.9 MB</pre>
<p><span id="more-1592"></span>Para obtener información sobre una imagen, ejecutamos el siguiente comando:</p>
<pre>docker inspect debian
 [
 {
 "Id": "sha256:9a02f494bef8d0d088ee7533aa1ba4aaa1dbf38a97192d36fa79a51279bc04de",
 "RepoTags": [
 "debian:latest"
 ],
 "RepoDigests": [],
 "Parent": "",
 "Comment": "",
 "Created": "2016-01-25T22:24:37.914712562Z",
 "Container": "c59024072143b04b79ac341c51571fc698636e01c13b49c523309c84af4b70fe",
 "ContainerConfig": {
 "Hostname": "e06f5a03fe1f",
 "Domainname": "",
 "User": "",
 "AttachStdin": false,
 "AttachStdout": false,
 "AttachStderr": false,
 "Tty": false,
 "OpenStdin": false,
 "StdinOnce": false,
 "Env": null,
 "Cmd": [
 "/bin/sh",
 "-c",
 "#(nop) CMD [\"/bin/bash\"]"
 ],
...</pre>
<p style="text-align: justify;">Para buscar imágenes en <strong>docker hub</strong>, podemos utilizar la siguiente instrucción:</p>
<pre>$ docker search debian
 NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
 ubuntu                         Ubuntu is a Debian-based Linux operating s...   3152      [OK]
 debian                         Debian is a Linux distribution that's comp...   1103      [OK]
 neurodebian                    NeuroDebian provides neuroscience research...   15        [OK]
 jesselang/debian-vagrant       Stock Debian Images made Vagrant-friendly ...   7                    [OK]
 armbuild/debian                ARMHF port of debian                            6                    [OK]
 mschuerig/debian-subsonic      Subsonic 5.1 on Debian/wheezy.                  4                    [OK]
 eboraas/debian                 Debian base images, for all currently-avai...   3                    [OK]
 datenbetrieb/debian            minor adaption of official upstream debian...   1                    [OK]
 maxexcloo/debian               Docker base image built on Debian with Sup...   1                    [OK]
 reinblau/debian                Debian with usefully default packages for ...   1                    [OK]
 webhippie/debian               Docker images for debian                        1                    [OK]
 eeacms/debian                  Docker image for Debian to be used with EE...   1                    [OK]
 lephare/debian                 Base debian images                              1                    [OK]
 lucasbarros/debian             Basic image based on Debian                     1                    [OK]
 servivum/debian                Debian Docker Base Image with Useful Tools      1                    [OK]
 fike/debian                    Debian Images with language locale installed.   0                    [OK]
 nimmis/debian                  This is different version of Debian with a...   0                    [OK]
 visono/debian                  Docker base image of debian 7 with tools i...   0                    [OK]
 opennsm/debian                 Lightly modified Debian images for OpenNSM      0                    [OK]
 thrift/debian                  build/docker/debian                             0                    [OK]
 maticmeznar/debian             A nice Debian template.                         0                    [OK]
 pl31/debian                    Debian base image.                              0                    [OK]
 mariorez/debian                Debian Containers for PHP Projects              0                    [OK]
 icedream/debian-jenkinsslave   Debian for Jenkins to be used as slaves.        0                    [OK]
 konstruktoid/debian            Debian base image                               0                    [OK]</pre>
<p>Y para borrar una imagen:</p>
<pre>docker rmi ubuntu:14.04</pre>
<h2>&#8220;Dockerizando&#8221; un servidor web apache</h2>
<p style="text-align: justify;">Como primera aproximación para crear una imagen desde la que podamos crear un contenedor que ofrezca un servicio, como, por ejemplo, un servidor web, daremos los siguientes pasos:</p>
<ul>
<li style="text-align: justify;">Crearemos un contenedor desde una imagen base, por ejemplo desde la imagen &#8220;debian&#8221;.</li>
<li style="text-align: justify;">Accederemos a ese contenedor e instalaremos un servidor web apache2.</li>
<li style="text-align: justify;">Crearemos una nueva imagen a partir de este contenedor que nos permitirá crear nuevos contenedores con el servidor web instalado.</li>
</ul>
<p style="text-align: justify;">Hay que indicar que el método que vamos a utilizar no es el que se usa habitualmente y que en la próxima entrada estudiaremos el procedimiento habitual para conseguirlo, que será generar directamente una imagen con un servidor web instalado utilizando para ello un fichero <strong>Dockerfile </strong>y el comando <em>docker build</em>. De todas maneras vamos a mostrar este ejemplo que nos puede ser muy útil para seguir profundizando en el funcionamiento de docker.</p>
<p style="text-align: justify;">En primer lugar vamos a crear un contenedor desde la imagen &#8220;debian&#8221; que habíamos descargado, para ello vamos a ejecutar la siguiente instrucción:</p>
<pre>$ docker run -i -t --name primero debian /bin/bash
root@61ee5966f2f3:/# apt-get update
root@61ee5966f2f3:/# apt-get install apache2
root@61ee5966f2f3:/#exit</pre>
<p style="text-align: justify;">Como vemos, hemos instalado en el contenedor el servidor web apache. Cuando se crea un contenedor, su sistema de fichero estará formado por la unión de las capas de la imagen origen, es decir tendrá el sistema de ficheros que tiene la imagen. Estas capas que forman el sistema de archivo del contenedor tienen dos características: son compartidas entre contenedores distintos que se crean desde la misma imagen, y son de sólo lectura, no se pueden modificar, por lo que cuando se crea un nuevo contenedor se añade una nueva capa de lectura y escritura donde se va registrando la creación de los nuevos ficheros y la modificación y borrado de los ficheros existentes.</p>
<p style="text-align: justify;">Por lo tanto el sistema de archivo de nuestro contenedor estará formado por tres capas: las dos originales de la imagen &#8220;debian&#8221;, y una tercera donde hemos creado los ficheros al instalar el servidor web. Cuando salimos de un contenedor de una sesión interactiva el contenedor se detiene (stop). Podemos ver la lista de contenedores detenidos:</p>
<pre>$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
 61ee5966f2f3        debian              "/bin/bash"         8 minutes ago       Exited (0) 3 seconds ago                       primero</pre>
<p style="text-align: justify;">Una vez que hemos hecho una modificación en nuestro contenedor, si queremos utilizar el servidor web deberíamos crear una nueva imagen a partir del contenedor, para ello:</p>
<pre>$ docker commit -m "Añadido apache" -a "José Domingo" 61ee5966f2f3 jose/apache:v1</pre>
<p style="text-align: justify;">Con <strong>commit</strong> vamos a crear una nueva imagen en nuestro repositorio local, se indica un comentario con la opción <strong>-m</strong>, el autor con la opción <strong>-a</strong>, el identificador del contenedor y finalmente el nombre de la nueva imagen.</p>
<pre>$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 jose/apache         v1                  81aeeac1781a        9 seconds ago       193.3 MB
 debian              latest              9a02f494bef8        12 days ago         125.1 MB</pre>
<h2 style="text-align: justify;">Creando un nuevo contenedor con el servidor web</h2>
<p style="text-align: justify;">Ahora vamos a crear un nuevo contenedor a partir de la imagen que hemos creado anteriormente: <em><strong>jose/apache:v1</strong></em>. Este contenedor lo vamos a crear usando la opción <strong>-d</strong> que nos permite &#8220;<em>demonizar</em>&#8221; el contenedor, es decir que se ejecute indefinidamente, para ello:</p>
<pre>$ docker run -d -p 8000:80 --name segundo jose/apache:v1 /usr/sbin/apache2ctl -D FOREGROUND
 85218e86d6e48f360eb1c6e49c62da5e4ffd94b00308734e84f0253e72ec647d</pre>
<pre>$ docker ps
 CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
 85218e86d6e4        jose/apache:v1      "/usr/sbin/apache2ctl"   3 seconds ago       Up 2 seconds        0.0.0.0:8000-&gt;80/tcp   segundo</pre>
<p style="text-align: justify;">Vemos que el contenedor se está ejecutando, además con la opción <strong>-p</strong> mapeamos un puerto del equipo donde tenemos instalado el docker engine, con un puerto del contenedor. Además la instrucción que se ejecuta en el contenedor es la que nos permite ejecutar apache2 en segundo plano. En este caso podemos ver que accediendo al puerto 8000 de nuestro servidor docker accederemos al puerto 80 del contenedor.</p>
<p style="text-align: justify;"><a class="thumbnail" href="http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/docker1.png" rel="attachment wp-att-1579"><img class="size-full wp-image-1579 aligncenter" src="http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/docker1.png" alt="docker1" width="865" height="324" srcset="http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/docker1.png 865w, http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/docker1-300x112.png 300w, http://www.josedomingo.org/pledin/wp-content/uploads/2016/02/docker1-768x288.png 768w" sizes="(max-width: 865px) 100vw, 865px" /></a></p>
<p style="text-align: justify;">Podemos ver los procesos que se están ejecutando en nuestro contenedor:</p>
<pre>$ docker top segundo
 UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
 root                11211               7597                0                   20:49               ?                   00:00:00            /bin/sh /usr/sbin/apache2ctl -D FOREGROUND
 root                11227               11211               0                   20:49               ?                   00:00:00            /usr/sbin/apache2 -D FOREGROUND
 www-data            11228               11227               0                   20:49               ?                   00:00:00            /usr/sbin/apache2 -D FOREGROUND
 www-data            11229               11227               0                   20:49               ?                   00:00:00            /usr/sbin/apache2 -D FOREGROUND</pre>
<p style="text-align: justify;">Para para nuestro contenedor:</p>
<pre>$ docker stop segundo</pre>
<p style="text-align: justify;">Y lo podemos volver a ejecutar:</p>
<pre>$ docker start segundo</pre>
<p style="text-align: justify;">O en un solo paso:</p>
<pre>$ docker restart segundo</pre>
<p style="text-align: justify;">Y finalmente para borrar el contenedor:</p>
<pre>$ docker stop segundo
$ docker rm segundo</pre>
<h2>Conclusiones</h2>
<p style="text-align: justify;">En esta entrada hemos aprendido a gestionar las imágenes que nos podemos descargar del registro <strong>docker hub</strong>, además hemos estudiado la posi