---
author: yamila
comments: true
date: 2013-01-24 13:03:02+00:00
layout: post
slug: tu-tasa-de-paro
title: Tu tasa de paro
wordpress_id: 137
categories:
- General
tags:
- kaleidos.net
- piweek
- ttparo.es
---

<figure>
  <img src="/images/2013/01/ttdp.png"
       alt="ttdp.es" />
  <figcaption>ttdp.es</figcaption>
</figure>
<!-- more -->
Se trata de una web que consume y explota los microdatos de la EPA que el [INE](http://ine.es/) distribuye trimestralmente donde se pueden ver las tasas de paro de perfiles concretos. Un perfil es la combinación de género, rango de edad, nivel de estudios y provincia. A la hora de trabajar con los datos, tuvimos que responder algunas cuestiones; ¿qué datos introducimos? ¿cuáles aportan valor realmente? El ideólogo del proyecto fue [Carlos Gil Bellosta](http://www.datanalytics.com/) que puso a nuestra disposición todo su talento como estadístico para que estas decisiones incluso tuvieran sentido.

Con un sencillo código de colores, se puede detectar fácilmente cómo está un perfil respecto a la media global; también se puede ver en qué otras provincias ese mismo perfil tiene menos paro, o la evolución de un perfil a lo largo del tiempo. Y como _bonus_, puedes llevarte un [widget](http://tutasadeparo.es/widget) con la tasa de paro de un perfil concreto.

El proyecto está desarrollado con [Django](https://www.djangoproject.com/), [Postgresql](http://www.postgresql.org/) y [Backbone](http://backbonejs.org/) esencialmente. Y es software libre por lo que <del>está</del> _estará-en-cuanto-lo-ponga-en-github_ a disposición de quien quiera, bajo una licencia AGPLv3.

El trabajo fue muy duro; en dos semanas hicimos el estudio de UX, el diseño, la maquetación, ¡el código!, el despliegue, el tratamiento de los datos, etc... En ratos libres (léase, 'horas de sueño') hemos puesto a punto algunos detalles, y aún quedan ideas por desarrollar. Todo el equipo estamos muy contentos y orgullosos del resultado, ¡a ver qué tal se porta!
