---
author: yamila
comments: true
date: 2014-04-28 19:26:31+00:00
layout: post
slug: migrar-de-domain-wordpress-com-a-domain-es
title: Migrar de domain.wordpress.com a domain.es
wordpress_id: 378
categories:
- General
- Tutorials
tags:
- vim
- wordpress
---

Hoy he tenido que realizar una tarea bastante sencilla pero que me ha costado un par de intentos por no tener claro qué tenía que hacer. El caso es que tengo un blog en [http://dendarii.wordpress.com](http://dendarii.wordpress.com) (dentro del servidor de Wordpress) que quería pasar a gestionar en mi servidor, bajo [http://dendarii.es](http://dendarii.es) (en mi servidor privado).

Por si alguna vez me vuelve a tocar algo similar, o por si resulta de utilidad para alguien, aquí dejo los pasos que he seguido:

<!-- more -->

1) Lo primero es, claro, montar un wordpress en mi servidor. Para esto está fenomenal la [documentación de wordpress](http://codex.wordpress.org/Installing_WordPress#Famous_5-Minute_Install). No entro aquí en cómo desplegar un wordpress con nginx.

2) Después de tener esto montado, he ido al panel de administración de _dendarii.wordpress.com_ y en Tools > Export, he exportado el XML. Este XML contiene todo el contenido (etiquetas, posts, configuración) y urls para las imágenes. A este XML le he hecho algunas sustituciones (¡viva VIM!) como el usuario o el domino (http://dendarii.wordpress.com > http://dendarii.es)

3) En mi servidor flamante y nuevo, he instalado este plugin: [Wordpress Importer](http://wordpress.org/plugins/wordpress-importer/) (ya sabéis, copiarlo en wp-content/plugins y activarlo)

4) Para terminar la preparación del proceso, hay que crear una carpeta wp-content/uploads/aaaa/mm. Ojo, la carpeta _mm_ debe tener permisos 777 para poder realizar el proceso.

5) Una vez que todo esto está listo, procedemos a la importación. He ido al panel de _dendarii.es_ en la sección Tools > Import. Ahí nos pide que seleccionemos un fichero .xml (me lo descargué en el punto 2). Después, "upload file and import". Nos abre una pantalla con opciones (cambiar usuario, alguna url, importar medias). Una vez seleccionada la configuración de importación, se activa el proceso. Me ha dado algunos errores con imágenes, pero en el caso de este blog no es crítico perder este tipo de datos, así que no necesito ir más allá.

pd. He lanzado una consulta por Twitter sobre este tema y he recibido muchas y buenas respuestas de ayuda. Gracias a todos ^_^


