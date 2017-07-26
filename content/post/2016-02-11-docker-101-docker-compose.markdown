---
author: yamila
comments: true
date: 2016-02-11 21:43:17+00:00
slug: docker-101-docker-compose
title: Docker 101 - docker-compose
wordpress_id: 1066
tags:
- docker
- my modus
---

Third post of this _Docker 101_ series. We've seen what containers are, how to build them, how to run them, how to manage visibility. All of this under the _docker_ command.

In this post we're learning about **docker-compose** which, according to the [official documentation](https://docs.docker.com/compose/) is _a tool for defining and running multi-container Docker applications._. Just what we need right now!

<!-- more -->

It's important that you have present [previous](http://moduslaborandi.net/2016/02/docker-101-hello-world/) [posts](http://moduslaborandi.net/2016/02/docker-101-dockerfile/) to go forward.

Docker-compose is a tool to build and run multiple containers and deal with its configuration from a single file, which is called **docker-compose.yml**. In the scope of this introduction we're only learning the _running_ options, so we'll keep almost all the _Dockerfiles_.

**IMPORTANT**: docker-compose is a tool written in Python. But it's only compatible with Python2.7 (yeah, from the past). That said, you'll need to create a new virtualenv with Python2.7 (called "docker") and install _docker-compose_.

Move on. We have our tree directory:



    docker-101
    |__api-example
       |__src
       |__docker
          |__postgres
          |__api




**Reminder**: We are executing all the commands in the "docker-101" directory.

You can remove the "postgres" directory, because we're not using it any longer (I'm leaving it in the repository because it's needed in the previous posts). Besides, remove "myapp-postgres" image:



    $ docker rmi myapp-postgres




In the "docker" directory, add a file called **docker-file.yml**. If you're not familiar with the extension, it's a file in yaml format. Easy to use and understand. And let's add content to the file to replace our deleted image:




    myapp-postgres:
        image: postgres:9.4.5
        container_name: myapp-postgres
        environment:
            - "POSTGRES_DB=myapp"




Take a look at the file. We're creating a new key named "myapp-postgres", which parts from the image of postgres; beside, we're giving it a name and an environment variable. Just as we did with the Dockerfile and _docker run_, remember? If we want to run this image in a container:



    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-postgres




Now we can see this a running container with Postgres. Exercise: try to enter in the container and take a look at the database. Is there a database named "myapp"? Congratulations!

**Note**: If you don't pass any argument to the "compose up", docker-compose will launch all the services defined in the yaml file.

It's time to see the potential of _docker-compose_. Add the following code at the end of the file:



    myapp-api:
        image: myapp-api:1.0
        container_name: myapp-api
        ports:
            - "5005:5005"




You can probably understand all the information in the YAML file: we're creating a new key called "myapp-api", to run our well known "myapp-api:1.0" image. We're giving it a name and we're exposing the inner port (5005) in our host machine (also, 5005). Let's run the image in a new container:



    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-api




To check if everything went well, try to get the API:



    $ curl http://localhost:5005/api/v1




Great! So now, in one file we can set all the information to run two containers. But it seems not too epic to run just two independent containers. With docker-compose we can easily create an ecosystem of linked containers.



## Our new scenario



Our new scenario is sligthly different from the previous one:




  * We keep having our api
  * We keep having our postgres; but in this scenario, the api will be using the postgres_
  * We'll serve everything through a container with nginx



So, let's go step by step. First of all, take a look at the **api-example** code: you'll see that it's expecting some environment variables, which will be useful to deploy. From this point, I'm assuming that you know enough Python to understand the code in the api-example. If you're not familiar with this, drop me a line and I'll try to explain it to you, but I'll pass over that here because it's out of the scope of this tutorial.

Right now, we have our configuration file "myapp.ini" where it's said that the database will be sqlite. Thanks to environment variables, we can configure our postgres database. We also need to link our api with the database. Let's change our _docker-compose.yml_; after editing, it should look like:



    myapp-postgres:
        image: postgres:9.4.5
        container_name: myapp-postgres
        environment:
            - "POSTGRES_DB=myapp"

    myapp-api:
        image: myapp-api:1.0
        container_name: myapp-api
        ports:
            - "5005:5005"
        links:
            - myapp-postgres:myapp-postgres
        environment:
            - "MY_APP_DB_DRIVERNAME=postgres"
            - "MY_APP_DB_DATABASE=myapp"
            - "MY_APP_DB_HOST=myapp-postgres"
            - "MY_APP_DB_PORT=5432"
            - "MY_APP_DB_USERNAME=postgres"
            - "MY_APP_SERVER_PORT=5005"
        command: /venv/bin/gunicorn -b 0.0.0.0:5005 --access-logfile - --error-logfile - --log-level debug 'wsgi:load_application("create-db", "with-fixtures")'




