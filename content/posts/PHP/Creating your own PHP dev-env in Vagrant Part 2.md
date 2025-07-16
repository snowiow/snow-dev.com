---
title: 'Creating your own PHP dev-env in Vagrant: Part 2'
tags:
  - apache
  - bash
  - php
  - vagrant
categories:
  - PHP
  - Vagrant
date: 2016-06-03
description: In the last post we installed vagrant and enabled the vagrant settings we need in the Vagrantfile. We have set a base image of ubuntu 14.04, made a synced folder and enabled a private network connection between host and guest system. Now it's time to write our first shell scripts, which will configure our guest system to serve as a web server. Vagrant comes with a neat feature called provisioning. This can be shell scripts, which will be executed or files, which will be uploaded onto the guest system. Of course there are more provisioners to explore. For a full reference head over here. In this tutorial we will focus on these two provisioners.
---
# Table of Contents
* [[Create your own PHP dev-env in Vagrant|Part 1]]
* [[Creating your own PHP dev-env in Vagrant Part 3|Part 3]]
* [[Creating your own PHP dev-env in Vagrant Bonus 1|Bonus 1]]
* [[Creating your own PHP dev-env in Vagrant Bonus 2|Bonus 2]]
* [Full Code Base on GitHub](https://github.com/snowiow/vagrant-template)

In the last post we installed vagrant and enabled the vagrant settings we need
in the Vagrantfile. We have set a base image of ubuntu 14.04, made a synced
folder and enabled a private network connection between host and guest system.
Now it's time to write our first shell scripts, which will configure our guest
system to serve as a web server. Vagrant comes with a neat feature called
provisioning. This can be shell scripts, which will be executed or files, which
will be uploaded onto the guest system. Of course there are more provisioners
to explore. For a full reference head over
[here](https://www.vagrantup.com/docs/provisioning/). In this tutorial we will
focus on these two provisioners. Before we setup our apache webserver, let's
start with a simple script, which should run before. This script will update
the ubuntu system and should be a great intro into provisioning. First we
create a new directory inside of our vagrant directory, called __sh__. This
directory will contain all of our shell scripts, which we will write during
this tutorial. Everything could be in a single shell script, but I like to
split the shell scripts into specific categories. The first will be called
_update.sh_ and contains the the following content:

``` bash
#!/usr/bin/env bash

apt-get update
```

The first line will be in every script we write. It's a dynamic binding to tell
the operating system where to find the program, which should execute this
script. Afterwards we update the system by executing _apt-get update_. As you
can see, we don't need to issue _sudo_ rights. Everything inside these shell
scripts will be run as a super user automagically. Now we need to get vagrant
to provision this file. That's why we add the following line after the
definition of our base system.

``` ruby
config.vm.box = "ubuntu/trusty64"
config.vm.provision "shell", path: "sh/update.sh"
```

This statement is pretty straight forward. We tell vagrant that we want to
provision something, which is of the type _shell_ and the path to the shell
script is _sh/update.sh_. If we would start our system now, it would do a
system update at the beginning. Now that we are working on the latest state of
the system, we can start adding new software. It's time to create the apache
webserver. We add a new filed called _apache.sh_ to the sh directory, which we
created earlier. We start off in the same way, by downloading the software via
apt-get:

``` bash
apt-get install -y apache2
```

Followed by this:

``` bash
if ! [ -L /var/www/html ]; then
    rm -rf /var/www/html
    ln -fs /vagrant/projects /var/www/html
fi
```

Here we delete any symlinks on the _/var/www/html_ location, if they exist and
create a new one afterwards. The symlink will be between our projects
directory and the apache web server. So everything which changes in one place,
will automatically change in the other location as well. It's like the synced
folders, but it's an inter system feature of linux. The next part is
optional. I copied the whole _apache.conf_ out of it's base installation and
keep it on my host system to change some things. On system startup I copy it
back onto the guest and replace the base config with it.

``` bash
if [ -f /vagrant/tmp/apache2.conf ]; then
    mv /vagrant/tmp/apache2.conf /etc/apache2/apache2.conf
else
    >&2 echo "Error: apache2.conf not found"
fi
```

This is a check, if a file is in the _vagrant/tmp_ directory, called
_apache2.conf_. If there is one, it will be moved over to the position of the
original file. Otherwise an error is printed to the error channel of the
console. But how do we get the file to the _tmp_ directory? Remember that shell
provision isn't the only thing? Because now file provisioning comes into play.

``` ruby
config.vm.provision "file", source: "conf/apache2.conf", destination: "/vagrant/tmp/apache2.conf"
```

We add this line under the shell provision. This copies the file from the
_conf_ directory to the _tmp_ directory of the guest system. Why we need to do
this extras step, you may ask? Why not copy it directly to the original
location? Well, it's pretty simple: In the provision context you don't have any
root rights. But later in the shell script you do. That's why we provide a
place for our files, where the shell scripts have access to later. The next
thing I do, is changing the rights on the webspace and the log directory, to be
able to create and read files without any problems.

``` bash
chmod -R 755 /var/www
chmod -R 755 /var/log
```

The last thing I do is activating mod rewrite, which allows web applications to
rewrite the routing of requests. Many frameworks rely heavily on this feature.

``` bash
a2enmod rewrite
```

And this is our full _apache2.sh_ script:

``` bash
#!/usr/bin/env bash

apt-get install -y apache2

#Symlink the apache webspace to the shared folder of the host and guest syste,
if ! [ -L /var/www/html ]; then
    rm -rf /var/www/html
    ln -fs /vagrant/projects /var/www/html
fi

#Copy the apache2.conf to the right location
if [ -f /vagrant/tmp/apache2.conf ]; then
    mv /vagrant/tmp/apache2.conf /etc/apache2/apache2.conf
else
    >&2 echo "Error: apache2.conf not found"
fi

#grant permission to webserver
chmod -R 777 /var/www
chmod -R 777 /var/log

#enable mod_rewrite
a2enmod rewrite
```

In the next part we will reach forward to install php and mysql. You already
learned the main functionalities, which we need to get our dev-env running.
From now on it gets far more repetitive, but some small challenges are still
waiting on our way. [[Creating your own PHP dev-env in Vagrant Part 3|Goto Part 3]]
