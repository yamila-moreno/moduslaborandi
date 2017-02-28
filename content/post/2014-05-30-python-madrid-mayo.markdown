---
author: yamila
comments: true
date: 2014-05-30 09:35:46+00:00
slug: python-madrid-mayo
title: Python Madrid Mayo
wordpress_id: 392
categories:
- General
---

Ayer tuvimos la sesión de [Python Madrid](http://twitter.com/python_madrid) en las oficinas de [Kaleidos](http://kaleidos.net). Últimamente estamos probando el formato de dos charlas breves, y parece que está funcionando muy bien. Las charlas de ayer fueron sobre usar Django al estilo Flask y una introducción a Pandas.

![python_madrid](/images/2014/05/python_madrid.jpg)

<!-- more -->



### Simplifying Django



[Andrey](http://www.niwi.be/) ([@niwibe](http://twitter.com/niwibe)) nos contó cómo usar Django en un proyecto de un fichero único. La idea es que un proyecto muy pequeño y ligero se puede desplegar con Django al estilo Flask. Partimos de un proyecto que no necesita persistencia, ni autenticación, ni... Para esto, creó un decorador que acopla las urls a las funciones, añadió la carga del _manage.py_ en el fichero y también la configuración básica en el _settings_ de Django.

La charla fue una demostración de que se puede reconfigurar Django, más que un enfoque para un proyecto. Hay proyectos pequeños y sencillos que aguantan bien con Flask, y otros que pueden crecer más y entonces, es más apropiado usar Django. Según Andrey, si el proyecto tiene más de un fichero (o se prevé que crezca), él ya usaría Django de base. ¿Y vosotros?

Aquí tenéis las [slides](http://slides.niwi.be/2014/simplifying-django/) de la charla y también [los ejemplos](https://github.com/niwibe/niwi-slides/tree/master/2014/simplifying-django/code).



### Pandas



[Kiko](http://pybonacci.wordpress.com/) ([@Pybonacci](http://twitter.com/pybonacci)) nos hizo una introducción a Pandas. Pandas es una <del>librería</del> biblioteca que funciona sobre NumPy y a la que agrega más funcionalidad y, sobre todo, facilidad para trabajar con grandes series de datos. La charla consistió en una introducción y vista general de las principales funcionalidades que aporta Pandas frente a NumPy, con muchos ejemplos. También se comentaron algunas carencias de Pandas, lo que nos dio pie a ver las diferencias con R.

Aquí tenéis las [slides](https://github.com/Pybonacci/notebooks/tree/master/pandas_pymad_201405) de la charla.

¿Qué herramientas usáis para la manipulación de grandes sets de datos?

**Para estar estar al día de Python Madrid**:
- puedes suscribirte a la [lista de correo](https://lists.es.python.org/listinfo/madrid)
- seguir su [twitter](http://twitter.com/python_madrid)
- o apuntarte al [Meetup](http://www.meetup.com/Madrid-Python-Meetup)
