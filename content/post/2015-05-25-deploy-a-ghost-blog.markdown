---
author: yamila
comments: true
date: 2015-05-25 16:39:25+00:00
slug: deploy-a-ghost-blog
title: Deploy a ghost blog
wordpress_id: 848
categories:
- Open source
- Tutorials
tags:
- blog
- ghost
- nginx
- postgresql
- supervisor
---

This weekend I discovered [Ghost](http://ghost.org) blog and I found it very interesting. I have been thinking of changing my [trip blog (ES)](http://dendarii.es) for a while, and ghost gave me the perfect excuse.

![Ghost-Logo-on-White](/images/2015/05/Ghost-Logo-on-White.png)

This tutorial shows how to install and deploy ghost with postgres, supervisor and nginx.

<!-- more -->



## Assumptions



To follow properly this post, you need to know something about: git, postgresql, supervisor and nginx. Let's start!



## Installation



First of all, get the source:




    $ mkdir my_blog
    $ cd my_blog
    $ wget -c https://ghost.org/zip/ghost-0.6.4.zip
    $ unzip ghost-0.6.4.zip




Now, we need to install the dependencies (you need [NodeJS](https://nodejs.org)):



    $ npm install --production




This may take a while depending on your bandwidth, but everything is ok, so go and take some fresh air. Great! After this command, you'll have a new directory called _node_modules_, where all the dependencies are installed. Let's move on. Now we're going to configure the blog:




    $ cp config.example.js config.js




This **config.example.js** is the best way to get the system configured quicky. With this, we can run our blog:




    $ npm start
    # And this is the expected output:
    > ghost@0.6.4 start /path/to/my_blog
    > node index
    # creating new tables
    Migrations: Database initialisation required for version 003
    Migrations: Creating tables...
    Migrations: Creating table: posts
    Migrations: Creating table: users
    Migrations: Creating table: roles
    Migrations: Creating table: roles_users
    Migrations: Creating table: permissions
    Migrations: Creating table: permissions_users
    Migrations: Creating table: permissions_roles
    Migrations: Creating table: permissions_apps
    Migrations: Creating table: settings
    Migrations: Creating table: tags
    Migrations: Creating table: posts_tags
    Migrations: Creating table: apps
    Migrations: Creating table: app_settings
    Migrations: Creating table: app_fields
    Migrations: Creating table: clients
    Migrations: Creating table: accesstokens
    Migrations: Creating table: refreshtokens
    Migrations: Populating fixtures
    Migrations: Populating permissions
    Migrations: Creating owner
    Migrations: Populating default settings
    Migrations: Complete
    Ghost is running in development...
    Listening on 127.0.0.1:2368

    #UP AND RUNNING!
    Url configured as: http://localhost:2368
    Ctrl+C to shut down




So, now you have your ghost blog running. You can take 3 minutes playing with it, in **http://localhost:2368** but don't create many contents, because we are changing the database to a postgresql. By default, the installation comes with sqlite as a database, as you have seen, which is a bad idea for production environments. To do this, it's necessary to set the dependencies. Edit **package.json** and look for the **optionalDependencies > pg** entry to read:




    "pg": "latest"




After this change, you need to reinstall dependencies with:




    $ npm install --production




Next step to configure postgres is edit the file **config.jg**, look for the **config > production > database ** section, and change the _filename_ entry with the following:




    database: {
      client: 'pg',
      connection: {
        host     : '127.0.0.1',
        port     : '5432',
        user     : 'myBlogUser',
        password : 'myBlogPassword',
        database : 'myBlogDatabase',
        charset  : 'utf8'
    },




Now, start again:




    $ npm start




And you have your blog up and running with postgresql. Now you can spend more time learning about its features, and adding some cute theme.



## Changing themes



You can also try a nice [theme](http://marketplace.ghost.org/themes/free/). To install a new theme:



    $ cd content/themes
    $ wget http://url/to/my/theme.zip
    $ unzip theme.zip




And the theme will be available after restarting (**npm start**) in **Settings > General**.

Cool, uh? Now, it's time for further steps; let's make this blog, production ready.



## Supervisor



To manage the process with supervisor, you have to create a new file in **/etc/supervisor/conf.d** (depending on the OS) named **myblog.ini** and write the following:




    [program:myblog]
    environment=NODE_ENV=production
    directory=/path/to/my_blog
    command=node index.js
    stdout_logfile=/path/to/logs/myblog.log
    stderr_logfile=/path/to/logs/myblog-err.log
    user=myuser




Now, with **root** privileges, **reread** configuration and **start myblog**.

Done! We are very close, last step!



## Nginx



It's time to configure nginx. So, create the file **/etc/nginx/sites-available/myblog** (root privileges), and write:




    server {
            listen 80;
            server_name myblog-domain.net;
            root /path/to/myblog;

            location / {
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header Host $http_host;
                    proxy_set_header X-Forwarded-Proto $scheme;
                    proxy_pass http://localhost:2368;
            }

            location /content/images/ {
                    root /path/to/myblog;
            }

            location /assets/ {
                    root /path/to/myblog/content/themes/mytheme;
            }
    }




It's pretty straightforward configuration, and the only thing you may notice is that there are two directories for statics. It's the way I could configurate, but, I would like to know a different way, feel free to share in the comments!

Congratulations, you did it!!! After this, you need to **restart nginx**, go to **http://myblog-domain.net** and enjoy your new brand blog.

**Migrating from wordpress**

Well, some of you (mom, dad, hi) may have your blog in Wordpress and would like to migrate the data. There are some [tutorials](https://ghostforbeginners.com/how-to-transfer-blog-posts-from-wordpress-to-ghost/) for this, but they suggest to move the images to the cloud, while I want to keep them with me, in my server. If that's your case, wait for the next post, about how to migrate our data and images from Wordpress to Ghost.

And also, I will put online my brand new blog. Stay tuned!!

**Thanks!** to Angela [Ghilbrae](http://twitter.com/ghilbrae) because her notes to deploy [her blog](http://blog.arandomtable.com/) set me in the right direction, and to Alex [lekum](http://twitter.com/lekum) for his help with supervisor configuration.

**Edit:** [Alberto](http://twitter.com/albertogargar) has followed this tutorial and added [some notes in his blog](http://blog.algargar.com/2015/08/15/agregando-procesos-a-supervisor-root/) (in spanish). Thanks for sharing!
