---
author: yamila
comments: true
date: 2013-01-29 22:11:03+00:00
layout: post
slug: agile-girls-y-tdd
title: Agile girls y TDD
wordpress_id: 215
categories:
- General
- Metodology
tags:
- agilegirls
- tdd
---

![@agilegirls](/images/2013/01/agile-girls.jpg)
Yo debería estar haciendo mi curso de [MongDB](https://education.10gen.com/courses/10gen/M101P/2013_Spring/about) pero lo de hoy bien merece un post. Hoy fui por primera vez a una sesión de las [Agile Girls](http://www.agile-girls.com/) ([@agilegirls](https://twitter.com/agilegirls)) en la que [@laura_morillo](https://twitter.com/laura_morillo) nos ha dado una introducción a [TDD](http://en.wikipedia.org/wiki/Test-driven_development).
<!-- more -->
De primeras el concepto de reunirnos _porque somos chicas y programamos_ me chocaba un poco, pero como la cabecera de la web es una chica con un portátil y un gato no me lo he pensado dos veces. He descubierto que nos sentíamos más libres de preguntar y equivocarnos. Además, al reunirnos por una cuestión de forma, cada una de las que hemos ido, aportamos conocimientos muy dispares; me ha encantado que cuando tocaba pensar en próximos talleres, han salido un montón de ideas y gente animada a compartir lo que sabe y a prepararse alguna charla. Así que ha sido un descubrimiento genial.

La sesión de hoy era una introducción al TDD; tras una brevísima teoría, por parejas hemos hecho una kata: [FizzBuzz](http://en.wikipedia.org/wiki/Fizz_buzz).



> Programa tests que fallan



Programar un test que pasa no tiene ningún sentido. Hay que empezar por el _rojo_.



> Programa el código mínimo para que pasen los tests



Hay que generar tests que sean ejemplos concretos y hacer que lleguen a _verde_.


> Los tests también se refactorizan



Tras el verde, no solo toca _refactor_ del código, sino también los tests. Por ejemplo, los nombres de los tests deberían tener sentido y servir de documentación.

Me he puesto con [@amaiac](https://twitter.com/amaiac) que está bastante más versada y me ha ayudado mucho a seguir algunas reglas básicas y a obligarme a hacer los tests uno a uno. Y el resultado...




    # -*- coding: utf-8 -*-

    import unittest
    from fizzbuzz import fizzbuzzgame

    class FizzBuzzTest(unittest.TestCase):

        def test_fizzbuzz_no_multiplo_3_ni_5(self):
            self.assertEqual(fizzbuzzgame(1), 1)

        def test_fizzbuzz_multiplo_3(self):
            self.assertEqual(fizzbuzzgame(12), 'fizz')

        def test_fizzbuzz_multiplo_5(self):
            self.assertEqual(fizzbuzzgame(65), 'buzz')

        def test_fizzbuzz_multiplo_3_y_5(self):
            self.assertEqual(fizzbuzzgame(15), 'fizzbuzz')

    if __name__ == '__main__':
         unittest.main()




¿Sacáis fácilmente la implementación con estos tests?

Así que <del>por culpa de</del> gracias a Laura, añado el TDD como propósito para este año; esto merece la pena hacerse bien. Finalmente, gracias a [ClickDistrict](http://clickdistrict.es/) por acoger la reunión y al grupo por la predisposición mostrada. Tengo muchas ganas de que llegue ya la próxima sesión.
