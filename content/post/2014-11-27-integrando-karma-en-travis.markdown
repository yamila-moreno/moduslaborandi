---
author: yamila
comments: true
date: 2014-11-27 22:26:09+00:00
layout: post
slug: integrando-karma-en-travis
title: Integrando Karma en Travis
wordpress_id: 633
categories:
- General
---

Tras el post de [Alex](http://twitter.com/lekum) sobre [Travis 101](http://moduslaborandi.net/continuous-integration-101-travis/), tenía ganas de dedicarle un rato al tutorial.

Paralelamente, en [Kaleidos](http://kaleidos.net) estoy teniendo formación específica de Javascript (quién me mandaría aputarme), y claro, le estoy dando duro a los tests.
<!-- more -->

Así que me pareció una excusa genial para probar. Me creé un [repositorio](https://github.com/yamila-moreno/js-lessons) con los ejercicios que vamos haciendo y el objetivo es que con cada nuevo _push_ en máster se pasen los tests.

Como dice el tutorial, hay que añadirse una entrada en Travis y configurarse la integración. Después, hay que añadir en el directorio raíz un fichero _.travis.yml_ donde se configura la ejecución de los tests en Travis.

Antes de nada, varias cuestiones que he aprendido fallando:
- en la VM de Travis no tienen Chrome instalado, sólo se puede usar Firefox
- hay que configurar como lenguaje _node_js_ siguiendo la [documentación oficial](http://karma-runner.github.io/0.12/plus/travis.html) de Karma sobre esto

La configuración finalmente queda:




    language: node_js
    node_js:
        - 0.10
    before_install:
        - "export DISPLAY=:99.0"
        - "sh -e /etc/init.d/xvfb start"
    install:
        - "npm install"
    script:
        - "./node_modules/karma/bin/karma start --browsers Firefox --single-run"




Donde:
- las operaciones indicadas en **before_install** lanzan un servidor de X virtual, donde se lanzará en este caso el browser
- **npm install** instala las dependencias que están indicadas en _package.json_. En este caso, me he asegurado de que una de las dependencias sea _karma-firefox-launcher_ porque es el navegador con el que lanzaremos los tests en Travis
- en la línea del **script** indico con qué navegador quiero lanzarlo. Esto es así porque en el fichero _karma.conf.js_ he configurado dos navegadores (Chrome y Firefox) pero en Travis sólo quiero pasar Firefox. Aparte, _--single-run_ me retorna 0 si la prueba pasa o 1 si falla.

Con esto ya podéis ver los [builds](https://travis-ci.org/yamila-moreno/js-lessons/builds), ¡ahora mismo está en verde! \o/



