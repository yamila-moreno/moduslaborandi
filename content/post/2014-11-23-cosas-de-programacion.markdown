---
author: yamila
comments: true
date: 2014-11-23 20:32:37+00:00
layout: post
slug: cosas-de-programacion
title: Cosas de programación...
wordpress_id: 627
categories:
- General
---

Tengo un blog de viajes. Compré un tema de wordpress para hacerlo más bonito. Tiene algún bug. Así que me he creado un repo privado para poder probarlo en local y resolverlo y poder actualizar mi servidor con los fixes.
<!-- more -->
Entonces tengo que poner a correr el wordpress en local. Mejor usa Vagrant y VirtualBox. Primero actualizo mi Arch (tras un vergonzoso mes). Nuevo kernel. Reinicio. Ahora sí, instalo vagrant y virtualbox.

Ahora para usar Vagrant tienes que usar la imagen de Debian/Jessie de alguien que la ha subido a la VagrantCloud. No funciona a la primera. Uff qué pereza.

Vuelvo a mi idea de correrlo en local. Anda, si no tengo MySQL instalado. Fíjate que en los repos oficiales de Arch solo viene MariaDB. Pues me instalo MariaDB. ¿Pero ya sé yo algo de MariaDB?

Venga, lo intento con otra vez con Vagrant. Otra imagen y ahora parece que sí... (continuará)

Vale, pero ¿qué estaba haciendo yo al principio de todo esto? Pues eso, cosas de programar...

[Edito: Resulta que para poner en marcha vagrant tengo que configurar ciento y la madre, así que he vuelto a mi versión en local (me va a salir una bipolaridad). MariaDB/MySQL tiene una administración fantástica #no. El resumen es que cada vez me gusta más mi blog de viajes _tal y como está_ XDDDD]
[Edito: Pues parece que ahora voy a probar con Virtualbox a pelo... todo esto solo porque no quiero dedicar tiempo a configurar el entorno y sí ponerme cuanto antes con el tema de wordpres... Mi estrategia está a la altura de quien quiso comenzar una invasión durante el invierno ruso, en Rusia.]
[Edito: pues al final, y esto me debería dar más vergüenza reconocerlo... pues al final me he replicado todo el entorno en mi servidor, y accedo con un dominio ad-hoc. No me juzguéis, yo solo quería resolver un bug en un php... XDDDDD]
