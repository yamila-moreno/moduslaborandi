---
author: yamila
comments: true
date: 2014-07-18 13:40:17+00:00
slug: go-forrest-go
title: Go, Forrest, go
wordpress_id: 465
categories:
- General
---

_Go, Forrest, go_ es el original nombre que [Eduardo](https://twitter.com/eferro) puso al proyecto de [Piweek](http://piweek.com). Además le hizo este logo tan alucinante.

<figure>
  <img src="/images/2014/07/lo-go.jpg"
       alt="¿Es un castor? ¿Es un topo? ¡No! Es un gopher 8D" />
  <figcaption>¿Es un castor? ¿Es un topo? ¡No! Es un gopher 8D</figcaption>
</figure>

Así que no me quedó más remedio que apuntarme a su proyecto...
<!-- more -->
Esta Piweek era la excusa perfecta para aprender [Go](http://golang.org/) y una de sus características más interesantes: la facilidad para crear _goroutines_. En este caso, como tanto Eduardo como yo somos muy pythonistas así que no esperábamos que algo así fuera tan fácil :P

El proyecto que hemos hecho es una biblioteca que permite hacer consultas [SNMP](http://en.wikipedia.org/wiki/Simple_Network_Management_Protocol) de forma paralela y concurrente. Tiene una configuración de contención por host para evitar enviar demasiadas peticiones y saturar un destino. Hicimos una prueba síncrona con un servidor http y otra asíncrona con [AMQP](http://www.amqp.org/). Y todo el resultado está disponible en un [repositorio público](https://github.com/eferro/go-snmpqueries).

El lunes empezamos montando juntos un esqueleto; después fue tomando forma la biblioteca principal. El miércoles nos dividimos para hacer el trabajo: por un lado, el servidor http; por otro lado, la gestión de colas con amqp (un servicio que acepta consultas snmp desde un broker amqp y posteriormente publica las respuestas en el broker). Y el jueves terminamos de nuevo juntos y preparamos la demo. Hasta nos dio tiempo a hacer EL súper logo.

 Mis conclusiones de esta Piweek:

- Gracias a Go hemos podido levantar una pequeña aplicación que lanza muchísimas peticiones ligeras. Con Python nos habría resultado mucho más complicado.
- Con todo, Go no me ha gustado mucho; la sintaxis no es tan limpia y el sistema de paquetes me ha dejado estupefacta y ojiplática a partes iguales.
- Trabajar con Eduardo ha sido un lujazo. Es un compi excepcional, y una mala bestia del tema de comunicaciones.
- Y hace dibujos con los esquemas. Y tiene una gran paciencia explicando.

Me ha gustado mucho dedicar una semana a un lenguaje de programación del que no sabía nada, a un protocolo del que no sabía nada y con un compañero con quien apenas había trabajado (más allá de un genial [desk surfing](http://www.kaleidos.net/blog/54/desk-surfing-en-alea-soluciones/)). ¡Piweek en estado puro!

<figure>
  <img src="/images/2014/07/go-team.jpg"
       alt="Os presento a los ganadores de la VI Piweek" />
  <figcaption>Os presento a los ganadores de la VI Piweek</figcaption>
</figure>
