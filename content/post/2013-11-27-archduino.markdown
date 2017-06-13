---
author: yamila
comments: true
date: 2013-11-27 23:28:16+00:00
slug: archduino
title: Archduino
wordpress_id: 333
tags:
- arduino
---

[txistako de gratis]
En las últimas semanas he tenido que resolver este tema varias veces, y al no tenerlo apuntado, he tenido que redescubrir la solución una y otra vez. Ineficaz.

Estos son los pasos que he tenido que seguir para conseguir que mi usuario (no privilegiado) pudiera arrancar correctamente el IDE de arduino, y pudiera escribir en el arduino por el puerto serie.
<!-- more -->
0. Instalar algunas librerías:



    avr-binutils
    avr-gcc
    avr-libc




1) Conseguir el IDE de arduino. En el [AUR](https://aur.archlinux.org/) de Arch las versiones están algo desordenadas (hay una versión git y otra obsoleta...), así que me descargué el binario de la web oficial de Arduino.



    wget -c http://arduino.googlecode.com/files/arduino-1.0.5-linux64.tgz
    tar xvf arduino-1.0.5-linux64.tgz
    cd arduino-1.0.5




En este directorio, vemos que tenemos un ejecutable llamado **arduino**

2) Añadir al usuario "yami" a los grupos necesarios. Esta operación la realiza root:


    # gpasswd -a yami uucp
    # gpasswd -a yami tty
    # gpasswd -a yami lock




3) Configurar (como root) los permisos sobre la carpeta **/run/lock**:



    cp /usr/lib/tmpfiles.d/legacy.conf /etc/tmpfiles.d/
    vim /etc/tmpfiles.d/legacy.conf




y donde dice

    d /run/lock <strong>0755</strong> root <strong>root</strong> -

poner

    d /run/lock <strong>0775</strong> root <strong>lock</strong> -



Para más información sobre este paso, os recomiendo ver la [referencia](http://playground.arduino.cc/Linux/All) en la web oficial.

4) Una vez hecho lo anterior, reiniciamos el ordenador y volvemos al directorio donde tenemos el ejecutable de arduino y como usuario no privilegiado ejecutamos:


    yami@winterfell $ ./arduino



Con esto se nos arranca un IDE gráfico, con menús, etc, que permite interactuar con el arduino (¡tenlo conectado!). Para probar que está ok, navega por el menú hasta Tools y confirma que tienes disponible el menú **Serial port**.

¡Espero que os resulte útil!
