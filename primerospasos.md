---
layout: index
prev: << Introducción a docker
prev_url: /docker/introduccion
next: Ejecutando una aplicación web en Docker >>
next_url: /docker/aplicacionweb
---

# Primeros pasos con Docker

En una <a href="/docker/introduccion">entrada anterior</a>, veíamos los fundamentos de docker, y repasábamos los principales componentes de la arquitectura de docker:</p>
<ul>
<li style="text-align: justify;"><strong>El cliente de Docker</strong> es la principal interfaz de usuario para docker, acepta los comandos del usuario y se comunica con el daemon de docker.</li>
<li style="text-align: justify;"><strong>Imágenes de Docker (Docker Images)</strong>: Las imágenes de Docker son plantillas de solo lectura, es decir, una imagen puede contener el sistema de archivo de un sistema operativo como Debian, pero esto solo nos permitirá crear los contenedores basados en esta configuración. Si hacemos cambios en el contenedor ya lanzado, al detenerlo esto no se verá reflejado en la imagen.</li>
<li style="text-align: justify;"><strong>Registros de Docker (Docker Registries)</strong>: Los registros de Docker guardan las imágenes, estos son repositorios públicos o privados donde podemos subir o descargar imágenes. El registro público del proyecto se llama Docker Hub y es el componente de distribución de Docker.</li>
<li style="text-align: justify;"><strong>Contenedores de Docker (Docker Containers)</strong>: El contenedor de docker aloja todo lo necesario para ejecutar una aplicación. Cada contenedor es creado de una imagen de docker. Cada contenedor es una plataforma aislada.</li>
</ul>
<p><span id="more-1577"></span></p>
<h2>Instalación de docker</h2>
<p style="text-align: justify;">Vamos a instalar docker engine en un equipo con Debian Jessie, para ello, en primer lugar nos aseguramos de instalar el paquete que permite trabajar a la utilidad apt con htpps, y los certificados CA:</p>
<pre>$ apt-get update
$ apt-get install apt-transport-https ca-certificates
</pre>
<p>A continuación añadimos la clave GPG:</p>
<pre>$ apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
</pre>
<p>Y añadimos el nuevo repositorio:</p>
<pre>$ nano /etc/apt/sources.list.d/docker.list
</pre>
<p>Indicando el repositorio para Debian Jessie:</p>
<pre>deb https://apt.dockerproject.org/repo debian-jessie main
</pre>
<p>Y ya estamos en disposición de realizar la instalación:</p>
<pre>$ apt-get update
$ apt-get install docker-engine
</pre>
<pre>$  docker version
Client:
Version:      1.10.0
API version:  1.22
Go version:   go1.5.3
Git commit:   590d5108
Built:        Thu Feb  4 18:16:19 2016
OS/Arch:      linux/amd64

Server:
Version:      1.10.0
API version:  1.22
Go version:   go1.5.3
Git commit:   590d5108
Built:        Thu Feb  4 18:16:19 2016
OS/Arch:      linux/amd64
</pre>
<h3 style="text-align: justify;">Ejecutando docker con un usuario sin privilegios</h3>
<p style="text-align: justify;">El demonio docker cuando se indicia siempre se ejecuta como root y espera una conexión a un socket unix y no a un puerto TCP. Por defecto el socker unix es propiedad del usuario root por lo que debemos usar el cliente docker como root. Para solucionar esto podemos añadir nuestro usuario sin privilegio al grupo docker, todos los usuarios pertenecientes a este grupo tienen permiso de lectura y escritura sobre el socket por lo que podremos conectar al docker engine desde nuestro usuario.</p>
<pre># usermod -a -G docker usuario</pre>
<p>Y ya podemos usar el usuario para utilizar el cliente docker.</p>
<h3>Acceder a docker engine desde otra máquina</h3>
<p>En ocasiones es preferible tener instalado el cliente docker y el demonio en diferentes máquinas, para ello hay que configurar el docker engine para que escuche por un puerto TCP, para ello:</p>
<pre>$ nano /etc/systemd/system/multi-user.target.wants/docker.service</pre>
<pre>...
 ExecStart=/usr/bin/docker daemon -H fd:// -H tcp://0.0.0.0:2376
 ...</pre>
