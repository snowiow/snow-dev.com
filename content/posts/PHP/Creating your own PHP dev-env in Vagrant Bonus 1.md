---
title: "Creating your own PHP dev-env in Vagrant: Bonus 1"
tags:
  - apache
  - php
  - vagrant
categories:
  - PHP
  - Vagrant
date: 2016-06-03
description: In the previous posts we've successfully set up a development environment for PHP. In the following posts I will present some bonus things, which you can do, to optimize your work with vagrant. In this post I will show you how to set a virtual host in apache inside your guest system. Because IP addresses can be forgotten quite easily, it's much more handy to have a short named address under which you can access your web app. In this project we will create a virtual host for our main project directory called mysite.dev. So let's get started!
---

# Table of Contents

* [[Create your own PHP dev-env in Vagrant|Part 1]]
* [[Creating your own PHP dev-env in Vagrant Part 2|Part 2]]
* [[Creating your own PHP dev-env in Vagrant Part 3|Part 3]]
* [Bonus 2](/posts/creating-your-own-php-dev-env-in-vagrant-bonus-2.html)
* [Full Code Base on GitHub](https://github.com/snowiow/vagrant-template)

In the previous posts we've successfully set up a development environment for
PHP. In the following posts I will present some bonus things, which you can do,
to optimize your work with vagrant. In this post I will show you how to set a
virtual host in apache inside your guest system. Because IP addresses can be
forgotten quite easily, it's much more handy to have a short named address
under which you can access your web app. In this project we will create a
virtual host for our main project directory called _mysite.dev_. So let's get
started! First we create a config file inside our _conf_ directory with the
following content:

``` apache
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName mysite.dev
    ServerAlias www.mysite.dev
    ErrorLog "/var/log/apache2/mysite.dev-error_log"
    CustomLog "/var/log/apache2/mysite.dev-access_log" common
    <Directory "/var/www/html">
       Order allow,deny
       Allow from all
       AllowOverride All
   </Directory>
</VirtualHost>
```

This is the virtual host definition. The first line says that it's listening on
port 80, which is the default port. The _DocumentRoot_ is the path to
our webspace, which was symlinked to the vagrant synced directory in an earlier
post. _ServerName_ is the address, how you can access the page and the
_ServerAlias_ is the alternative if the _ServerName_ isn't accessible. The
following two lines are the paths to the access and error log of our virtual
host. If anything bad happens, those two files are the place to start
troubleshooting. The following lines are some directory options about what the
web app is allowed to do and what not. Next we need a new shell script, which
we call _mysite.sh_. Everything required to setup your project(composer update,
migrations etc.) will be defined in this script. In this tutorial we won't do
all this, but only activating our virtual host address.

``` bash
if [ -f /vagrant/tmp/mysite.conf ]; then
    chown $USER:$USER /var/www/html
    mv /vagrant/tmp/mysite.conf /etc/apache2/sites-available/mysite.conf
    a2ensite mysite.conf
else
    >&2 echo "Error: mysite.conf not found"
fi
```

# Make changes effective

```
service apache2 restart
```

Most of this should be familiar to you from the previous chapters. First we
check if the file exists in our temp directory. If it does, we create the
directory for our project and set the owner of this directory. Afterwards we
move the file to the _sites-available_ directory of apache and enable it. To
make sure all settings are active, we restart the apache demon. As you already
know, we need to get the _mysite.conf_ into the _tmp_ directory. That's why we
add another file provisioner to our _Vagrantfile._

``` ruby
config.vm.provision "file", source: "conf/mysite.conf", destination: "/vagrant/tmp/mysite.conf"
```

To activate the new provision you either need to execute

```
vagrant reload --provision
```

Or _vagrant destroy_ and _vagrant up_ again. The last step is to make the
virtual host known to your host system. For this, you need to add a line into
your hosts file. On unix like systems you can find the file in the _/etc/_
directory. The file should begin with the following

``` ini
#
# /etc/hosts: static lookup table for host names
#

#<ip-address>	<hostname.domain.org>	<hostname>
```

After this intro we add the following line

``` ini
192.168.33.10 mysite.dev
```

Which maps the IP address of our private network hosted by vagrant to a domain
name. Ok let's check it out. Type in _mysite.dev_ into the address bar of your
browser. You should now see the php info page from the last tutorial. In the
next post I want to cover the database access. I will show two ways to you, how
you can interact with your database from your host system. The first way will
be with MySQL Workbench via an SSH connection. The second is by installing
PHPMyAdmin on the guest system and access it via the hosts web browser.  [[Creating your own PHP dev-env in Vagrant Bonus 2|Bonus 2]]
