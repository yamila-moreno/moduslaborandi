---
author: yamila
comments: true
date: 2014-07-29 17:44:25+00:00
slug: introduccion-a-fabric-ii
title: Introducción a Fabric (II)
wordpress_id: 498
categories:
- Tutorials
tags:
- fabric
- python
---

A raíz del taller-que-terminó-en-post sobre [cómo desplegamos aplicaciones Django en Kaleidos](http://moduslaborandi.net/ksm-despliegue-de-apps-de-python-kaleidos-style/), me acordé de la [no muy buena introducción a Fabric](http://moduslaborandi.net/introduccion-a-fabric/) que hice en su momento.

Así que aprovechando el taller y los materiales que nos dio Jesús, aquí tenéis _otra_ introducción a Fabric, esta vez con ejemplos sobre cómo usamos Fabric en [Kaleidos](http://kaleidos.net) para despliegues de apliaciones Django.
<!-- more -->

**Pequeño _disclaimer_**: Aunque el taller anterior está hecho con Python3, resulta que Fabric no es compatible con esta versión. Esto significa que para desplegar con Fabric vamos a crear un nuevo virtualenv con Python2.7, tanto en nuestra máquina local, como en el servidor remoto.



### Conocimientos previos


Es la continuación natural del tutorial que menciono; si ya sabes cómo hacer despliegues manuales, no te interesa leértelo antes, aunque sí deberías echar un vistazo a la estructura que dejamos; si nunca has hecho un despliegue con Django, te recomiendo encarecidamente que le dediques un rato al tutorial :)

Por lo demás, si no sabes nada de Fabric, no te preocpues, empezamos por el principio.



### Fabric



Es una biblioteca y una aplicación de línea de comandos para realizar despliegues por SSH (gracias, [documentación oficial](http://www.fabfile.org/)). Tiene una configuración muy sencilla y potente. Nunca he puesto a prueba qué tal escalará para grandes despliegues; lo que yo hago habitualmente con Fabric es desplegar entornos de desarrollo o preproducción.

Sirve para automatizar y sistematizar los despliegues. Esto es muy importante por varios motivos:




  * si somos varias personas en un proyecto, nos aseguramos que todas hacemos los despliegues igual
  * también nos aseguramos que los despliegues son siempre iguales y repetibles
  * nos ayuda a no dejarnos ningún paso por error u omisión



Hay que instalar la biblioteca en un _virtualenv_ (creado con Python2.7):



    $ mkvirtualenv despliegue -p /usr/bin/python2.7
    $ workon despliegue
    (despliegue)$ pip install fabric






### Elementos fundamentales



**fabfile.py**

Si invocamos Fabric desde un directorio donde exista un documento llamado **fabfile.py**, el script asumirá por defecto que tiene que usar este fichero. Se pueden tener las órdenes en otro documento e indicar qué documento es por parámetro. Por sencillez, vamos a crear un documento llamado **fabfile.py** dentro del directorio _demo-app_.




    (despliegue)$ cd demo-app
    (despliegue)$ touch fabfile.py




**Tareas**

Las tareas de Fabric son las instrucciones que vamos a dar para desplegar. Tareas pueden ser:





  * Conéctate al servidor remoto
  * Actualiza el repositorio
  * Reinicia el supervisor



Se expresan como funciones de Python, que reciben argumentos normalmente. Podemos editar nuestro documento _fabfile.py_ y poner lo siguiente:




    def hello(name='world'):
        print(u'Hello {}'.format(name))




Guardamos y desde el terminal podemos ejecutar:




    (despliegue)$ fab hello
    hello world

    Done.
    (despliegue)$ fab hello:name=yami
    hello yami

    Done.




¡Bravo! Ya tenemos nuestra primera tarea hecha. En los próximos pasos mejoraremos sustancialmente el fichero :)

**Qué más aporta la <del>librería</del> <del>biblioteca</del> bibliería**

Al instalar Fabric, tenemos acceso a una serie de métodos y decoradores muy útiles a la hora de actualizar:




    from fabric.api import local, settings, abort, run, cd, env, put, get
    from fabric.decorators import task, hosts, with_settings




No está en el alcance de este tutorial explicar cada método, sólo indico ahí unas cuantas interesantes. Y explicaremos más en detalle las que vayamos a usar para nuestro despliegue.





  * _@task_: es un decorador que indica que el método decorado es una tarea. Así, cuando ejecutemos _fab --list_ solo mostrará y solo se podrán invocar estas tareas
  * _@hosts_: es un decorador donde indicamos una lista de a qué servidor o servidores se tiene que conectar
  * _run(something)_: ejecuta la instrucción _something_ en el servidor remoto (host)
  * _cd(path)_: se mueve al directorio indicado en el servidor remoto (host). En Kaleidos lo usamos con _context managers_





### Configuración sencilla



Ya tenemos lo necesario para ir configurando nuestro despliegue (borra lo que tuvieras escrito hasta ahora en el fichero). Normalmente se despliega _después_ de algún cambio, así que deberíamos realizar algún cambio en el código para darle más emoción (recuerda que si quieres arrancar el _runserver_ para probar tus cambios tienes que hacerlo en el virtualenv "taller").

Editamos _fabfile.py_ y lo primero que hacemos es **conectarnos al servidor**:




    from fabric.decorators import task, hosts
    from fabric.api import run

    @task
    @hosts('myuser@myhostprivado.com')
    def deploy():
        run('echo "Hola!!")




Como veis, en _@hosts_ indicamos cuál es el servidor remoto:




  * _myuser_: se refiere al usuario en la máquina remota
  * _myhost_: se refiere al servidor en sí, llamado por el nombre de dominio o por la IP



Estoy dando por hecho que sabéis qué es SSH, que tenéis el servidor SSH levantado en el puerto 22, que es el puerto por defecto, y que accedéis a través de clave privada. Esto último no es imprescindible pero más cómodo.

Si ahora ejecutáis _fab deploy_, os saldrá algo como:




    [myuser@myhostprivado.com] Executing task 'goodbye'
    [myuser@myhostprivado.com] run: echo "goodbye!"
    [myuser@myhostprivado.com] out: goodbye!
    [myuser@myhostprivado.com] out:
    Done.
    Disconnecting from myhostprivado.com... done.




¡Genial! Vemos que se conecta, que realiza el _echo_ y, ojo, **cierra la conexión al terminar**.

Una vez que nos conectamos al servidor, lo suyo es **colocarse en el directorio desde donde ejecutaríamos las operaciones** a mano; para esto vamos a usar el método _cd_ en un _[context managers](http://eigenhombre.com/2013/04/20/introduction-to-context-managers/)_; nuestro _fabfile.py_ quedaría así:




    from fabric.decorators import task, hosts
    from fabric.api import cd, run

    @task
    @hosts('myuser@myhostprivado.com')
    def deploy():
        with cd('/path/to/repository'):
            run('hello, i am at: ')
            run('pwd')




Si ejecutamos esta tarea, veremos cómo en la salida de la consola nos imprime el path que le indicamos en _cd('/path/to/respository')_.

Sigamos. Después de estar en el directorio correcto, toca **actualizar el código**; de forma que nuestro documento quedará así:




    from fabric.decorators import task, hosts
    from fabric.api import cd, run

    @task
    @hosts('myuser@myhostprivado.com')
    def deploy():
        with cd('/path/to/repository'):
            run('git reset --hard')
            run('git fetch')
            run('git checkout master')
            run('git pull')




Como veis, hago un _reset --hard_ para asegurarme de que se deshacen los cambios que se hubieran hecho directamente en este servidor (¡anatema!) y se suben los que yo quiero y están en el repositorio. En este caso, estamos desplegando la rama _master_.

Para ejecutar algunas operaciones en remoto dentro del **entorno virtual** hay que hacer un truco; a mi entender es feillo, pero es que la vida a veces es así. Seguimos modificando, pues, nuestro _fabfile.py_:




    from fabric.decorators import task, hosts
    from fabric.api import cd, run

    def execute_in_remote(command, venv='despliegue'):
        return 'source /path/home/.bashrc && ' \
               'source /etc/bash_completion.d/virtualenvwrapper && ' \
               'source /etc/profile && ' \
               'workon {} && ' \
               '{}'.format(venv, command)

    @task
    @hosts('myuser@myhostprivado.com')
    def deploy():
        with cd('/path/to/repository'):
            run('git reset --hard')
            run('git fetch')
            run('git checkout master')
            run('git pull')

       with cd('/path/to/respository/demo-app'):
            execute_in_remote(run('python manage.py collectstatic'))




Muchos cambios:




  * Añadimos el método _execute_in_remote_ que se encarga de lanzar los comandos en el virtualenv. Al no tener el decorador _@task_, Fabric no lo expone como una tarea del despliegue
  * Nos movemos al directorio _demo-app_, que es donde está _manage.py_
  * Ejecutamos el _collectstatic_ dentro del entorno virtual



He visto que hay una [biblioteca](https://pypi.python.org/pypi/fabric-virtualenv) que facilita y embellece la integración entre Fabric y Virtualenv, pero no la he probado aún, aunque tiene buena pinta.

También podemos realizar otras operaciones habituales como _npm install_ si lo usáramos en nuestro proyecto. A veces tenemos operaciones muy pesadas que no queremos ejecutar siempre, sino solo cuando es estrictamente necesario; para esto, vamos a parametrizar algunas órdenes de forma que podamos indicar qué queremos ejecutar en cada caso.




    from fabric.decorators import task, hosts
    from fabric.api import cd, run

    def execute_in_remote(command, venv='despliegue'):
        return 'source /path/home/.bashrc && ' \
               'source /etc/bash_completion.d/virtualenvwrapper && ' \
               'source /etc/profile && ' \
               'workon {} && ' \
               '{}'.format(venv, command)

    @task
    @hosts('myuser@myhostprivado.com')
    def deploy(**kwargs):
        with cd('/path/to/repository'):
            run('git reset --hard')
            run('git fetch')
            run('git checkout master')
            run('git pull')

       with cd('/path/to/respository/demo-app'):
            execute_in_remote(run('python manage.py collectstatic'))

            if 'requirements' in kwargs:
                run('pip install -r relative/path/to/requirements.txt')
            if 'load_fixtures' in kwargs:
                run('python manage.py loaddata admin_user')




En el _fabfile.py_ hemos añadido el parámetro _**kwargs_. De esta forma, reinstalaremos los _requirements_ solo cuando se haga expreso y solo cargaremos la fixture cuando se pida explícitamente:




  * _fab deploy_ **no** ejecuta las instrucciones _pip install..._ ni _python manage.py..._
  * _fab deploy:requirements=1_ **sí** ejecuta la instrucción _pip install..._ pero no _python manage.py..._
  * _fab deploy:load&UnderBar;fixtures=1_ **sí** ejecuta la instrucción _python manage.py..._ pero no _pip install..._
  * _fab deploy:requirements=1,load&UnderBar;fixtures=1_ **sí** ejecuta ambas instrucciones



Ya prácticamente lo tenemos. Ahora seguramente, querramos **reiniciar _supervisor_** para que recargue los cambios que hemos añadido:




    from fabric.decorators import task, hosts
    from fabric.api import cd, run, sudo

    def execute_in_remote(command, venv='despliegue'):
        return 'source /path/home/.bashrc && ' \
               'source /etc/bash_completion.d/virtualenvwrapper && ' \
               'source /etc/profile && ' \
               'workon {} && ' \
               '{}'.format(venv, command)

    @task
    @hosts('myuser@myhostprivado.com')
    def deploy(**kwargs):
        with cd('/path/to/repository'):
            run('git reset --hard')
            run('git fetch')
            run('git checkout master')
            run('git pull')

       with cd('/path/to/respository/demo-app'):
            execute_in_remote(run('python manage.py collectstatic'))

            if 'requirements' in kwargs:
                run('pip install -r relative/path/to/requirements.txt')
            if 'load_fixtures' in kwargs:
                run('python manage.py loaddata admin_user')

        sudo('supervisorctl restart demo')




Habíamos dejado _supervisor_ configurado para que sólo pudiera lanzarlo un usuario con privilegios; así que para esto, podemos usar _sudo_ de Fabric, que lanza una operación en remoto con sudo. Sería equivalente a:



    run('sudo supervisorctl restart demo')




¡Voilá! Ya lo tenemos. Podemos actualizar periódicamente y de forma determinista nuestro proyecto. ¡Enhorabuena!

Aunque el alcance inicial también cubría la conexión por túnel o la gestión de varios hosts, considero que este tutorial ha quedado muy largo y como introducción es suficiente. A partir de aquí, puedes investigar en la documentación oficial para hacer tus scripts más complejos si lo necesitas.

Como siempre, me encantará saber vuestra opinión sobre el tutorial o cualquier sugerencia de mejora.

