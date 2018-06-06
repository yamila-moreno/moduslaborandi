---
author: yamila
title: Logging en Python (un tutorial frustrado)
slug: python-logging
date: 2018-01-01 14:00:00+00:00
comments: true
tags:
- python
---

En un proyecto, tenía la <em>sencilla</em> tarea de meter logs en unas aplicaciones; es una cuestión recurrente que hay que resolver en todos los proyectos y que normalmente lo resolvemos de primeras en cada proyecto. Pero esta vez decidí que quería entender bien cómo funciona el logging de Python. Es un tutorial frustrado porque al final del camino tengo la sensación de que sigo sin comprender bien el logging de Python. Con todo, comparto aquí los resultados.

<!--more-->

## Conocimientos requeridos

Este tutorial no pretende enseñar cuestiones de <em>logging</em> desde cero, sino explicitar los problemas y las soluciones que me he encontrado al ir a meter logs en una aplicación mediana. Para seguirlo correctamente necesitarás manejarte con soltura en Python, saber algo de Flask para entender los ejemplos bien, y haberte pegado ya con el logging de Python previamente.


## Contexto

En mi equipo estamos desarrollando una plataforma basada en microservicios, cada uno con una responsabilidad bien definida y acotada. Hay dos servicios especialmente relevantes para el proyecto y para este tutorial: un API y una plataforma de recoleción de datos, que llamamos <em>colector</em>. Ambos usan además una biblioteca de lectura de ficheros que también hemos desarrollado durante el proyecto y que llamamos <em>biblioteca de ficheros</em>. Hay bastantes más piezas en la plataforma pero por claridad nos quedaremos con estas tres piezas. Como información adicional, la plataforma está escrita en Python 3.6.

## Primeros pasos

Incluso para aplicaciones pequeñas me encontré con que la mayoría de tutoriales dan pequeños <em>snippets</em> muchas veces sin contexto. La <a href="https://docs.python.org/3/howto/logging.html#logging-basic-tutorial" target="_blank">documentación oficial</a> incluye un tutorial también que a mí no me llegó a servir de mucho. Con todo, encontré algún <a href="https://fangpenlin.com/posts/2012/08/26/good-logging-practice-in-python/" target="_blank">post</a> que sí me fue muy útil. Al final de esta primera fase, estos son los primeros pasos que para mí resultaron ser útiles.

**Crear un fichero de configuración**

Hay varias formas de crear este fichero, <em>yml</em>, <em>json</em> e <em>ini</em>, y son esencialmente equivalentes. En el proyecto usamos un `logging.ini`, pero daría igual porque llevan la misma información y la estructura es también prácticamente la misma. ¿Qué contenía este primer fichero de configuración?

```
# logging.ini
[formatters]
keys=simple-formatter

[handlers]
keys=console-handler

[loggers]
keys=root

[formatter_simple-formatter]
format=%(name)s - [%(levelname)s] - %(asctime)s - %(message)s

[handler_console-handler]
class=StreamHandler
level=INFO
formatter=simple-formatter
args=(sys.stdout,)

[logger_root]
level=INFO
handlers=console-handler
qualname=MYAPP-API
```

> IMPORTANTE: debe existir un `logger_root` o al cargar la configuración dará error; por este motivo, puse el logger principal como root. En este momento no necesitaríamos más.

Una estructura muy sencilla pero suficiente para empezar a logar mensajes. Es importate conocer el <a href="https://docs.python.org/3/_images/logging_flow.png" target="_new">flujo que sigue una instrucción de logging</a> para configurarlo correctamente en nuestra aplicación. Aunque nuestro <em>logger</em> tenga `level=DEBUG` si el <em>handler</em> tiene un nivel más restrictivo, los mensajes por debajo del nivel del <em>handler</em> no saldrán. En este punto recomiendo hacer pruebas en un script sencillo para asegurarnos de que salen los mensajes que queremos y como queremos.

**Cargar la configuración**

Una vez tenemos el fichero con una configuración básica, debemos cargar esta configuración en la aplicación. Es importante que carguemos esta configuración cuando se lanza la aplicación, de forma que esté disponible lo antes posible para quien lo necesite. En el siguiente ejemplo `myapp.py`, tenemos una aplicación <em>Flask</em> con un <em>Cli</em> para lanzar los comandos.

```
# myapp.py
import logging.config

import click
from flask import Flask

from myfiles import read


@click.group()
def cli():
  pass

@cli.command()
def serve():
  # en este momento se carga la configuración
  logging.config.fileConfig('/ruta/hasta/logging.ini')
  app = Flask(__name__)
  app.add_url_rule('/read', view_func=login, methods=['GET'])
  app.run(host='0.0.0.0', threaded=True)

if __name__ == '__main__':
    cli()
```

**Usar el logger en la aplicación**

Una vez que tenemos la configuración cargada y disponible, podemos usarla a través de nuestra aplicación de la siguente forma:

```
# myfiles.py
import logging

logger = logging.getLogger('MYAPP-API')

def read():
  content = "some content"
  logger.info("User read the content")
  return content
```

## Aplicaciones de terceros

Con lo anterior, ya podemos sacar logs de nuestra aplicación a través de consola. Antes he comentado que en la plataforma usamos una <em>biblioteca de ficheros</em> y queremos también disponer de los logs de la biblioteca. Para esto, hay que seguir una convención que he encontrado bastante extendida, y es poner un handler nulo en la biblioteca, de forma que la aplicación superior (nuestra api, por ejemplo) se encargue de logar ese mensaje. Así que en nuestra biblioteca tendríamos lo siguiente:

