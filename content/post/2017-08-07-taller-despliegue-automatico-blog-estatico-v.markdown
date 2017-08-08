---
author: yamila
title: Taller - Despliegue automático de un blog estático (V y fin)
slug: taller-despliegue-automatico-blog-estatico-v
date: 2017-08-07 14:00:00+00:00
comments: true
tags:
- hugo
- pelican
- caddy
- github
- travis-ci
thumbnailImagePosition: left
thumbnailImage: https://farm5.staticflickr.com/4402/35611037194_f5271e0241.jpg
---

Quinto y último post de la serie "*quiero publicar mi blog estático automáticamente*". Este último post contiene bastantes pasos, pero es igualmente sencillo de seguir si has hecho los cuatro anteriores. Este post consiste en crear nuestro repositorio en <a href="https://github.com" target="_new">Github</a>, darlo de alta en <a href="https://travis-ci.org" target="_new">Travis CI</a> y configurar lo necesario para que travis se encargue del despliegue.

<!--more-->

Si has llegado hasta aquí, voy a suponer que <a href="{{< ref "" >}}" target="_new">has practicado con algún generador estático</a>, <a href="{{< ref "" >}}" target="_new">has creado y configurado tu servidor</a>, <a href="{{< ref "" >}}" target="_new">has instalado caddy</a>, <a href="{{< ref "" >}}" target="_new">has configurado caddy como servicio y ya sabes cómo desplegar manualmente tu blog</a>. Todo lo anterior es necesario para terminar la serie.

El objetivo es tener el blog en un repositorio de github y que, cuando escribamos un nuevo post en markdown y lo subamos al repositorio, se despliegue automáticamente. Es importante que el repositorio sea de Github porque travis-ci solo funciona con estos repositorios.

## Preparamos el repositorio

Accedemos a <a href="https://travis-ci.org" target="_new">Travis CI</a> con las credenciales de Github. A través de la interfaz, habilitamos nuestro proyecto dentro de travis.


## Preparamos las claves

Desde nuestra máquina entramos en el servidor a través de una clave privada (en local) emparejada con una clave pública (en el servidor). Queremos que travis-ci haga lo mismo. Así que lo primero es crear un nuevo par de clave pública/privada. Cuando nos pida un nombre, escribimos `/home/<user>/.ssh/id_rsa_travis`, y aceptamos todo lo demás (sin passphrase):
```
(mihost)$ ssh-keygen -t rsa -C "agent@travis-ci.org" -b 4096
```

Al terminar, tendremos enl a carpeta `/home/<user>/.ssh` las nuevas claves:
```
(mihost)$ ls /home/yami/.ssh
id_rsa_travis
id_rsa_travis.pub
```

Ya podemos *entregar las claves*. Copiamos la **clave pública** dentro del servidor, en el fichero `/home/<user>/.ssh/authorized_keys` debajo de la que ya existe. Y ahora deberíamos meter la clave privada en el repositorio para que travis-ci pudiera usarla, **pero esto es muy mala idea**, porque estaríamos dando acceso a nuestro servidor a cualquiera. Aquí travis ofrece la posibilidad de subir la clave privada cifrada por travis, de forma que solo travis pueda descifrarla y usarla.

## Vamos a cifrar claves

Copiamos la clave privada en la raíz del repositorio (**IMPORTANTE** no hagáis commit/push del repo mientras esté la clave privada sin cifrar); yo le cambié el nombre al fichero por "travis_access":
```
(mihost)$ mv id_rsa_travis travis_access
```

Nos instalamos la gema (de ruby) llamada `travis`:
```
(mihost)$ gem install travis
```

Nos colocamos dentro del repositorio y lanzamos los siguientes comandos:
```
$ (mihost)$ touch .travis.yml
$ (mihost)$ travis login          # te pedirá las credenciales de github
$ (mihost)$ travis encrypt-file travis_access --add
```

Comprobamos varias cosas:

