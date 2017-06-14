---
author: yamila
comments: true
date: 2017-06-14 10:02:22+00:00
slug: swarm-101-create-a-cluster
title: Swarm 101 - Create a cluster
tags:
- docker
- swarm
thumbnailImagePosition: left
thumbnailImage: https://ih0.redbubble.net/image.282529586.5735/flat,200x200,075,f.u2.jpg
---

Nuevo tutorial para principiantes, para iniciarse en Swarm, una herramienta de orquestación que viene por defecto con docker. Este tutorial lo he escrito después de seguir <a href="https://docs.docker.com/engine/swarm/swarm-tutorial/" target="_new">el tutorial de la documentación oficial</a> y tras destilar lo que entiendo que es más útil para empezar.
<!--more-->

Igualmente, añado algunas explicaciones de lo que personalmente más me costó entender.

<h2>Previously in Kubernetes</h2>

Hace unos meses hice un tutorial de <a href="" target="_new">Kubernetes</a>, el orquestador de Google, y que actualmente está teniendo más tracción. Durante el tutorial y después, me sentí muy así:

<figure>
  <img src="https://i1.wp.com/vinodvihar75.files.wordpress.com/2014/12/dogs-2.jpg?w=290&h=217&crop&ssl=1" alt="Dog typing in a keyboard" />
  <figcaption>Don't know what I'm doing</figcaption>
</figure>

Me pareció una herramienta muy sofisticada y muy poco beginer-friendly. Esto tenía sentido dado que es una herramienta relativamente nueva, y esto afecta a la madurez de la documentación; además, está resolviendo un problema complejo, y que estaba totalmente fuera de mi zona de conocimiento. Así que la conclusión fue que los orquestadores no eran lo mío.

Sin embargo, tras dedicar un par de semanas a estudiar y probar las cuestiones básicas de Swarm, creo que sí hay una forma de acercarse a los orquestadores más adecuada para beginers. ¡Así que vamos allá!

<h2>Objetivo detallado</h2>

He consultado varios tutoriales sobre Swarm, y he visto que hay distintas aproximaciones, así que considero útil compartir el alcance concreto de este tutorial. Lo que pretendo que veamos es:

* conceptos básicos de orquestación: estado, réplicas, nodos, manager, worker, routing
* comandos para crear un cluster de Swarm
* comandos básicos para desplegar imágenes en un Swarm
* cómo escribir un <code>docker-compose.yml</code> en versión 3, que pueda usarse con Swarm

<h2>Assumptions</h2>

Este tutorial asume que entiendes lo básico de docker. Si es la primera vez que vas a tocar docker, te recomiendo que dediques un rato a este <a href="" target="_new">tutorial de introducción a docker</a> y a este otro <a href="" target="_new">tutorial de introducción a Dockerfile</a>.

Además, pondré un ejemplo más detallado que empieza una vez que hemos conseguido poner nuestra aplicación en una imagen de docker accesible a través de un registry. Este tema lo he tratado recientemente en esta <a href="" target="_new">introducción a Gitlab CI</a>

<h2>¡Empezamos!</h2>

Queremos aprender a orquestar con Swarm. **Orquestación** es el conjunto de herramientas y prácticas para poner en producción aplicaciones que puedan estar en alta disponibilidad (el servicio siempre responde), donde la infraestructura escale rápidamente (si hay un pico de demanda, el servicio se puede hacer más grande de forma rápida). Para ello vamos a usar el <em>modo Swarm</em> que viene por defecto con docker. Y necesitaremos un **cluster**, que es el conjunto de máquinas físicas disponibles para Swarm donde desplegamos los workers de Swarm. Estas máquinas han de tener visibilidad entre ellas.

Vamos a montarnos un cluster en nuestra máquina que replique lo que podría ser un cluster en la nube: vamos a usar Vagrant para levantar 3 máquinas virtuales (nodos). Podéis usar <a href="https://github.com/yamila-moreno/vagrant-cluster" target="_new">este Vagrantfile</a> y su provisión. Si hacéis <code>vagrant up</code> junto al <em>Vagrantfile</em>, se levantarán 3 máquinas virtuales con ubuntu y con docker instalado. Lo podéis comprobar con <code>vagrant status</code>.

{{< alert warning >}} No ejecutéis los comandos de Swarm en vuestra máquina directamente; activar el modo Swarm, modifica la forma de trabajar de vuestro docker y probablemente os toque dar marcha atrás. Lo mejor es que uséis las VMs en Vagrant. {{< /alert >}}

Swarm trabaja distribuyendo los servicios entre las máquinas del cluster. Cada máquina, por defecto, es un **worker**; pero además, algunas máquinas son **managers**: estos managers conocen el estado del cluster, si un nodo se cae, lo detectan automáticamente y ponen en marcha medidas para recuperar el **estado de la aplicación** deseado. En un cluster de Swarm siempre debe haber número impar de managers de forma que puedan usar algoritmos de consenso.

Así que lo primero que vamos a hacer es inicializar una de las máquinas como manager (server1) y vamos a asociar el resto (server2 y server3) como workers:

```
# accedemos a server1
(myhost)$ vagrant ssh server1
(server1)$ docker swarm init --advertise-addr 10.0.15.21
```

Este comando nos devuelve el comando que debemos usar para asociar el resto de máquinas al cluster de Swarm. Algo similar a:
```
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

Si por lo que sea perdemos este comando porque se nos cierra la ventana o similar, podemos recuperarlo ejecutando lo siguiente desde la máquina con el manager:
```
(server1)$ docker swarm join-token worker
```

Ahora entramos en las otras dos máquinas y ejecutamos el <code>docker swarm join --token...</code> en server2 y server3. ¡Listo! Ya tenemos un cluster de Swarm montado. Un par de comandos útiles son:
```
# para ver los nodos que intervienen en el cluster
(server1)$ docker node ls
# para ver qué servicios hay en el cluster
(server1)$ docker service ls
ID                            HOSTNAME     STATUS     AVAILABILITY     MANAGER STATUS
niqaah8mhlou9gij2fye046sq *   server1      Ready      Active           Leader
qowv0xmux806zu5u7f7or4yed     server2      Ready      Active
w92gfg7dwl33esmmk7acnhcdk     server3      Ready      Active
```
El asterisco indica el nodo en el que estás en ese momento, y <em>Leader</em> indica que ese es un nodo manager.

¡Bien hecho! Hasta aquí hemos visto algunos conceptos principales y hemos creado un cluster. El siguiente post veremos cómo crear servicios, actualizarlos e inspeccionarlos. Pero ahora puede ser un buen momento para que estires las piernas. ¡Hasta el próximo post!
