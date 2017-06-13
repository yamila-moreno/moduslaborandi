---
author: lekum
comments: true
date: 2014-11-14 08:47:22+00:00
slug: continuous-integration-101-jenkins
title: 'Continuous Integration 101: Jenkins'
wordpress_id: 606
tags:
- continuous integration
- django
- github
- jenkins
- python
---

![00_Jenkins_Logo](/images/2014/11/00_Jenkins_Logo.png)


# Introducción


Si en el [anterior artículo](http://moduslaborandi.net/continuous-integration-101-travis/) vimos cómo configurar un sistema de CI para un proyecto Python-Django con Travis CI , hoy veremos cómo hacerlo con [Jenkins](http://jenkins-ci.org/). Si los conceptos os quedaron más o menos claros, no tendréis dificultad para realizar este proceso, pues guarda muchas similitudes.

<!-- more -->

Voy a referirme al mismo proyecto que en el anterior artículo y la instalación del software la ejemplificaré para el caso de realizarse sobre una plataforma **debian jessie**, aunque para otras distribuciones Linux seguramente sea análoga.

Jenkins es un sistema de CI con bastante solera, anteriormente se llamaba [Hudson](http://en.wikipedia.org/wiki/Hudson_%28software%29), y nació enfocado para Java, pero eso no quita para que realmente sea una herramienta muy versátil pues dispone de innumerables plugins para personalizar su comportamiento y ajustarlo a nuestras necesidades.


# Instalación del entorno


Lo primero será instalar el propio software de Jenkins. Para poder acceder a los repositorios con las versiones más modernas disponibles, añadiremos la clave pública a nuestro keyring:


    wget http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key
    sudo apt-key add jenkins-ci.org.key


y añadiremos la siguiente línea al fichero **/etc/apt/sources.list**:


    deb http://pkg.jenkins-ci.org/debian binary/


Actualizaremos el índice de paquetes e instalaremos Jenkins:


    sudo apt-get update && sudo apt-get install jenkins


Instalaremos a continuación otros paquetes para realizar los tests de este proyecto en particular:


    sudo apt-get install git iceweasel python3 python-virtualenv xvfb


Y por último, haremos una serie de pasos para poder tener **node.js** con **npm** para poder instalar **phantomjs** para correr nuestros tests de Javascript. Ojo, esto puede tardar bastante rato porque estamos compilando a partir de la última versión del repositorio, de manera  que podréis aprovechar para levantaros de la silla e ir al aseo:


    sudo apt-get install curl build-essential openssl libssl-dev
    git clone https://github.com/joyent/node.git
    cd node
    ./configure
    make && sudo make install
    curl -L -O https://npmjs.org/install.sh
    chmod +x install.sh
    sudo ./install.sh
    sudo npm install -g phantomjs


Si hemos llegado a este punto, habremos instalado todo el software necesario en el servidor.


# Configuración del entorno


La instalación en Debian arranca Jenkins con una interfaz web en el puerto TCP 8080 (por ejemplo, _http://mihost:8080_). El puerto se puede cambiar mediante la variable **HTTP_PORT** en el fichero **/etc/default/jenkins**.

Una vez dentro, lo primero que haremos será instalar un **Plugin** llamado **Locale Plugin** que nos permitirá poner la interfaz de Jenkins en inglés, ya que por defecto utiliza una traducción dependiendo del idioma del navegador y no tiene unos menús consistentes. Suponiendo que nuestro navegador está en español, esta sería la secuencia de acciones:

Ir a **Administrar Jenkins** -> **Plugins** -> **Todos los plugins** -> **Filter:** "Locale"

Seleccionar el **Locale Plugin** y elegir **Descargar ahora e instalar después de reiniciar**

Reiniciar Jenkins poniendo en el navegador la URL _http://mihost:8080/safeRestar_t.

Una vez que Jenkins reinicie, iremos a **Administrar Jenkins** -> **Configurar el Sistema** -> **Locale** y pondremos **Default Locale: en-US** y marcaremos la opción **Ignore browser preference and force this language to all users.**

Aplicaremos los cambios, y ya tendremos la interfaz de Jenkins en inglés:

![01_Interfaz_Jenkins](/images/2014/11/01_Interfaz_Jenkins.png)

A continuación, configuraremos el login. Para ello, iremos a **Manage Jenkins** -> **Configure global security** y elegiremos **Security Realm: Jenkins’ own user database**, nos aseguraremos de que la opción **Allow users to sign up** esté desmarcada y en **Authorization** escogeremos **Project-based Matrix Authorization Strategy**. Crearemos un usuario y le daremos permisos para todo, ya que en nuestro caso, va a ser un sistema monousuario muy sencillo para nuestro uso privado.

![02_Global_Security](/images/2014/11/02_Global_Security.png)

Llegado a este punto, es posible que Jenkins nos pida que creemos una contraseña para el usuario, de manera que a continuación nos solicite que nos loguemos con ese usuario y contraseña para continuar.

Parar proseguir con la configuración, instalaremos los siguientes plugins:


* Shining Panda Plugin (para Python y virtualenv)
* GIT plugin
* Xvfb plugin (un framebuffer para arrancar un navegador para los tests de Selenium).


Los plugins se instalan desde **Manage Jenkins** -> **Manage Plugins** y requieren un reinicio de Jenkins para poder activarlos.

En **Manage Jenkins** -> **Configure System** -> **Python** nos aseguraremos de tener una Python Installation con una ruta que apunte al ejecutable de Python 3.X (por ejemplo **/usr/bin/python3.4**). En Xvfb installations, crearemos una **Xvfb installation** que tenga **Directory in which to find Xvfb executable**: **/usr/bin**

![03_Configuration](/images/2014/11/03_Configuration.png)


# Creación y ejecución de la tarea de Jenkins


Bueno, ya va siendo hora de que empiece la acción. En el menú principal de Jenkins, iremos a **New Item** -> **Freestyle Project** y le daremos un nombre a nuestro proyecto.

En la pantalla de configuración, realizaremos las siguientes acciones:

En **Source Code Management** -> **Git** -> **Repository URL** pondremos la url del repo del proyecto (por ejemplo: _https://github.com/lekum/tddwp.git_)

En **Build Environment**, marcaremos la opción **Start Xvfb before the build, and shut it down after**.

En **Build**, crearemos un item de tipo **Virtualenv Builder** con versión de **Python 3.X** con las líneas para instalar los requisitos en el virtualenv y correr los tests de Python:


    pip install -r requirements.txt
    pip install selenium
    python manage.py test lists
    python manage.py test accounts
    python manage.py test functional_tests


En **Build**, crearemos otro item pero esta vez de tipo **Execute Shell** con los comandos para correr los tests de Javascript mediante phantomjs:


    phantomjs superlists/static/tests/runner.js lists/static/tests/tests.html
    phantomjs superlists/static/tests/runner.js accounts/static/tests/tests.html


![04_Configuration_project](/images/2014/11/04_Configuration_project.png)

Una vez hayamos guardado la configuración, si volvemos a la pantalla principal de Jenkins, podremos ver listado nuestro nuevo proyecto. Para ejecutarlo, hacemos click en su nombre y después en la acción **Build Now**. En el recuadro de **Build History** aparecerá una barra de progreso, y haciendo click en la ejecución, podremos ver cómo se está desarrollando y tener un feedback directo si nos vamos a la pantalla de **Console Output:**

![05_Console_output](/images/2014/11/05_Console_output.png)

Y ¡voila! Si todo va bien, tendremos nuestra _bola verde_ (&ast;). Jenkins utiliza unos iconos que hacen una analogía meteorológica del resultado de tus 5 últimas builds, con un espectro que va desde un sol radiante si han ido todas bien hasta una feroz tormenta si no ha conseguido pasar ninguna.

![06_Bola_verde](/images/2014/11/06_Bola_verde.png)

El siguiente paso, si queremos evitarnos tener que lanzar la build a mano, sería configurar el **Polling** al repositorio Git (en la ayuda contextual de Jenkins de la sección correspondiente viene más información), o mejor aún, un hook en el repositorio para [notificar a Jenkins de que se ha hecho un push](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin#GitPlugin-Pushnotificationfromrepository). Este ejercicio lo dejo a cargo del lector.

(&ast;) _Vale, no te asustes, tu bola será azul, pero para eso existe un plugin llamado **Green Balls** que puedes instalar si así lo prefieres :-)_
