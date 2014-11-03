---
layout:     post
title:      Installing Docker on Ubuntu 14.10
date:       2014-10-31 14:00:00
categories:
  - docker
  - drupal

---

This post is quick look at installing docker (and using it a bit) on a Ubuntu 14.10 based laptop.

I know many people don't think of Docker as a secure system when compared to virtual machines provided by something like KVM or Xen. But that is just one viewpoint.

Another way to look at Docker is that it's an easy way to use container technology to limit what applications can do on a server, ie. a simpler way to use MAC security concepts. If one looks at Docker as a replacement for virtual machines, then yes, it might not look as secure. But that is probably the wrong way to think about Docker, at least at this time. OK, so maybe no multi-tenancy yet. But single tenancy? Why not!

<!-- more -->

If you want to read up on Docker there is a nice [Docker cheat sheet](https://github.com/wsargent/docker-cheat-sheet) available. Also the official documentation is quite good.

### Step 1: Install Docker

Seems like the Docker package is still called ```docker.io```? I thought they had fixed that. There is another package named "docker" so they had to choose a different name. At least it seems the Docker command is just ```docker``` now.

```bash
curtis$ sudo apt-get install docker.io
SNIP!
curtis$ docker --version
Docker version 1.2.0, build fa7b24
```

At first, you won't be able to run Docker commands without sudo. Permission denied!

```bash
curtis$ docker ps
2014/10/31 10:10:25 Get http:///var/run/docker.sock/v1.14/containers/json: dial unix /var/run/docker.sock: permission denied
```

If that's OK with you, then don't do the next step.

### Step 2: Add your user to the Docker group

Now add your user to the Docker group.

```bash
curtis$ sudo addgroup curtis docker
```

At this point you should relogin, but if you don't feel like doing that you can run:

```bash
curtis$ sudo su - curtis
```

to relogin as yourself in order to have the Docker group permissions.

Then restart Docker.

```bash
curtis$ sudo service docker restart
```

Now I can run Docker commands without sudoing to root.

```bash
curtis$ docker info
Containers: 0
Images: 0
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Dirs: 0
Execution Driver: native-0.2
Kernel Version: 3.16.0-24-generic
Operating System: Ubuntu 14.10
WARNING: No swap limit support
curtis$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### Step 3: Start using Docker

As an example, let's install Drupal via Docker.

```bash
curtis$ cd ~/docker/
curtis$ git clone git@github.com:ccollicutt/docker-drupal.git
curtis$
```

Next we'll build the image.

```bash
curtis$ docker build -t drupal .
Sending build context to Docker daemon   128 kB
Sending build context to Docker daemon
Step 0 : FROM    debian:wheezy
 ---> 61f7f4f722fb
Step 1 : MAINTAINER Ricardo Amaro <mail_at_ricardoamaro.com>
 ---> Running in b3442ebbab36
 ---> d504c0f689ce
Removing intermediate container b3442ebbab36
Step 2 : RUN apt-get update
 ---> Running in 9217ffa15398
Get:1 http://security.debian.org wheezy/updates Release.gpg [836 B]
Get:2 http://security.debian.org wheezy/updates Release [102 kB]
Get:3 http://http.debian.net wheezy Release.gpg [1655 B]
Get:4 http://http.debian.net wheezy-updates Release.gpg [836 B]
Get:5 http://http.debian.net wheezy Release [168 kB]
Get:6 http://http.debian.net wheezy-updates Release [124 kB]
SNIP!
Removing intermediate container f3df93fa04b6
Step 15 : EXPOSE 80
 ---> Running in b3a940ab9909
 ---> a253747b933c
Removing intermediate container b3a940ab9909
Step 16 : CMD ["/bin/bash", "/start.sh"]
 ---> Running in 13003e29d659
 ---> 3a75b3b16395
Removing intermediate container 13003e29d659
Successfully built 3a75b3b16395
```

Now the image has been built.

```bash
curtis$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
drupal              latest              3a75b3b16395        30 seconds ago      416.5 MB
debian              wheezy              61f7f4f722fb        10 days ago         85.1 MB
```

We can start up a Docker container based on that image.

```bash
curtis$ docker run -d drupal:latest
06ec202e2d1d09ab4649faa3cf41a551520f937bfff032bf3dfd3326037b9965
```

A container has been started.

```bash
curtis$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
06ec202e2d1d        drupal:latest       "/bin/bash /start.sh   45 seconds ago      Up 45 seconds       80/tcp              evil_ptolemy  
```

We can check out what IP it has. By default the IPs containers get are in the ```172.17.0.0/16``` network.

```bash
curtis$ docker inspect 06ec202e2d1d | grep 172
        "Gateway": "172.17.42.1",
        "IPAddress": "172.17.0.23",
```

If I go to that IP in a browser, the drupal login page will show up.

![drupal docker](https://raw.githubusercontent.com/flatsec/flatsec.github.io/master/images/posts/docker-drupal.png)

We can run ```docker log <container>``` to see what is going on. Most Dockerfiles for creating images will print login information and passwords and such to the log so that they can be accessed. Often each time passwords will be different. This probably isn't the best thing to do, ie. printing passwords to the log, so some Dockerfiles will allow environment variables for passwords, which is what I would suggest doing. But in this case we're just tossing up a quick drupal install for testing.

Below we can see ```Installation complete.  User name: admin  User password: admin``` so that is the Drupal login to use.

```bash
curtis$ docker log <container>
SNIP!
You are about to create a sites/default/settings.php file and DROP all tables in your 'drupal' database. Do you want to continue? (y/n): y
No tables to drop.                                                          [ok]
Starting Drupal installation. This takes a few seconds ...                  [ok]
Installation complete.  User name: admin  User password: admin              [ok]
141031 16:29:16 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
/usr/local/lib/python2.7/dist-packages/supervisor-3.1.3-py2.7.egg/supervisor/options.py:296: UserWarning: Supervisord is running as root and it is searching for its configuration file in default locations (including its current working directory); you probably want to specify a "-c" argument specifying an absolute path to a configuration file for improved security.
  'Supervisord is running as root and it is searching '
2014-10-31 16:29:25,083 CRIT Supervisor running as root (no user in config file)
2014-10-31 16:29:25,100 INFO RPC interface 'supervisor' initialized
2014-10-31 16:29:25,100 CRIT Server 'unix_http_server' running without any HTTP authentication checking
2014-10-31 16:29:25,101 INFO supervisord started with pid 389
2014-10-31 16:29:26,103 INFO spawned: 'httpd' with pid 392
2014-10-31 16:29:26,106 INFO spawned: 'memcached' with pid 393
2014-10-31 16:29:26,110 INFO spawned: 'mysqld' with pid 394
2014-10-31 16:29:27,330 INFO success: httpd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2014-10-31 16:29:27,331 INFO success: memcached entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2014-10-31 16:29:27,331 INFO success: mysqld entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```

Back in the browser, login to Drupal and start testing it out.

To remove the container, run ```docker stop <container> && docker rm <container>```.

### Conclusion

Even as someone who is interested in security I am still a big fan of Docker. I think at some point Docker will become as secure as KVM/Xen based virtual machines or OpenVZ containers, Solaris Zones, BSD Jails, etc, but there is still more work to do. However, if you take a different viewpoint of containers and do not try to compare them to virtual machines, there are situations where a Docker based system could be more secure, especially if you think of Docker as an easy way to use container technologies that provide additional separation from the underlying OS.

Also, Docker is not just a container sytem, it's also a software delivery system that allows IT workers to develop systems that can be deployed anywhere Docker can be. On its own that feature is quite powerful.
