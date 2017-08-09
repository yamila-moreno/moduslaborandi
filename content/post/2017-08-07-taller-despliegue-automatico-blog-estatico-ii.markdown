---
draft: true
author: yamila
title: Taller - Despliegue automático de un blog estático (II)
slug: taller-despliegue-automatico-blog-estatico-ii
date: 2017-08-07 11:00:00+00:00
comments: true
tags:
- hugo
- pelican
- caddy
- github
- travis-ci
thumbnailImagePosition: top
thumbnailImage: https://farm5.staticflickr.com/4380/36308878171_7038f84a63.jpg
---

En el <a href="{{< ref "2017-08-07-taller-despliegue-automatico-blog-estatico-i.markdown" >}}" target="_new">post anterior</a> pusimos en marcha un blog de prueba. El siguiente paso consiste en preparar nuestro servidor, configurar un usuario y unas medidas de seguridad básicas.

<!--more-->

Esta parte del tutorial está explicada para <a href="https://www.scaleway.com/" target="_new">Scaleway</a>; si usáis otro hosting también sirve, solo que las configuraciones específicas serán un poco distintas y tendréis que adaptarlas. Si no tenéis hosting, os recomiendo este porque es muy barato y sirve perfectamente para las pruebas del taller.

Lo primero que vamos a necesitar es una clave; en Scaleway > Usuario > Credentials podemos dar de alta una clave (pública). Esta clave se añadirá en el `authorized_keys` siempre que se arranque una máquina.

Arrancamos un servidor nuevo. En Scaleway, recomiendo coger un Starter > VCS1 > ubuntu xenial, sin nada especial. Tras un par de minutos, tendremos el servidor levantado, y nos habrá asignado una IP.

Una vez que tenemos el servidor, vamos a forzar a que los usuarios solo se puedan conectar si tienen la clave pública dada de alta, e impedir que se conecten con usuario/contraseña. Para esto, dentro del servidor editamos el fichero `/etc/ssh/sshd_config`, buscamos la línea que dice `# PasswordAuthentication yes` y la cambiamos por `PasswordAuthentication no`.

Es buena práctica no usar directamente `root` sino un usuario con menos privilegios, así que accedemos al servidor y creamos uno:

```
(root)$ adduser yami # poned el nombre que queráis
```

Y ahora, para que podamos acceder con este usuario, tenemos que copiar la clave que teníamos en el `authorized_keys` de root al correspondiente de nuestro usuario:
```
(root)$ cat /root/.ssh/authorized_keys
(root)$ su yami
(yami)$ cd
(yami)$ mkdir .ssh
(yami)$ cd .ssh
(yami)$ vim authorized_keys # y dentro copiamos la clave que hemos sacado antes en el fichero con el mismo nombre
```
Guardamos y salimos del servidor. Si desde nuestro ordenador ahora intentamos:
```
(mihost)$ ssh yami@ip-de-la-máquina
```
podemos entrar en el servidor sin problemas. Voilá! Hemos configurado nuestro servidor. ¡Enhorabuena! Si tenéis cualquier duda o comentario, usad la sección de disqus. El siguiente paso consiste en instalar el servidor de aplicaciones, Caddy y hacer una pequeña prueba.



