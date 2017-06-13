---
author: yamila
comments: true
date: 2015-01-27 14:47:02+00:00
slug: groovy-101-gradle-travis-and-jenkins
title: 'Groovy 101 (III): Gradle, Travis and Jenkins'
wordpress_id: 774
tags:
- git
- gradle
- groovy
- jenkins
- travis
---

After my winter holidays, I come back with this self-learning process. My first idea was to learn to compile Groovy from command line: not a good idea. That's not its natural way and, as far as I have seen, the command line with Groovy is for simple scripts.

That said, the next step is [Gradle](https://www.gradle.org/), a tool to build and manage applications.

<!-- more -->

_Gradle_ is a tool to help us creating applications. Not only Groovy apps, but I'm focused on this part. It helps us to:




  * create the application and a basic tree directory
  * compile the code
  * compile the documentation
  * run the tests



And many other functionalities. For the scope of this post we are taking only a few of them. First of all, install _Gradle_ (you should have [GVM](http://moduslaborandi.net/groovy-101-groovycalc-with-tests-and-travis-ci-integration/) installed):




    $ gvm install gradle




To check if everything went ok:



    $ gradle -v




Now, we are creating an application to manage lists of books (yeah, I know, you weren't expecting something so original ;-)):




    $ mkdir gradleBooks
    $ cd gradleBooks
    $ gradle init --type groovy-library




With this we have new directories and files:



    aran :: gradleBooks » tree
    .
    ├── build.gradle
    ├── gradle
    │   └── wrapper
    │       ├── gradle-wrapper.jar
    │       └── gradle-wrapper.properties
    ├── gradlew
    ├── gradlew.bat
    ├── settings.gradle
    └── src
        ├── main
        │   └── groovy
        │       └── Library.groovy
        └── test
            └── groovy
                └── LibraryTest.groovy

    7 directories, 8 files




Without the `--type` option, it creates a simplier structure for a `java` application. Brief overview of this tree:




  * `build.gradle`: it's a configuration script to define projects, tasks and dependencies. Take a look at line 11: `apply plugin: 'groovy'`.
  * `gradlew`: the gradle wrapper. A script to build the project
  * `gradle.bat`: the gradle wrapper, for Windows environments
  * `settings.gradle`: it's used for managing dependencies in multimodule projects
  * `src` directory: this is the main project directory
  * `src/main` directory for the code
  * `src/test` directory for the tests



Even with this little code, we can _build_ the project:



    aran :: gradleBooks » ./gradlew clean build




And `BUILD SUCCESSFULL!` Great!

Before going ahead with the project, we are going to prepare the environment:




  * integration with [Travis CI](https://travis-ci.org/repositories)
  * integration with [Jenkins CI](http://jenkins-ci.org/)



The `init` script creates a dummy `LibraryTest` with a dummy test, which fits perfect to our purpose. First of all, we have to run the tests:



    aran :: gradleBooks » ./gradlew test




If everything went well (as expected), you are not seeing any special output in the console. This script runs the tests and also generates html outputs; this reports are generated in `build/reports/tests/index.html`. So we are going to have a browser with this page to check the reports frequently. We will review this reports deeper later when we have more tests.



## Integration with Travis CI



In this blog, you can find some [this](http://moduslaborandi.net/groovy-101-groovycalc-with-tests-and-travis-ci-integration/) and [this](http://moduslaborandi.net/continuous-integration-101-travis/) 101 Travis Integration.

I have the [repository](https://github.com/yamila-moreno/gradleBooks) to integrate with Travis. Now we create our `.travis.yml`, as you can see [here](https://github.com/yamila-moreno/gradleBooks/blob/master/.travis.yml). As with Travis you can only see whether the tests pass or fail, we are building a more sophisticated continuous integration system with Jenkins.



## Integration with Jenkins CI



This is a _super 101_ Jenkins configuration. First of all, read [this post](http://moduslaborandi.net/continuous-integration-101-jenkins/) to install Jenkins. When the post starts configuring Python, come back :) You should be logged in the Jenkins panel.

Now, we are going to create a new `job` in Jenkins which will test automatically our Gradle application. We will also link the output of our tests with Jenkins, so we can see there the generated reports.


  * "New item"
  * Give it a name, for instance, _gradleBooks_ (the same name as the application), and select the type of project; we are going to choose `Freestyle project`. Then "OK"
  * Now you can see the configuration area of the project:
    * Description (optional)
    * In Source Code Management, select `GIT` as we installed the GIT plugin before. Enter the URL of the repository. Master branch in used by default.
    * In Build Triggers, select `Poll SCM`, which means that the system will execute the build periodically. In the Schedule box you can set how often; for instance:


> H/15 * * * *

Every 15 minutes, Jenkins will clone the repository, and if there's any changes, then it will execute the build again


* In Build section, select `Execute shell` and add the usual command:
    ./gradlew clean test
* In Post-build Actions, select `Publish JUnit test result report`. There, we can add the path to the XMLs:
    build/test-results/*.xml



And that's all! You can see the result in [my Jenkins instance](http://moduslaborandi.net:9876/job/gradleBooks/). Now we can continue developing our application and Jenkins will periodically check the tests. Anytime, you can go to the project panel and click `Build now`. Besides, in the Build History column, you can always view the reports: the console output and the tests results.

In the next post we will start building our Groovy application. For now, you should have learn:
- how to install gradle
- how to create a groovy application with gradle
- how to run tests
- how to integrate the tests in Travis
- how to integrate the tests in Jenkins

I hope it's being useful :) See you in the next post!
