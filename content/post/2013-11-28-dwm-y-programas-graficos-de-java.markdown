---
author: yamila
comments: true
date: 2013-11-28 10:49:41+00:00
layout: post
slug: dwm-y-programas-graficos-de-java
title: DWM y programas gráficos de Java
wordpress_id: 348
categories:
- General
post_format:
- Aside
tags:
- archlinux
- arduino
- java
---

Con el tema de probar [arduino](http://moduslaborandi.net/archduino/) me encontré con que el IDE se renderizaba fatal con DWM. Para esto, [RTFM](http://es.wikipedia.org/wiki/RTFM), en la web oficial, [https://wiki.archlinux.org/index.php/dwm](https://wiki.archlinux.org/index.php/dwm), pone la solución:

1) Descargar e instalar el paquete **wmname**

2) Como usuario normal, ejecutar

    wmname LG3D



Se puede añadir esta orden el fichero **.xinitrc**, justo antes de cargar DWM y voilá.
