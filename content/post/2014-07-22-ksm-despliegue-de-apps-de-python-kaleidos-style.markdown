---
author: yamila
comments: false
date: 2014-07-22 14:40:12+00:00
slug: ksm-despliegue-de-apps-de-python-kaleidos-style
title: 'KSM: despliegue de apps de Python (Kaleidos style)'
wordpress_id: 473
categories:
- Open source
- Tutorials
tags:
- kaleidos.net
- my modus
- python
---

Este post es un _paso a paso_ del taller que [Jesús Espino](http://twitter.com/jespinog) nos ha dado en el marco de los [Kaleidos Summer Mondays](http://www.kaleidos.net/blog/346/kaleidos-summer-mondays/): _Despliegue de aplicaciones Python estilo Kaleidos_

Para los ansiosos, aquí están las [transparencias](https://github.com/jespino/taller-despliegue/tree/master/slides).
<!-- more -->
**Pequeño _disclaimer_**: en [Kaleidos](http://kaleidos.net) no nos dedicamos a desplegar aplicaciones o a mantenimientos de despliegues; el objetivo de este taller es poder realizar despliegues para entornos de desarrollo o entornos de producción pequeños.


### Conocimientos previos


Este taller cuenta con que te manejas con soltura en [Python](http://python.org), has hecho tus pinitos con [Django](http://djangoproject.com) y te suena el concepto de "servidor de aplicaciones", ya sea porque alguna vez has desplegado con [Apache](http://apache.org) o porque conoces [Nginx](http://nginx.org).


### Requisitos

* Python3
* [Virtualenvwrapper](http://virtualenvwrapper.readthedocs.org/en/latest/) es un wrapper sobre [Virtualenv](http://virtualenv.readthedocs.org/en/latest/). Es una herramienta para crear entornos de Python aislados del sistema. Los requisitos del despliegue se instalan dentro del virtualenv, y quedan independientes del python del sistema (que puede actualizarse periódicamente). Esto permite tener varios entornos virtuales, cada uno con sus propias dependencias y versiones.
* [Git](http://git-scm.com/) es un sistema de control de versiones. En este taller el uso de Git es más bien anecdótico, así que no te preocupes si no lo conoces bien.

          $ sudo apt-get install python3 virtualenvwrapper git



### Montamos la aplicación en local


1) Clonamos el repositorio:
Podemos usar el proyecto de prueba que Jesús habilitó para el taller.


    $ git clone http://github.com/jespino/taller-despliegue



2) Creamos un virtualenv:


    $ mkvirtualenv taller -p /usr/bin/python3



Para activar un virtualenv:


    $ workon taller
    (taller)$



Sabemos que estamos dentro del virtualenv porque el prompt lo indica con el nombre del entorno entre paréntesis.

Para desactivar un virtualenv:


    (taller)$ deactivate



Los virtualenvs se guardan por defecto en _/home/usuario/.virtualenvs/nombre_del_virtualenv_. Si tenemos activado el venv, todo lo que instalemos con [pip](https://pip.pypa.io/en/latest/), se instalará dentro de ese directorio.

3) Instalamos los requisitos de la aplicación:
En el directorio _demo-app_ del repositorio que nos hemos clonado en el punto 1, tenemos el fichero _requirements.txt_, que incluye los requisitos del sistema. Para instalarlos:


    $ workon taller
    (taller)$ cd /path/to/repo/demo-app
    (taller)$ pip install -r requirements.txt



Esto instala Django1.6, Gunicorn y Django-celery. De todos estos hablaremos más adelante.

**Protip**: es buena costumbre tener varios ficheros de requisitos. El habitual _requirements.txt_ con los requisitos comunes a todos los entornos; junto a este, un _requirements-devel.txt_ con los requisitos de desarollo (por ejemplo, Django-debug-toolbars), y _requirements-deploy.txt_ con los requisitos exclusivos de producción (por ejemplo, Gunicorn). En este taller no llegamos hasta ese nivel de detalle y los tenemos todos en un único fichero.

4) Sincronizamos la bbdd:


    (taller)$ cd demo-app
    (taller)$ python manage.py syncdb
    # no es necesario que creemos un superusuario
    (taller)$ python manage.py runserver



Abrimos un navegador y entramos en _http://localhost:8000_. ¡Funciona!

Hasta aquí son los pasos normales para levantar una aplicación Django en local. Ahora comienza la parte del taller donde vamos integrando herramientas de despliegue.


### Gunicorn


[Gunicorn](http://gunicorn.org/) es un servidor wsgi; en Kaleidos solemos usar este, aunque hay otros como uwsgi. A modo de prueba, podemos lanzar la aplicación con el servidor de Gunicorn:


    (taller)$ gunicorn demo.wsgi --access-logfile -



Donde:




  * _--access-logfile -_ indica que queremos que la salida del log de acceso sea stderr.
  * _demo.wsgi_ es el parámetro APP_MODULE, que apunta al fichero _wsgi.py_. Es decir, si tu aplicación se llama "demo" y dentro tienes el fichero "wsgi.py", entonces sería _demo/wsgi.py_.


Si ahora entramos en _http://localhost:8000/admin_ vemos que no se están sirviendo los estáticos (CSS, JS, imágenes) porque Gunicorn por defecto sólo sirve la app Django. Más adelante veremos cómo usar nginx para servir los ficheros estáticos.

**OJO**: En el momento de la redacción de este tutorial, la _currennt version_ de Gunicorn tiene un bug abierto; la versión 18 funciona correctamente y por eso está esa versión en el _requirements.txt_; si estás siguiendo el tutorial de una forma más libre, tenlo en cuanta a la hora de instalar Gunicorn.



### Servidor en producción


Ahora que tenemos nuestro proyecto listo para el despliegue, pasamos a la máquina remota donde va a estar el entorno final. En Kaleidos llevamos esta forma de desplegar hasta los entornos de Pre-producción normalmente. A partir de aquí, se encargan empresas especializadas.

Para estos despliegues, solemos usar [Ubuntu Server](http://www.ubuntu.com/download/server) debido a que otras distribuciones tradicionalmente usadas para entornos de producción son más conservadoras a la hora de actualizar versiones, y muchas veces no tienen las versiones más modernas que necesitamos o preferimos.

En este momento, el tutorial cuenta con que tenéis algún servidor "en la nube", que representa el entorno final. Así que tenemos que acceder a dicha máquina y repetir los primeros pasos:




  * Instalar python3, virtualenvwrapper, git
  * Clonar el repositorio
  * Crear un virtualenv
  * Instalar los requisitos
  * Sincronizar la base de datos


**Nota**: La base de datos que estamos usando en el taller es [Sqlite](http://www.sqlite.org/): es una bbdd muy ligera, poco potente y no recomendable para entornos de producción. En este punto como muy tarde, deberíais pasar a configurar la aplicación con una base de datos más robusta, como por ejemplo, [PostgreSQL](http://www.postgresql.org/).

En un servidor de producción también interesa crear un usuario _exprofesso_ para la aplicación:


    $ sudo adduser



Y creamos un usuario básico _demo_ sin permisos de _sudo_.


### Supervisor


[Supervisor](http://supervisord.org/) es un sistema de arranque y monitorización de procesos. En este caso, vamos a monitorizar el (único por ahora) proceso de gunicorn. Como lanzarlo a mano o reiniciarlo es pesado, hacemos la tarea más sencilla con supervisor.

Lo primero que tenemos que hacer es instalar supervisor:


    $ sudo apt-get install supervisor



El fichero de configuración principal está en _/etc/supervisor/supervisor.conf_. En este fichero se indica que además de esta configuración, incluye los ficheros de configuración que se definan dentro de la carpeta _conf.d/*.conf_:


    [include]
    files = /etc/supervisor/conf.d/*.conf



Así que creamos un fichero de configuración propio de nuestra aplicación en ese directorio; lo llamamos _demo.conf_ y ponemos la siguiente configuración básica:


    [program:demo]
    process_name = demo
    command = /home/demo/.virtualenvs/taller/bin/gunicorn demo.wsgi
    autostart = true
    autorestart = true
    directory = /home/demo/taller-despliegue/demo-app
    user = demo



Donde:




  * _command = /home/demo/.virtualenvs..._ indica el path del gunicorn dentro del virtualenv
  * _directory = /home/demo/taller-despliegue..._ indica el directorio donde tenemos la aplicación


Para usar Supervisor por línea de comandos:


    $ sudo supervisorctl



Esto abre un terminal de Supervisor donde lo primero que vemos es el _status_ de los ficheros configurados. Si escribimos _help_ nos salen las distinta opciones, de las cuales, destacamos:




  * _status_: muestra el status actual.
  * _reread_: relee la configuración y avisa si hay algo nuevo.
  * _update_: re-carga los procesos nuevos de los que avisa _reread_.
  * _reload_: **ACHTUNG!** Este inocente comando reinicia todos los proyectos. Normalmente no quieres usarlo.
  * _stop_: detiene un servicio que tengamos configurado
  * _start_: arranca un servicio que tengamos configurado
  * _restart_: ejecuta un stop y un start.


**Ojo**: _start_ no recarga la configuración, sino que arranca la configuración que tuviera en memoria; si cambiamos la configuración, debemos hacer _reread_ y _update_.


### Nginx


Y finalmente, toca servir nuestra flamante aplicación con [Nginx](http://nginx.org/) que es un _proxy de alto rendimiento_ (gracias, Wikipedia). Como en pasos anteriores, lo primero es instalarlo:


    $ sudo apt-get install nginx



Si conocéis Apache, los ficheros de configuración os sonarán; en cualquier caso, vamos a añadir un fichero para servir nuestro Django. Estos ficheros de configuración se ubican en:




  * /etc/nginx/sites-available, las configuraciones disponibles
  * /etc/nginx/sites-enabled, las configuraciones cargadas. Normalmente aquí solo hay enlaces simbólicos al directorio sites-available


Lo primero es que vamos a borrar el enlace simbólico del fichero de configuración por defecto:


    $ sudo rm /etc/nginx/sites-enabled/default



Ahora creamos nuestro fichero de configuración en _sites-available_ y lo enlazamos desde _sites-enabled_:


    $ sudo su
    $ touch /etc/nginx/sites-available/demo
    $ cd /etc/nginx/sites-enabled
    $ ln -s ../sites-available/demo demo



Editamos este _demo_ y ponemos la siguiente configuración básica:


    server {
        listen 80 default_server;

        location / {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }



Donde:




  * _proxy_pass http://127.0.0.1:8000;_ indica que las peticiones que entran por _/_ van al proceso que esté levantado en _127.0.0.1:8000_ que en este caso sabemos que es gunicorn
  * Las otras dos líneas setean ciertas cabeceras para que lleguen hasta la aplicación Django


Ahora ya podemos ver cómo lo sirve nginx. Recargamos nginx...:


    $ sudo /etc/init.d/nginx reload



Y si entramos en la IP de la máquina desde nuestro navegador ya vemos la aplicación funcionando. Sin embargo, aún no tenemos estáticos...

**Protip**: por simplicidad, en este tutorial solo configuramos un virtualhost, pero lo ideal sería tener uno por cada proyecto que tengamos. En cada virtualhost definiríamos el _server_name_, que indica el dominio al que responde cada apliación.


### Ficheros estáticos


En una aplicación Django hay dos _settings_ básicos relativos a los estáticos:




  * STATIC_ROOT. Es el directorio donde el comando _collectstatic_ deja los estáticos
  * STATIC_URL. Es la URL desde donde se sirven los estáticos


Como ya hemos dicho antes, lo ideal es que sea Nginx quien sirve los estáticos; para ello, editamos nuestro fichero _/etc/nginx/conf.d/demo.conf_ y lo ponemos así:


    server {
        listen 80 default_server;

        location / {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /static {
            alias /home/demo/taller-despliegue/demo-app/static;
        }
    }



Esta misma configuración habría que repetirla para los ficheros _media_, que en Django se refiere a los que suben los usuarios a través de la aplicación.


### Socket unix


Hasta ahora hemos configurado nuestro nginx para que se conecte con un socket TCP hacia gunicorn; ahora vamos a usar un socket unix porque rinde mejor y es más seguro. Esto se configura en dos pasos:

1) Arranque de gunicorn. Tenemos que añadir el binding al socket:



    command = /home/demo/.virtualenvs/taller/bin/gunicorn demo.wsgi <strong>-b unix:/home/demo/demo.sock</strong>




2) Configuración de Ngix. Aquí tenemos dos opciones, os muestro cómo quedarían ambas opciones en nuestro fichero _demo.conf_:


    # OPCIÓN A
    server {
        listen 80 default_server;

        location / {
            proxy_pass http://unix:/home/demo/demo.sock:/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /static {
            alias /home/demo/taller-despliegue/demo-app/static;
        }
    }





    #OPCIÓN B
    upstream demo_upstream {
        server unix:/home/demo/demo.sock fail_timeout=0;
    }
    server {
        listen 80 default_server;

        location / {
            proxy_pass http://demo_upstream/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /static {
            alias /home/demo/taller-despliegue/demo-app/static;
        }
    }





### Postfix


También solemos montar un servidor de correos. Para un uso ligero, en el que la aplicación no envía correos a usuarios, pero sí nos resulta interesante recibir correos de error o algún formulario de contacto, podemos usar [Postfix](http://www.postfix.org/). Lo primero es instalar Postfix:


    $ sudo apt-get install postfix



Podemos aceptar la configuración por defecto. Es interesante saber que Django usa por defecto este servidor, así que si seteáis la tupla [ADMIN](https://docs.djangoproject.com/en/dev/ref/settings/#admins), Django se encarga de enviar los correos a través de Postfix.


### Y fin...


Usamos otras herramientas junto con Django, como [Celery](http://www.celeryproject.org/) para las tareas asíncronas, o [RabbitMq](http://www.rabbitmq.com/) para las colas de mensajes. Además, para automatizar los despliegues usamos [Fabric](http://www.fabfile.org/). Queda fuera del alcance de este tutorial entrar en detalle en esto, pero os invito a que busquéis otros tutoriales al respecto.

Me quedo con ganas de entrar más a fondo en Fabric. Hice un [post de introducción a Fabric](http://moduslaborandi.net/introduccion-a-fabric/), pero me doy cuenta de que no resulta del todo útil. En breve publicaré otro post ahondando en esta herramienta y basándome precisamente en este tutorial.

La idea de este tutorial es que podáis seguirlo aunque no tengáis mucha idea (como es mi caso ^_^); pero he puesto unos cuantos enlaces para que podáis seguir indagando y profundizando. Prácticamente todos los contenidos son debidos a Jesús, a quien agradezco enormemente que se preparara un taller tan práctico y útil. Los fallos han de ser necesariamente míos :)

Si has llegado hasta aquí, ¡enhorabuena! Me hará mucha ilusión cualquier comentario que quieras poner; y si se te ocurre cómo mejorar este tutorial, estaré muy agradecida.
