---
author: lekum
comments: true
date: 2014-11-06 08:18:28+00:00
slug: continuous-integration-101-travis
title: 'Continuous Integration 101: Travis'
wordpress_id: 590
tags:
- continuous integration
- django
- github
- python
- travis
---

Siguiendo la estela los artículos de temática relacionada con TDD y Django, quiero compartir con vosotros una configuración sencilla y rápida de un entorno de TDD que estoy usando en proyectos de **Python - Django** para poder automatizar la ejecución de las pruebas.
<!-- more -->

La idea es despreocuparnos de la ejecución manual de las pruebas y que haya un sistema automatizado ejecutándolas de manera periódica o ante ciertos eventos clave. Integrado dentro de un sistema que genere las builds de nuestro proyecto, conformaría un entorno de[ Integración Continua](http://en.wikipedia.org/wiki/Continuous_integration).

En este primer artículo utilizaremos un sistema que no requiere de recursos hardware o software adicionales y que se puede aplicar sobre todo proyecto que tenga un repositorio en Github. Estamos hablando de [Travis CI](https://travis-ci.org/).

Travis CI es muy sencillo de configurar. Para empezar, no hay más que entrar en su web con las credenciales de GitHub, darle al botón "+" (**Add New Repository**) y seleccionar los repositorios que queremos empezar a activar del listado de repositorios públicos:

![repositorios_travis](/images/2014/11/repositorios_travis.png)

Una vez activado, si hacemos click en el símbolo de llave inglesa junto al interruptor, accederemos a los **Settings** del repositorio en Travis:

![settings_travis](/images/2014/11/settings_travis.png)

En esta pestaña podemos marcar que se haga una _build_ (una ejecución de una serie de instrucciones, entre ellas los tests) cada vez que se realiza un _push_ o un _pull request_ al repositorio.

En la pestaña **Current** podremos inspeccionar la salida por consola de la última ejecución de una build (o de la ejecución actual, si hay una en curso) y en **Build History** tendremos el histórico de builds del repositorio.

El siguiente paso consiste en crear un fichero llamado .travis.yml en el directorio raíz del repositorio. La sintaxis de dicho fichero [YAML](http://en.wikipedia.org/wiki/YAML) se describe en detalle en la [documentación](http://docs.travis-ci.com/user/customizing-the-build/) de Travis, pero como hay mucha variedad de opciones, pasaré a describir el contenido de un fichero que tengo en un repositorio de un proyecto Python - Django:


    language: python
    python:
        - "3.4"
    before_install:
        - "export DISPLAY=:99.0"
        - "sh -e /etc/init.d/xvfb start"
    install:
        - "pip install -r requirements.txt"
        - "pip install selenium"
    script:
        - "python manage.py test lists"
        - "python manage.py test accounts"
        - "python manage.py test functional_tests"
        - "phantomjs superlists/static/tests/runner.js lists/static/tests/tests.html"
        - "phantomjs superlists/static/tests/runner.js accounts/static/tests/tests.html"


Podremos seleccionar el lenguaje del entorno de ejecución (**language**) y la versión, en este caso, de **python**.

En la sección **before_install**, podremos ubicar acciones que queremos que se realicen antes incluso de instalar las dependencias. en este caso, las acciones que tengo configuradas son las que necesito para poder arrancar [XVFB](http://en.wikipedia.org/wiki/Xvfb), que es un display server que nos permitirá ejecutar los tests funcionales con [Selenium](http://www.seleniumhq.org/) arrancando un navegador en un entorno que no tiene interfaz gráfica (_headless_). Podréis encontrar más información sobre este tipo de pruebas en [esta sección](http://docs.travis-ci.com/user/gui-and-headless-browsers/) de la documentación de Travis.

En el apartado de **install**, utilizaremos pip para instalar las dependencias del proyecto cada vez que se lance una build. Esto es así porque Travis destruye el entorno al finalizar cada ejecución, para garantizar que no hay contaminación entre pruebas.

Finalmente, en la sección de **script**, indicaremos los comandos que ejecutaremos para realizar las pruebas. En mi caso estoy haciendo pruebas de un proyecto Django, por lo que utilizo el comando test de _manage.py_ para ejecutar los tests unitarios de las distintas aplicaciones que componen el proyecto, y los tests funcionales del proyecto (los que utilizan Selenium para simular interacciones extremo a extremo). Las dos últimas líneas son para poder ejecutar los tests de javascript con el motor [PhantomJS](http://phantomjs.org/), que no requiere arrancar un navegador.

Una vez hayamos terminado el fichero, podremos validar su sintaxis mediante la herramienta online [Travis-WebLint](http://lint.travis-ci.org/).

¡Y ya está todo! No nos quedará más que realizar un _push_ al repositorio y esperar el inevitable e-mail de Travis indicando el resultado de los tests, que esperemos, sea positivo :-)

![passed_travis](/images/2014/11/passed_travis.png)
