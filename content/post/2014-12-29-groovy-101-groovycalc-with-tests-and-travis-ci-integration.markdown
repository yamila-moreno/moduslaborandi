---
author: yamila
comments: true
date: 2014-12-29 15:06:44+00:00
layout: post
slug: groovy-101-groovycalc-with-tests-and-travis-ci-integration
title: 'Groovy 101 (I): groovyCalc with tests and Travis-CI integration'
wordpress_id: 722
categories:
- Tutorials
tags:
- groovy
- travis
---

![](http://upload.wikimedia.org/wikipedia/commons/thumb/3/36/Groovy-logo.svg/614px-Groovy-logo.svg.png)

The time has arrived; I cannot delay it more... I have to learn [Groovy](http://groovy-lang.org/) and [Grails](https://grails.org/). In [Kaleidos](http://kaleidos.net) I've been moved to a new project developed with Grails so I need to be productive (ASAP! ^_^).

But, I don't know Groovy. Even more, I don't know Java. And I think it would be useful to document my own process. That said, let's start in the beginning...

<!-- more -->

**Disclaimer** Almost all the documentation I found for beginners assumes the student knows Java; most tutorials start with the differences between Groovy and Java. As I don't know Java, this posts are going to be slightly different. Maybe I'll make some comparisons with Python, but I'll try to keep those comparisons as anecdotal notes.



## Assumptions



You have a Linux operating system. You know the command line. You know git and github. You know something about programming (variables, methods, class...). You can check the specific syntax on your own.



## What is Groovy



From the oficial documentation:


<blockquote>
Groovy is a powerful, optionally typed and dynamic language, with static-typing and static compilation capabilities, for the Java platform aimed at multiplying developers’ productivity thanks to a concise, familiar and easy to learn syntax. It integrates smoothly with any Java program, and immediately delivers to your application powerful features, including scripting capabilities, Domain-Specific Language authoring, runtime and compile-time meta-programming and functional programming.
</blockquote>





## Installing Groovy



To manage and install Groovy components, it's recommended to use [GVM tool](http://gvmtool.net/).



    $ curl -s get.gvmtool.net | bash




Once you have installed _gvm_, you need to add the following to your _.bashrc_ file:



    #THIS MUST BE AT THE END OF THE FILE FOR FVM TO WORK!!
    [[ -s "/path/to/.gvm/bin/gvm-init.sh" ]] && source "/path/to/.gvm/bin/gvm-init.sh"




Now we can install Groovy:



    $ gvm install groovy




Good! Next step.



## Hello world



Now you have installed Groovy, you have some tools like _goovyConsole_, or _groovysh_. Go to a terminal and type:



    $ groovysh




A groovy interpreter is open, and we're going to make our _Hello, world_ with something like this:



    aran :: ~/formacion/groovy101 » groovysh
    Groovy Shell (2.3.9, JVM: 1.8.0_25)
    Type ':help' or ':h' for help.
    -----------------------------------------------------------------
    <strong>groovy:000> println "Hello, world!"
    Hello, world!</strong>
    ===> null
    groovy:000>




WIN! Our first Groovy command ;-) Keep going!

**Note**: for specific issues concerning _groovysh_, check [this page](http://groovy-lang.org/groovysh.html).

Now, create a file called _hello.groovy_ and write the following:



    println "Hello, world!"




And in the terminal, type:



    $ groovy hello.groovy




You should have something like this:



    aran :: ~/formacion/groovy101 » groovy hello.groovy
    Hello, world!
    aran :: ~/formacion/groovy101 »




Ou yeah! This is the beginning; next steps: the infinite, and beyond.



## groovyCalc



It's the moment to build a simple calculator. Let's create a directory with these two files:



    aran :: ~/formacion/groovy101 » ls groovyCalc
    Calc.groovy
    CalcTest.groovy




In the _Calc.groovy_ file, you have to write the actual code, while in the _CalcTest.groovy_ file (using [jUnit](http://groovy-lang.org/testing.html)), you write the tests. In [this repository](https://github.com/yamila-moreno/groovyCalc) you can find an example with code and tests. You can clone it, or write one by yourself (recommended).

To run these tests:



    $ groovy CalcTest.groovy




It only displays errors; so "no news, good news" ;-) Now you can extend the calculator, or you can create a new simple project to learn the Groovy syntax. And don't miss the tests, they are also very important!



## Bonus: Travis



Even if those tests are very very simple, I think it's useful to integrate them in a continous integration tool. I have tried with [Travis CI](https://travis-ci.org/), but you could use any other you feel confortable with.

If you have never used Travis, you can read [this post Travis 101 (spanish)](http://moduslaborandi.net/continuous-integration-101-travis/) to get used. The _.travis.yml_ file for this project would be:




    language: groovy
    groovy:
        - 2.3.9
    jdk:
        - oraclejdk8
    script:
        - groovy CalcTest.groovy




**Note**: the _jdk_ line is to force to use java8, instead of java7 (default). Currently, it's not supported openjdk8, only oraclejdk8.

You can check the [status of this repository](https://travis-ci.org/yamila-moreno/groovyCalc).

And that's all for now. From this moment, you can make some _katas_ and exercises to learn the Groovy syntax. Next steps, we will learn something about _gradle_ and _spock_. But now it is your turn to improve.

I would love to read your comments and feedback on this kind of posts :)
