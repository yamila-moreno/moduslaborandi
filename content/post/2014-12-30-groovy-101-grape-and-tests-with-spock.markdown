---
author: yamila
comments: true
date: 2014-12-30 15:54:43+00:00
layout: post
slug: groovy-101-grape-and-tests-with-spock
title: 'Groovy 101 (II): Grape and tests with Spock'
wordpress_id: 738
categories:
- Tutorials
tags:
- grape
- groovy
- spock
- travis
---

Second round! We have learnt how to create a [simple program](http://moduslaborandi.net/groovy-101-groovycalc-with-tests-and-travis-ci-integration/) with Groovy, make some tests with jUnit, and integrate them with Travis.

This is a short post to next iteration to add two elements: [Grape](http://groovy.codehaus.org/Grape) (package manager) and [Spock](http://spock-framework.readthedocs.org/en/latest/) (framework for testing).
<!-- more -->


## Grape


Grape allows us to manage packages inside our Groovy scripts. It's already installed with Groovy. It's very useful when we have a simple script or application and we have a dependency. For instance, we have our tests file and we need the Spock library; then, we should place this in the top of our tests file:


    @Grab(group='org.spockframework', module='spock-core', version='0.7-groovy-2.0')



Now, we can import this library and it's ready to be used in our script:


    import spock.lang.*



When we run the tests (`$ groovy FizzBuzzSpeck.groovy`), the script will look for this dependency in our system; if it's not installed, then it will download it; maybe the first time takes a bit more time because it's installing the dependencies. Those libraries are installed in `~/.groovy/grapes/`

**Excercise**: search for a library for managing HTTP requests; then, make a script which uses this library to show some data from this amazing resource: [Star Wars API](http://swapi.co/).

**Edit**: https://github.com/yamila-moreno/groovySWAPI


## Spock


Spock is (as far as I can see) one of the most important testing frameworks for the Groovy ecosystem. So our next task consists on keep learning the Groovy syntax, travis integration, while we add tests made with Spock.

I have added a little [FizzBuzz kata](https://github.com/yamila-moreno/groovyFizzBuzz) with Spock tests. I've made this with TDD but I haven't refactored the [tests](https://github.com/yamila-moreno/groovyFizzBuzz/blob/master/FizzBuzzSpec.groovy), so it's useful as an example.


## Travis


I have integrated the tests of this FizzBuzz kata with Travis, and I found that travis only have Groovy with version 1.8. This gave a problem of versions with Spock (Spock 0.7 requires Groovy 2.0, as you can see in the `@Grab` line above). In fact, my [previous groovy repository](https://github.com/yamila-moreno/groovyCalc) had the same problem, but I didn't realized because it worked anyway.

To solve it, I have to tell the Travis script to install manually the specific version of Groovy:


    language: groovy
    groovy:
        - 2.3.9
    jdk:
        - oraclejdk8

    before_install:
        - wget http://dl.bintray.com/groovy/maven/groovy-binary-2.3.9.zip
        - unzip groovy-binary-2.3.9.zip
        - export PATH=$PWD/groovy-2.3.9/bin:$PATH

    script:
        - groovy -version #just to check the version
        - groovy FizzBuzzSpec.groovy



You can check the [travis status](https://travis-ci.org/yamila-moreno/groovyFizzBuzz/builds/45460365).

And that's all for now! You should make some excercises to ensure your knowledge of Groovy, Travis, Grab, Spock... See you soon!

As always, comments or feedback (even golden palaces) are welcome. Specially the golden palaces. You can skip the comments if you have a palace for me... Ey! It's Christmas!