- ha aparecido un nuevo fichero travis_access.enc
- en el fichero .travis.yml ha aparecido una sección "before_install", que es donde indica que tiene que descifrar la clave.

Genial! **IMPORTANTE** borramos el fichero `travis_access` del repo y dejamos solo el fichero `travis_acces.enc`. Ahora ya podemos añadir nuestra clave privada al repositorio; en tiempo de ejecución, travis la descifrará y la usará para desplegar en nuestro servidor.

Toca el último paso; configurar .travis.yml para que ejecute las operaciones para desplegar, ¡vamos! Según tengamos Hugo o Pelican elegiremos una configuración u otra en el fichero `.travis.yml`.

## Configuramos el .travis.yml para pelican

```
# Aquí lo vais a cambiar por el lenguaje que uséis
language: python
python:
- '3.6'

# Sólo vamos a manejar una rama máster, pero es buena práctica indicar sobre qué rama quieres operar
branches:
  only:
  - master

# Estas líneas sirven para instalar openssl y rsync antes de que empiece a ejecutar el script, harán falta
addons:
  apt:
    packages:
      - rsync
      - openssh-client

# Estas son las líneas que añadimos antes en el comando "--add", y que no tocaremos
before_install:
- openssl aes-256-cbc -K $encrypted_a994296750d9_key -iv $encrypted_a0042967d0d9_iv -in travis_access.enc -out travis_access -d

# en esta sección instalaremos los requisitos si los tenemos y sobre todo, las 4 últimas líneas son las importantes para gestionar la clave privada
install:
  - pip install pelican Markdown
  - eval $(ssh-agent -s)
  - chmod 600 travis_access && ssh-add travis_access
  - mkdir -p ~/.ssh
  - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

# Generamos nuestro contenido estático
script: pelican content

# Y realizamos el rsync igual que antes
after_success:
  - rsync -avz --delete output/ usuario@misuperweb.com:blog/ # cambiad aquí por vuestros datos de acceso
```

## Configuramos el .travis.yml para hugo

```
# Aquí lo vais a cambiar por el lenguaje que uséis
language: go
go:
- 1.8

# Sólo vamos a manejar una rama máster, pero es buena práctica indicar sobre qué rama quieres operar
branches:
  only:
  - master

# Estas líneas sirven para instalar openssl y rsync antes de que empiece a ejecutar el script, harán falta
addons:
  apt:
    packages:
      - rsync
      - openssh-client

# Estas son las líneas que añadimos antes en el comando "--add", y que no tocaremos
before_install:
- openssl aes-256-cbc -K $encrypted_a994296750d9_key -iv $encrypted_a0042967d0d9_iv -in travis_access.enc -out travis_access -d

# en esta sección instalaremos los requisitos si los tenemos y sobre todo, las 4 últimas líneas son las importantes para gestionar la clave privada
install:
  - GOPATH=$HOME/go go get github.com/spf13/hugo
  - eval $(ssh-agent -s)
  - chmod 600 travis_access && ssh-add travis_access
  - mkdir -p ~/.ssh
  - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

# Generamos nuestro contenido estático
script:
  - "$HOME/go/bin/hugo"

# Y realizamos el rsync igual que antes
after_success:
  - rsync -avz --delete public/ usuario@misuperweb.com:blog/
```
Ahora realizamos un commit (ojo que hay que añadir ficheros), y pusheamos. Vamos a travis-ci.org y vemos el output. Tras la salida exitosa, ¡voilá! tendremos ya todo listo. A partir de ahora solo tendremos que añadir posts en markdown y el sistema se encargará automáticamente de generar la salida y publicarlo. Puedes comprobarlo añadiendo un nuevo post en el repositorio (o cambiando uno existente), pusheando los cambios y esperando a ver que se despliega solo.

¡Hasta aquí! Espero que el tutorial os haya resultado útil. Si teńeis dudas o alguna sugerencia para mejorar esta serie de posts, usad los comentarios. Vuestro feedback me será muy útil.

¡Hasta la próxima!
