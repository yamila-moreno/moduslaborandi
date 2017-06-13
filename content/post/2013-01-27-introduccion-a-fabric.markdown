---
author: yamila
comments: true
date: 2013-01-27 23:54:00+00:00
slug: introduccion-a-fabric
title: Introducción a fabric
wordpress_id: 185
tags:
- fabric
- gettext
- my modus
- python
---

[Fabric](http://docs.fabfile.org/en/1.5/) es una librería hecha en [Python](http://www.python.org/) muy útil a la hora de estandarizar despliegues. Su gran virtud es que permite que todos los implicados en un desarrollo realicen las mismas operaciones a la hora de deplesgar sin dejarse ningún paso. Para eso, lo apropiado es que el script de fabric (fabfile.py) esté en el repositorio y vaya acumulando todas las órdenes necesarias.
<!-- more -->
Fabric se instala con [pip](http://pypi.python.org/pypi/pip) y se invoca con con _fab_.

El _theme_ de este blog viene con sus correspondientes ficheros [.po](http://en.wikipedia.org/wiki/Gettext) en varios idiomas, pero no en Español. Como no <del>pude</del> quise instalarle un plugin de internacionalización, decidí ir traduciendo el tema poco a poco directamente sobre los ficheros. Esto consiste en editar el fichero es_ES.po del tema con [Poedit](http://www.poedit.net/) que genera su correspondiente es_ES.mo, y subirlo a su sitio en el servidor donde está alojado. Son dos operaciones muy sencillas y básicas que me han servido de excusa perfecta para probar fabric.

Fabric permite ordenar operaciones en la máquina local y en el servidor remoto, de modo que podamos ejecutar el despliegue sin hacer [ssh](http://en.wikipedia.org/wiki/Secure_Shell). En el caso que nos ocupa quiero copiar los ficheros (scp) al servidor y una vez en el servidor colocarlos en su directorio y recargar [apache](http://www.apache.org/). Así que _show me the code_; el fichero fabfile.py quedaría:




    #-*- coding:utf-8 -*-
    from fabric.api import *
    from fabric.decorators import task

    # Defino el servidor remoto.
    # Es una tupla así que puedo definir
    # varios si tengo que hacer el mismo
    # despliegue en varias máquinas

    env.hosts = ['yami@moduslaborandi.net']

    # Copio los dos ficheros necesarios al servidor remoto
    # Los dejo en la /home del usuario 'yami'

    def scp_to_modus():
        #local('scp bugis/es_ES.* yami@moduslaborandi.net:')
        put('bugis/es_ES.*')

    # Después los muevo a su carpeta dentro del tema de wordpress
    # Esta operación la realiza en el servidor remoto
    # que hemos definido en envs.hosts

    def mv_in_modus():
        run('mv es_ES.* \
            moduslaborandi/wp-content/themes/bugis/languages')

    # Le cambio los permisos

    def chown_perms():
        run('sudo chown yami:www-data \
            moduslaborandi/wp-content/themes/bugis/languages/es_ES.*')

    # Recargo apache

    def reload_apache():
        #run('sudo /etc/init.d/apache2 reload')
        sudo('/etc/init.d/apache2 reload')

    # Llamo a todas las órdenes anteriores de una única invocación
    # El decorador @task indica que esta es la orden que se puede
    # llamar y que se mostrará como opción con fab --list

    @task
    def upload_translation():
        scp_to_modus()
        mv_in_modus()
        chown_perms()
        reload_apache()




Dado que este fichero se llama _fabfile.py_, la invocación a fab automáticamente va a leerlo. Si se llama de otra forma, habría que pasar el nombre del script como parámetro.

Desde mi consola, pues, haría la siguiente llamada:



    fab upload_translation




Esta es una introducción con un ejemplo sumamente sencillo, aunque con Fabric se pueden realizar despliegues complejos y controlados. Os recomiendo que os paséis por el [tutorial](http://docs.fabfile.org/en/1.5/tutorial.html) oficial y por su documentación para aprender más en detalle esta herramienta.

EDITO: Gracias a los comentarios de [@ae_bm](https://twitter.com/ae_bm) en Twitter, he añadido un par de llamadas directamente contra la api de Fabric. Dejo las anteriores comentadas para que se vea la equivalencia.
