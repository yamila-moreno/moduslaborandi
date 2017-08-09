---
draft: true
author: yamila
title: Taller - Despliegue automático de un blog estático (IV)
slug: taller-despliegue-automatico-blog-estatico-iv
date: 2017-08-07 13:00:00+00:00
comments: true
tags:
- hugo
- pelican
- caddy
- github
- travis-ci
---

Cuarto post de este tutorial/taller sobre despliegue automático de un blog estático. Después de <a href="{{< ref "2017-08-07-taller-despliegue-automatico-blog-estatico-iii.markdown" >}}" target="_new">instalar caddy</a>, el siguiente paso es configurarlo para que funcione como servicio.

Aprovecharemos también para preparar nuestro blog y desplegarlo manualmente.

<!--more-->

Queremos conseguir montar un servicio que podamos gestionar con `systemctl`. Este post es una receta para crear un servicio sobre un ejecutable que tengamos en el sistema. Está aplicado específicamente a caddy, pero sería muy sencillo aplicarlo a otro programa. Al instalar caddy, incluye un directorio con ejemplos para crear un servicio; usaremos ese ejemplo para nuestro servicio.

Entramos en el servidor como `root` y creamos un fichero:
```
(root)$ cp /ruta/a/caddy/init/linux-systemd/caddy.service /lib/systemd/system/caddy.service
```

Revisamos el fichero porque hay que cambiar un par de cosas:

- `User=yami`: aquí tenéis que poner vuestro usuario. Podéis borrar la línea de `Group`.
- `ExectStart=...`: aquí tenéis que poner correctamente las rutas al ejecutable de caddy y al Caddyfile. Además, si estás siguiendo la **opción b** (proviene del post anterior), tenéis que añadir al final de la línea `-port 80`.

Tras los cambios, toca activar y arrancar el servicio:
```
(root)$ systemctl daemon-reload
(root)$ systemctl enable caddy
(root)$ systemctl start caddy
```
De nuevo, volvemos al navegador y vamos a nuestro dominio (si lo tuviéramos, **opción a**) o a `http://misuperweb.com` (si seguimos la **opción b**). ¡Y funciona! Ahora, aunque la máquina se apague y se vuelva a arrancar, el servicio de caddy estará disponible y devolverá nuestro blog.

Ahora pongamos nuestro blog en producción manualmente. Vovemos a una consola en nuestra máquina (salimos del servidor), y toca preparar el blog que queremos montar. En el <a href="{{< ref "2017-08-07-taller-despliegue-automatico-blog-estatico-i.markdown" >}}" target="_new">primer post de la serie</a>, investigamos algunos generadores estáticos. El ejemplo lo voy a seguir con Hugo, que me gusta mucho, pero es esencialmente igual con otro generador.

Creamos un blog nuevo:
```
(mihost)$ hugo new site moduslaborandi  # poned el nombre de vuestro blog
```

Es buen momento para crear el repositorio en github. Creadlo y asociad el blog al repositorio (yo he añadido el directorio `public` al `.gitignore`, no lo necesitamos en el repo), ahora ya podéis instalar los temas como submódulos:

```
(mihost)$ cd moduslaborandi
(mihost)$ git submodule add https://github.com/kakawait/hugo-tranquilpeak-theme.git themes/hugo-tranquilpeak-theme # añado el tema que quiero como submódulo
(mihost)$ git submodule init
(mihost)$ git submodule update
```

Normalmente el tema viene con un `config.toml` de ejemplo; lo más sencillo es que lo copiemos en la raíz de nuestro blog y adaptemos los atributos. Tras configurar un poco, añadimos varios posts y probamos en local:
```
(mihost)$ hugo server
```

Entramos en `http://localhost:1313` y vemos el resultado. Haced las pruebas que necesitéis y continuad cuando estéis conformes (típicamente el primer post de un blog es un *Hola, mundo* pero podéis innovar cuanto queráis). Entonces generamos el blog en su versión estática:
```
(mihost)$ hugo
```

Si no existía ya, ahora hay una carpeta `public` con el contenido de html, css y js del blog. El siguiente paso para desplegarlo manualmente, es copiar el contenido de esta carpeta en el servidor, por ejemplo, con `rsync`:
```
(mihost)$ rsync -avz --delete public/ yami@moduslaborandi.net:blog/
```

Donde dice `yami` poned vuestro usuario, y cambiad también el domino por el que estéis usando. Con esto, ya está el blog en producción, entramos en el dominio a través del navegador, y **¡ya tenemos un blog!**.

Ya solo queda el paso final: configurar nuestro blog para que se despliegue automáticamente. Usaremos github y travis-ci. Mientras tanto, *happy hacking!*.