<pre>systemctl daemon-reload
service docker restart</pre>
<p>En la máquina cliente instalamos el cliente docker:</p>
<pre># apt-get install docker.io</pre>
<p>Y podemos utilizarlo indicando la dirección ip y el puerto del docker daemon:</p>
<pre>$ docker -H 192.168.0.100:2376 ps
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES</pre>
<p>En esta entrada vamos a utilizar el cliente docker desde la misma máquina donde tenemos el demonio.</p>
<h2>Nuestro primer contenedor &#8220;Hola Mundo&#8221;</h2>
<p>Para crear nuestro primer contenedor vamos a ejecutar la siguiente instrucción:</p>
<pre><code class="hljs php"><span class="hljs-string">$ docker run ubuntu /bin/echo 'Hello world'
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
8387d9ff0016: Pull complete 
3b52deaaf0ed: Pull complete 
4bd501fad6de: Pull complete 
a3ed95caeb02: Pull complete 
Digest: sha256:457b05828bdb5dcc044d93d042863fba3f2158ae249a6db5ae3934307c757c54
Status: Downloaded newer image for ubuntu:latest
<strong>Hello world</strong>
</span></code></pre>
<p style="text-align: justify;">Con el comando<strong> run</strong> vamos a crear un contenedor donde vamos a ejecutar un comando, en este caso vamos a crear el contenedor a partir de una imagen ubuntu. Como todavía no hemos descargado ninguna imagen del registro docker hub, es necesario que se descargue la  imagen. Si la tenemos ya en nuestro ordenador no será necesario la descarga. Más adelante estudiaremos como funcionan las imágenes en docker. Finalmente indicamos el comando que vamos a ejecutar en el contenedor, en este caso vamos a escribir un &#8220;Hola Mundo&#8221;.</p>
<p style="text-align: justify;">Por lo tanto, cuando se finaliza la descarga de la imagen, se crea un contenedor a partir de la imagen y se ejecuta el comando que hemos indicado, podemos ver la salida en pantalla. Una vez que se ejecuta la instrucción el contenedor se detiene (stop), podemos ver la lista de contenedores detenidos con el siguiente comando:</p>
<pre>$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS                  NAMES
b0ca02c7b956        ubuntu              "/bin/echo 'Hello wor"   About a minute ago   Exited (0) About a minute ago                          boring_jennings</pre>
<h2>Ejecutando un contenedor interactivo</h2>
<p style="text-align: justify;">En este caso usamos la opción <strong>-i</strong> para abrir una sesión interactiva, <strong>-t</strong> nos permite crear un pseoude-terminal que nos va a permitir interaccionar con el contenedor, indicamos un nombre del contenedor con la opción <strong>&#8211;name</strong>, y la imagen que vamos a utilizar para crearlo, en este caso &#8220;ubuntu&#8221;,  y por último el comando que vamos a ejecutar, en este caso <code>/bin/bash</code>, que lanzará una sesión bash en el contenedor:</p>
<pre>$ docker run -i -t --name contenedor1 ubuntu /bin/bash 
root@2bfa404bace0:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@2bfa404bace0:/# exit
$</pre>
<p style="text-align: justify;">Como podemos comprobar podemos ejecutar distintos comandos dentro del contenedor, por ejemplo hemos visto el árbol de directorios. Cuando salimos de la sesión, el contenedor se detiene. De nuevo podemos ver los contenedores detenidos:</p>
<pre>$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS                  NAMES
2bfa404bace0        ubuntu              "/bin/bash"              13 minutes ago      Exited (0) 9 minutes ago                             contenedor1
b0ca02c7b956        ubuntu              "/bin/echo 'Hello wor"   21 minutes ago      Exited (0) 15 minutes ago                            boring_jennings
</pre>
<p>A continuación vamos a iniciar el contenedor y nos vamos a conectar a él:</p>
<pre>$ docker start contendor1
contendor1
$ docker attach contendor1
root@2bfa404bace0:/#
</pre>
<p>Para obtener información del contenedor:</p>
<pre>$ docker inspect contenedor1
[
    {
        "Id": "2bfa404bace09e244df4528e41f94523dcaa4f8ddeae992259fde0d2151eea03",
        "Created": "2016-02-08T21:14:56.157787821Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2016-02-08T21:14:56.40323815Z",
            "FinishedAt": "2016-02-08T21:18:37.272925413Z"
        },
        "Image": "sha256:3876b81b5a81678926c601cd842040a69eb0456d295cd395e7a895a416cf7d91",
        "ResolvConfPath": "/var/lib/docker/containers/2bfa404bace09e244df4528e41f94523dcaa4f8ddeae992259fde0d2151eea03/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/2bfa404bace09e244df4528e41f94523dcaa4f8ddeae992259fde0d2151eea03/hostname",
        "HostsPath": "/var/lib/docker/containers/2bfa404bace09e244df4528e41f94523dcaa4f8ddeae992259fde0d2151eea03/hosts",
        "LogPath": "/var/lib/docker/containers/2bfa404bace09e244df4528e41f94523dcaa4f8ddeae992259fde0d2151eea03/2bfa404bace09e244df4528e41f94523dcaa4f8ddeae992259fde0d2151eea03-json.log",
        "Name": "/contenedor1",
       ...
</pre>
<h2>Creando un contenedor demonio</h2>
<p style="text-align: justify;">Crear un contenedor que ejecute una sola instrucción y que luego se pare no es muy útil, a continuación vamos a crear un contenedor que funcione como un demonio y este continuamente ejecutándose.</p>
<pre>$ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
7b6c3b1c0d650445b35a1107ac54610b65a03eda7e4b730ae33bf240982bba08
</pre>
<p style="text-align: justify;">En esta ocasión hemos utilizado la opción <strong>-d</strong> del comando <strong>run</strong>, para la ejecución el comando en el contenedor se haga en segundo plano. La salida que hemos obtenido el el <em>ID</em> del contenedor que se está ejecutando, aunque cuando visualizamos los contenedores en ejecución sólo vemos un <em>ID</em> corto:</p>
<pre>$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
7b6c3b1c0d65        ubuntu              "/bin/sh -c 'while tr"   2 minutes ago       Up 2 minutes                            happy_euclid
</pre>
<p>Podemos ver la salida del contenedor accediendo a los logs del contenedor, indicando el id o el nombre que se ha asignado:</p>
<pre>$ docker logs happy_euclid
hello world
hello world
hello world
hello world
...
</pre>
<p>Por último podemos parar el contenedor y borrarlo con las siguientes instrucciones:</p>
<pre>$ docker stop happy_euclid
$ docker rm happy_euclid
</pre>
<h2>Conclusiones</h2>
<p style="text-align: justify;">En esta primera entrada hemos hecho una introducción a la instalación de docker y hemos comenzado a crear contenedores. Realmente estos contenedores no nos sirven de mucho ya que ejecutan instrucciones muy básicas. En la próxima entrada vamos a trabajar com imágenes docker y vamos a profundizar en la creación de contenedores para ejecutar aplicaciones web.</p>
