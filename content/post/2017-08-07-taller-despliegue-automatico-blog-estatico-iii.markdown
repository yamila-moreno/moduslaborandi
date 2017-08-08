---
author: yamila
title: Taller - Despliegue automático de un blog estático (III)
slug: taller-despliegue-automatico-blog-estatico-iii
date: 2017-08-07 12:00:00+00:00
comments: true
tags:
- hugo
- pelican
- caddy
- travis-ci
---

Tercer post del tutorial; después de <a href="{{< ref "2017-08-07-taller-despliegue-automatico-blog-estatico-i.markdown" >}}" target="_new">aprender algo sobre generadores estáticos</a> y <a href="{{< ref "2017-08-07-taller-despliegue-automatico-blog-estatico-ii.markdown" >}}" target="_new">preparar nuestro servidor</a>, en este post toca instalar y configurar nuestro servidor HTTP, <a href="https://caddyserver.com" target="_new">Caddy</a>. ¡Al lío!

<!--more-->

Caddy es un servidor web HTTP/2 con HTTPS automático. Nuestros servicios se configuran de forma compacta y sencilla en un `Caddyfile`; el binario ocupa muy poco y da buen rendimiento. Por todo esto, lo vamos a usar para **servir nuestro blog**.

Entramos en el servidor, descargamos el binario y lo descomprimimos:
```
(yami)$ cd
(yami)$ mkdir caddy
(yami)$ cd caddy
(yami)$ wget https://github.com/mholt/caddy/releases/download/v0.10.6/caddy_v0.10.6_linux_amd64.tar.gz
(yami)$ tar -xzvf caddy_v0.10.6_linux_amd64.tar.gz
```

Si estáis usando el servidor recomendado para este taller, este binario servirá; si tenéis otro servidor con otro sistema, tal vez necesitéis otro binario. Podéis ver las opciones en <a href="https://github.com/mholt/caddy/releases/" target="_new">el listado de releases de Caddy</a>.

Ahora, como usuario `root`, vamos a permitir que el binario de caddy tenga permisos para escuchar en los puertos 80 y 443 (puertos privilegiados).
```
(root)$ cd /home/yami/caddy
(root)$ setcap cap_net_bind_service=+ep ./caddy
```

Ahora viene algo **importante**. Llegados a este punto, podemos tener una de las dos siguientes situaciones:
a) tenemos un dominio configurado y apuntando a nuestro servidor
b) no tenemos un dominio configurado y apuntando a nuestro servidor

Según tengáis la opción a o b, habrá que modificar ligeramente la configuración. Concretamente, si no tenéis el dominio apuntando al servidor (opción b), vamos a tener que desactivar tls, vamos a tener que forzar el puerto y no vamos a poder tener https automático. Esta opción es más insegura, aunque si la puedo explicar es porque yo misma he pasado por ahí, y no vengo a juzgar a nadie :P En todo caso, sería mucho mejor tener un dominio correctamente configurado.

En lo que queda de post, indicaré cuándo es configuración de opción A (con dominio configurado) y cuándo es de opción B (sin dominio configurado). Si no indico nada, la instrucción vale para ambos casos.

Creamos un par de carpetas que nos serán útiles después:
```
(yami)$ cd
(yami)$ mkdir blog logs
```

Dentro de la carpeta `blog` creamos un `index.html` con un *Hola, mundo* sencillo. Es nuestra web de pruebas.
Y dentro de la carpeta `caddy` creamos un fichero llamado `Caddyfile`:

**Opción A**:
```
midominio.com, www.midominio.com {               # aquí ponéis el dominio correspondiente
  log /home/yami/logs/caddy.midominio.log
  root /home/yami/blog
}
```

**Opción B**:
```
misuperweb.com, www.misuperweb.com {
  tls off                                        # en esta línea está el meollo
  log /home/yami/logs/caddy.midominio.log
  root /home/yami/blog
}
```

Podemos validar el Caddyfile:
```
(yami)$ ./ruta/a/caddy -validate -conf=ruta/a/Caddyfile
```

Y si todo ha ido bien, probamos el servidor. **Opción A**:
```
(yami)$ ./ruta/a/caddy -conf=ruta/a/Caddyfile
```
Abrimos el navegador y entramos en el dominio que hayáis configurado.

**Opción B**:
Vamos a nuestra máquina local, y como `root` editamos el fichero `/etc/hosts` y añadimos una línea similar:
```
50.10.140.220   misuperweb.com    # poned la IP de vuestro servidor
```

Después entramos de nuevo en el servidor y lanzamos caddy:
```
(yami)$ ./ruta/a/caddy -conf=ruta/a/Caddyfile -port 80
```
Abrimos el navegador y entramos en `http://misuperweb.com`.

¡Felicidades! Ya tenemos una web fantástica :D En el próximo post, haremos que la gestión de caddy sea más sencilla y profesional (crearemos un servicio automático que levante caddy). Mientras tanto, si estáis siguiendo el tutorial y os surgen dudas, usad los comentarios \o/