We have just added our environment variables to use postgres, and the rest of lines are already familiar to us. So now we can run again our images (stop and remove the running containers before continuing):



    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-postgres
    # wait for some seconds to guarantee that the postgres image is completely up and running
    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-api




You can always check the status of your containers with _docker ps -a_ to see if everything went ok. If by any reason, there is a container with the STATUS "Exited", you can take a look at the logs of the container:



    $ docker logs CONTAINER-ID




Quite a lot of things!! With all this work, we have our system up and running with two containers, easy to build, even easier to run. It's fast and anyone in your team could have the same environment.




## Nginx



The last section of this tutorial is a little exercise adding a new container with Nginx. We're going to build a new image and add it to the docker-compose.yml

First, pull the nginx image:



    $ docker pull nginx:1.9.10




Also, add a new directory "nginx" next to "api", and two files. One, the _Dockerfile_:



    FROM nginx:1.9.10

    # forward request and error logs to docker log collector
    RUN ln -sf /dev/stdout /var/log/nginx/access.log
    RUN ln -sf /dev/stderr /var/log/nginx/error.log

    COPY api-example/docker/nginx/myapp.conf /etc/nginx/conf.d/default.conf

    VOLUME ["/var/cache/nginx"]

    EXPOSE 80 443

    CMD ["nginx", "-g", "daemon off;"]




and two: _myapp.conf_, where we will configure the nginx to serve our application:



    server_names_hash_bucket_size 128;
    server_names_hash_max_size 512;

    server {
        listen 80;
        server_name myapp.net;

        client_max_body_size 500M;
        charset utf-8;

        #            _____  _____
        #     /\    |  __ \|_   _|
        #    /  \   | |__) | | |
        #   / /\ \  |  ___/  | |
        #  / ____ \ | |     _| |_
        # /_/    \_\|_|    |_____|
        location /api/ {
            proxy_pass http://myapp-api:5005/api/;
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
        }
    }




Now we need to build the new image:



    $ docker build -t myapp-nginx:1.0 -f api-example/docker/nginx/Dockerfile .




If we're using nginx, we don't need to expose the gunicorn port, it will be a job for nginx. Thus, we need to change our _docker-compose.yml_ a little bit:



    myapp-postgres:
        image: postgres:9.4.5
        container_name: myapp-postgres
        environment:
            - "POSTGRES_DB=myapp"

    myapp-api:
        image: myapp-api:1.0
        container_name: myapp-api
        links:
            - myapp-postgres:myapp-postgres
        environment:
            - "MY_APP_DB_DRIVERNAME=postgres"
            - "MY_APP_DB_DATABASE=myapp"
            - "MY_APP_DB_HOST=myapp-postgres"
            - "MY_APP_DB_PORT=5432"
            - "MY_APP_DB_USERNAME=postgres"
            - "MY_APP_SERVER_PORT=5005"
        command: /venv/bin/gunicorn -b 0.0.0.0:5005 --access-logfile - --error-logfile - --log-level debug 'wsgi:load_application("create-db

    myapp-nginx:
        image: myapp-nginx:1.0
        container_name: myapp-nginx
        links:
            - myapp-api:myapp-api
        ports:
            - "5001:80"




Look carefully at the ports mapping in "myapp-nginx" entry. We are exposing the port 80 in the container (through nginx) in the port 5001 in our host. Ok, final test, let's run everything



    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-postgres
    # wait for some seconds to guarantee that the postgres image is completely up and running
    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-api
    (docker) $ docker-compose -f api-example/docker/docker-compose.yml up -d myapp-nginx




To check if everything went ok, we need to edit our _/etc/hosts_ file (in our machine!) and add the following line:



    127.0.0.1       myapp.net




and run:



    $ curl myapp.net:5001/api/v1




Congratulations!! You nailed it! The final reward of hard job is an up&running; environment. And some rest, you should take some rest, you deserve it! ;-)

Those posts were quite hard, but I hope that now you have an overview of the very basics of Docker. The [official documentation](https://docs.docker.com/), which I've linked several times, it's the perfect place to keep learning. There are a couple of ideas that may inspire you to try new things:




  * use docker as your development environment, using "VOLUME"s
  * use docker-compose also to build images
  * learn about registry and how could it be useful in your organization



If you have any questions about the tutorial, maybe something that isn't clear, please use the comments; they'll probably be useful to others. Besides, feel free to drop me a line about general feedback of this tutorial.

Happy hacking!

