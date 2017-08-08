---
author: yamila
title: Taller - Despliegue automático de un blog estático (I)
slug: taller-despliegue-automatico-blog-estatico-i
date: 2017-08-07 10:00:00+00:00
comments: true
tags:
- hugo
- pelican
- caddy
- github
- travis-ci
---

La semana pasada preparé un taller para unos pocos compañeros en <a href="http://kaleidos.net" target="_new">Kaleidos</a>. Consiste en un taller práctico en el que, paso a paso, terminamos desplegando automáticamente nuestro blog en nuestro servidor. El taller tuvo muy buena acogida, así que lo dejo por aquí por si resulta útil para alguien.

<!--more-->

## Requisitos
Para seguir bien este taller, cuento con que sabes qué es un servidor, sabes hacer ssh, y te manejas con la consola de linux. Igualmente, te vendrá bien saber lo básico de git. Y doy por hecho que vas a poder configurar tu dominio para que apunte a tu servidor.

## Tecnología
El blog: <a href="http://gohugo.io" target="_new">Hugo</a> y <a href="http://getpelican.com" target="_new">Pelican</a>

El servidor: <a href="http://caddyserver.com" target="_new">Caddy</a>

El servicio de despliegue continuo: <a href="http://travis-ci.org" target="_new">Travis CI</a>

## Generadores estáticos
Un generador estático es un programa que genera contenido estático (html/css/js). A partir de un contenido en markdown y un tema que nos hemos descargado previamente, genera el HTML/CSS/JS correspondiente. Estos contenidos estáticos los podemos servir de forma eficiente y sencilla desde casi cualquier parte. Es muy útil para blogs sencillos, y además es muy seguro.

En la web <a href="http://staticgen.com" target="_new">Static Gen</a> tenemos un ranking con un montón de generadores estáticos; los puedes filtrar por lenguaje de programación o verlos todos ordenados. A la hora de elegir uno u otro, importa poco el lenguaje en el que estén programados; es mucho más importante que tenga temas que te gustan y que tenga una comunidad interesante en la que aportar o a la que preguntar llegado el caso.

Para este tutorial, os sugiero que elijáis Hugo o Pelican, pues los ejemplos están ya preparados.

## Ejercicio
Descargad Hugo o Pelican (¡o los dos!), elegid algún tema y haced una prueba sencilla y que podáis tirar a la papelera. Ahora se trata solo de probar un generador.

* <a href="https://themes.gohugo.io/" target="_new">Temas de Hugo</a>
* <a href="http://pelicanthemes.com/" target="_new">Temas de Pelican</a>

Echad un vistazo a los ficheros de configuración (`pelicanconf.py` o `config.toml`) y a los atributos propios del tema.

Probad a crear varios posts, con imágenes y a ver el resultado en vuestro ordenador. Haced varias pruebas con borradores, con tags, etc. La idea es que aprovechéis para afianzar vuestro conocimiento de generadores estáticos.

Bien, pues, ¡así se hace un blog estático! Primer paso, conseguido. El siguiente paso consiste en preparar nuestro servidor para que aloje el blog.