```
# readfiles.py
import logging

logger = logging.getLogger("READ-FILES")
logger.addHandler(logging.NullHandler())

def read_files():
  logger.info("Reading files")
  return "content of a file"
```

Esto hará que se cree un logger llamado `READ-FILES`; cuando se use el <em>logger</em> dentro de esta biblioteca, se encontrará que no tiene un <em>handler</em> asociado, y lo enviará hacia arriba hasta que encuentre un <em>logger parent</em>. Este ancestro será el `logger_root` de nuestra API, que resolverá emitir o no el mensaje según lo que esté configurado. Además, para poder seguir usando un <em>qualname</em> hay que añadir un <em>logger</em> específico de api en nuestra configuración principal. Con estos cambios, los ficheros quedan así:

```
# logging.ini
[formatters]
keys=simple-formatter

[handlers]
keys=console-handler

[loggers]
keys=root,api

[formatter_simple-formatter]
format=%(name)s - [%(levelname)s] - %(asctime)s - %(message)s

[handler_console-handler]
class=StreamHandler
level=INFO
formatter=simple-formatter
args=(sys.stdout,)

[logger_root]
level=INFO
handlers=console-handler

[logger_api]
level=INFO
handlers=console-handler
qualname=MYAPP-API
propagate=0
```

> El comportamiento por defecto de `logger_api` sería emitir el log y después mandar el log hacia arriba para que _también_ lo gestionase el `logger_root` si le corresponde (revisad el flujo de logging para más detalle); para evitar este comportamiento, usamos el `propagate=0`, de tal forma que `logger_api` emite el log y después impide que se propague hacia arriba.

```
# myfiles.py
import logging
from readfiles import read_files

logger = logging.getLogger('MYAPP-API')

def read():
  content = read_files()
  logger.info("User read the content")
  return content
```

Nuestra API mostrará los logs usando el <em>logger</em> `logger_api` mientras que nuestra biblioteca usará el `logger_root`. **Atención** el nivel por defecto del `logger_root` es WARNING, así que si quieres otro nivel tendrás que especificarlo.


## Distintos entornos de logging y dejar los logs en ficheros

La siguiente cuestión que nos planteamos fue que en modo desarrollo estaba bien sacar los logs por consola, pero queríamos una configuración de producción para dejar los logs en ficheros. Encontré varias aproximaciones a esto, y decidimos probar una de ellas, que está funcionando bastante bien. Consiste en crear dos ficheros de configuración, uno `dev.ini` para el modo desarrollo y otro ` pro.ini` para el modo producción.

**Crear ficheros de configuración**

Igual que en los primeros pasos, tenemos que crear los respectivos ficheros de configuración, que serían los siguientes:

```
# dev.ini
[formatters]
keys=simple-formatter

[handlers]
keys=console-handler

[loggers]
keys=root,api

[formatter_simple-formatter]
format=%(name)s - [%(levelname)s] - %(asctime)s - %(message)s

[handler_console-handler]
class=StreamHandler
level=INFO
formatter=simple-formatter
args=(sys.stdout,)

[logger_root]
level=INFO
handlers=console-handler

[logger_api]
level=INFO
handlers=console-handler
qualname=MYAPP-API
propagate=0
```

```
# pro.ini
[formatters]
keys=simple-formatter

[handlers]
keys=console-handler,file-handler

[loggers]
keys=root,api

[formatter_simple-formatter]
format=%(name)s - [%(levelname)s] - %(asctime)s - %(message)s

[handler_console-handler]
class=StreamHandler
level=INFO
formatter=simple-formatter
args=(sys.stdout,)

[handler_file-handler]
class=handlers.TimedRotatingFileHandler
level=INFO
formatter=simple-formatter
args=('/var/log/myapp/api.log', 'midnight', )
maxBytes=10485760

[logger_root]
level=INFO
handlers=console-handler,file-handler

[logger_api]
level=INFO
handlers=console-handler,file-handler
qualname=MYAPP-API
propagate=0
```

> En la configuración del <em>handler</em> que envía el log a fichero, es interesante conocer cuáles hay. El que estamos usando, `TimeRotatingFileHandler`, es el que además de persistir en fichero, hace la rotación automática.

**Cargar la configuración**

Al cargar la configuración, leemos una variable de entorno que indica si estamos en desarrollo o en producción:

```
# myapp.py
import logging.config
from os import environ

import click
from flask import Flask

from myfiles import read


@click.group()
def cli():
  pass

@cli.command()
def serve():
  myapp_environ = environ.get('MYAPP_ENV', 'dev').lower()
  myapp_environ = myapp_environ in ['dev', 'pro'] and myapp_environ or 'dev'
  fileconfig = '/ruta/hasta/{}.ini'.format(myapp_environ)
  logging.config.fileConfig(fileconfig)

  app = Flask(__name__)
  app.add_url_rule('/read', view_func=login, methods=['GET'])
  app.run(host='0.0.0.0', threaded=True)

if __name__ == '__main__':
    cli()
```

De esta forma leerá una configuración u otra según el entorno que hayamos configurado por fuera. Esta es la configuración y el uso de la biblioteca de logging de Python en nuestra plataforma. Estas notas me resultarán útiles en el futuro y sería fantástico que también os resultaras útiles a más personas. A pesar de quedarme con la sensación de no haberlo entendido todo lo bien que me gustaría, creo que he podido extraer algunas nociones importantes. ¿Qué os parecen estas decisiones? ¿Cuáles usáis en vuestros respectivos proyectos?
