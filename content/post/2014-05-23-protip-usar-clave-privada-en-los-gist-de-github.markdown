---
author: yamila
comments: true
date: 2014-05-23 13:16:53+00:00
slug: protip-usar-clave-privada-en-los-gist-de-github
title: 'ProTip: usar clave privada en los gist de github'
wordpress_id: 387
categories:
- General
tags:
- gist
- git
---

Al crear un gist en [los gists de Github](https://gist.github.com/) te da únicamente la opción de clonarlo por https. De esta forma, siempre te va a pedir usuario/contraseña al hacer push.

Si tienes tu clave pública añadida a Github, lo único que tienes que hacer para poder pushear a través de ssh es:

1) te clonas el gist con la opción por defecto (https)

2) modificas el fichero .git/config:




    [remote "origin"]
        <strong>url = http://gist.github.com/584bf76e8adfd.git</strong>
        fetch = +refs/heads/*:refs/remotes/origin/*




para que se quede así:




    [remote "origin"]
        <strong>url = ssh://git@gist.github.com/584bf76e8adfd.git</strong>
        fetch = +refs/heads/*:refs/remotes/origin/*




Otra opción es clonarlo directamente con ssh




        git clone ssh://git@gist.github.com/584bf76e8adfd.git




