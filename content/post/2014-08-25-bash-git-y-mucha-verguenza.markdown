---
author: yamila
comments: true
date: 2014-08-25 17:18:32+00:00
slug: bash-git-y-mucha-verguenza
title: Bash, git y mucha vergüenza
wordpress_id: 550
categories:
- General
- Open source
tags:
- bash
- git
- linux
---

Esto es un post de vergüenza absoluta para mí, que nunca he usado [bash](http://www.gnu.org/software/bash/), pero todo hay que visibilizarlo... Y lo mismo le resulta útil a alguien.
<!-- more -->
La cosa viene así, tengo un script de _bash_ en un repositorio A, cuyas órdenes son: 1) ir al repositorio B y 2)realizar una operación concreta:


    cd ../../repositorio/b
    git reset --hard



Efectivamente, _git reset --hard_. Para quien no sepa nada de [git](http://git-scm.com/), esto normalmente significa **muerte y destrucción**, con variantes del tipo **con mucho dolor**, o **con napalm** o **y tu madre viene a verte sin avisar**. Es decir, ojito cuidado.

Pues resulta que dado el código anterior, si la orden "_cd..._" falla, el script sigue ejecutando las siguientes órdenes. Significa que si por algún motivo yo no programé bien ese "_cd..._", igualmente hace la segunda operación. Casualmente, el script está en un repositorio donde le ha parecido perfecto ejecutar esa orden.

Ya os imaginaréis que había puesto mal el "_cd..._". Así que en un segundo, he visto cómo en mi repositorio (el importante, no podría ser de otra forma), se iban todos mis (muchos) cambios al garete.

<figure>
  <img src="http://assets.sbnation.com/assets/557818/unleashHell_medium.jpg" width=""
       alt="Esto es git personificado gritando: "A mi señal, ¡¡ira y fuego!!"" />
  <figcaption>Esto es git personificado gritando: "A mi señal, ¡¡ira y fuego!!"</figcaption>
</figure>

¿Que por qué no los había commiteado? Pues tengo un montón de razones que lo justifican, pero lo cierto es que no... injustificable. Encima yo suelo hacer commits pequeños, pero aquí me lié con varias cosas... y nada, que no había commits.

¡Sin embargo! Este post tiene final feliz... gracias a nuestro super turbo peta tera experto en git, [Jesús Espino](http://twitter.com/jespinog), _que la barba le crezca larga_, y a que **había stasheado los cambios** en cierto momento (aunque el stash ya no existía para entonces), he podido recuperar los cambios. Y Jesús se ha quedado como...

<figure>
  <img src="http://cdn.meme.li/instances/500x/53765765.jpg" width=""
       alt="El éxito se le ha subido a la cabeza..." />
  <figcaption>El éxito se le ha subido a la cabeza...</figcaption>
</figure>

Bien, ¿y cómo tendría que haber hecho el código inicial para que funcionara correctamente?


    cd ../../repositorio/b && git reset --hard


De esta forma, si falla la primera instrucción no se ejecuta la segunda... #ay

Conclusión: a mi mantra de los [lunes](http://moduslaborandi.net/lunes), de _no tomar decisiones importantes_ añadiré otro:


> ... ni hagas scripts de bash
