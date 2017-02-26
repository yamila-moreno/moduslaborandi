---
author: yamila
comments: true
date: 2016-04-23 23:32:05+00:00
layout: post
slug: wordpress-and-nginx-bonus-vagrant
title: 'Wordpress and Nginx (bonus: Vagrant)'
wordpress_id: 1091
categories:
- Open source
- Tutorials
tags:
- my modus
- nginx
- vagrant
- wordpress
---

This post is about installing and configuring a Wordpress and serving it with Nginx. And a very plain introduction to Vagrant. Try to follow the tutorial step by step, and don't forget to comment! ;-)

![wnv](/images/2016/04/wnv.png)

<!-- more -->



##  Vagrant



First of all, we're going to use [Vagrant](https://www.vagrantup.com/). It's a tool to manage VirtualBoxes from command line.




    Install VirtualBox: https://www.virtualbox.org/
    Install Vagrant: https://www.vagrantup.com/




**Note**: Archlinux users, shoud install _net-tools_ to use Vagrant.

Once you've installed VirtualBox and Vagrant, next step is creating a Vagrant file to configure our virtual machine:




    $ vagrant init
    # and then
    $ vim Vagrantfile




Edit Vagranfile, remove all the content and write this down:



    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure(2) do |config|
      config.vm.box = "debian/jessie64"
      config.vm.network "private_network", ip: "10.0.15.20"
    end



We're using an official _Debian Jessie_, and we're creating a private network, so we can access the virtual machine from our host.

Then, running:



    $ vagrant up




in the directory where Vagranfile is, the system runs the virtualbox defined. After that, with:




    $ vagrant ssh




we can enter the virtual machine to configure our next steps. To check that we have everything ok, we can enter the virtual machine and run:



    $ python -m SimpleHTTPServer



and from the browser in our host,  go to _http://10.0.15.20:8000_ and check you see the files listed.

A couple of useful commands:



    $ vagrant halt # will stop the virtualbox
    $ vagrant destroy # will remove the virtualbox




Great! First big step completed. Now, let's install a Wordpress and rock'n'roll. Ok, maybe just install a Wordpress.



## Wordpress



Since this point, by default all the commands will be executed inside the virtual machine.




    $ sudo apt-update # let's update apt repositories
    $ sudo apt-get install mariadb-client mariadb-server php php5-mysql unzip vim




Let's configure the database:



    $ mysql -u root -p
    MariaDB [(none)]> CREATE DATABASE wordpress;
    MariaDB [(none)]> CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost;
    MariaDB [(none)]> FLUSH PRIVILEGES;
    MariaDB [(none)]> exit




Next, download and configure wordpress:



    $ cd /home/vagrant
    $ wget -c wget -c https://wordpress.org/latest.zip
    $ unzip latest.zip
    $ cd wordpress
    wordpress$ cp wp-config-example.php wp-config.php
    wordpress$ vim wp-config.php




Edit _wp-config.php_ adding the credentials of the database. Also, get a [new secret-key](https://api.wordpress.org/secret-key/1.1/salt/) and add to the file (remove the previous lines).

Our last step configuring Wordpress is to prepare permissions:



    $ cd wordpress/wp-content
    $ mkdir uploads
    $ sudo chown www-data:www-data uploads




And level completed! Awesome. Let's go with the final step.



## Nginx



Time to install some pieces of software:



    $ sudo apt-get install nginx php5-fpm




And now, we have to configure _php5-fpm_. In the directory _/etc/php5/fpm/pool.d_ we're going to create for each site wordpress we need.



    $ sudo vim /etc/php/fpm/pool.d/mysite.conf
    [mysite]
    user = www-data
    group = www-data
    listen = /tmp/php5-fpm-mysite.sock
    listen.owner = www-data
    listen.group = www-data
    pm = dynamic
    pm.max_children = 5
    pm.start_servers = 2
    pm.min_spare_servers = 1
    pm.max_spare_servers = 3
    chdir = /




Restart the service _php5-fpm_ and check if the socket is in _/tmp_.

Finally, we have to configure Nginx; create a new virtualhost with the following configuration:



    server {
            listen 80;

            server_name mysite.net www.mysite.net;

            root /home/vagrant/wordpress/;
            index index.html index.php;

            location = /favicon.ico {
                    log_not_found off;
                    access_log off;
            }

            location / {
                    # This is cool because no php is touched for static content.
                    # include the "?$args" part so non-default permalinks doesn't break when using query string
                    try_files $uri $uri/ /index.php?$args;
            }

        location ~ \.php$ {
              try_files $uri = 404;
              fastcgi_split_path_info ^(.+\.php)(/.+)$;
              include fastcgi_params;
              fastcgi_index index.php;
              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
              fastcgi_intercept_errors on;
              fastcgi_pass unix:/tmp/php5-fpm-mysite.sock;
            }

            location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                    expires max;
                    log_not_found off;
            }

    }



Restart Nginx and, Bob's your uncle, you have your new fantastic wordpress served with nginx.



## Success



Let's check the result. **In your host**, edit your _/etc/hosts_ file and add:



    10.0.15.20      mysite.net



Open your brower, go to **mysite.net** and there you are!! If you liked it, share it and drop me a line in the comments section!

Credits: I've used the sql commands directly from this [post](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-nginx-on-ubuntu-14-04).

