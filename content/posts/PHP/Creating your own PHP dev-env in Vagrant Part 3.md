---
title: 'Creating your own PHP dev-env in Vagrant: Part 3'
tags:
  - mysql
  - php
  - vagrant
categories:
  - PHP
  - Vagrant
date: 2016-06-03
description: In the previous post we've set up the apache web server successfully. Now it's time to add MySQL and PHP to finish the LAMP stack.
---
# Table of Contents
* [[Create your own PHP dev-env in Vagrant|Part 1]]
* [[Creating your own PHP dev-env in Vagrant Part 2|Part 2]]
* [[Creating your own PHP dev-env in Vagrant Bonus 1|Bonus 1]]
* [[Creating your own PHP dev-env in Vagrant Bonus 2|Bonus 2]]
* [Full Code Base on GitHub](https://github.com/snowiow/vagrant-template)

In the previous post we've set up the apache web server successfully. Now it's
time to add MySQL and PHP to finish the LAMP stack. Let's start with MySQL
first and add a _mysql.sh_ file to the __sh__ directory. We write the following
lines into that file.

``` bash
debconf-set-selections <<< "mysql-server mysql-server/root_password password my_pw"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password my_pw"
apt-get install -y mysql-server libapache2-mod-auth-mysql
```

The first two lines set values for the upcoming installation dialog of MySQL.
The installation dialog will ask us for a password and to repeat it. Because we
want to setup MySQL automatically without any interaction, we preset those
fields with our password of choice. In this case __my_pw__. Afterwards we
install MySQL and an extra package for apache. Afterwards we copy over the
config file of MySQL onto the guest system.

``` bash
if [ -f /vagrant/tmp/my.cnf ]; then
    mv /vagrant/tmp/my.cnf /etc/mysql/my.cnf
else
    >&2 echo "Error: my.cnf not found"
fi
```

We do it exactly the same way, as we did with the _apache.conf_. First we place
the _my.cnf_ in our _conf_ directory. Afterwards we add a file provisioner to
our _Vagrantfile_.

``` ruby
config.vm.provision "file", source: "conf/my.cnf", destination: "/vagrant/tmp/my.cnf"
```

This time it's important to replace a custom config file with the original,
because in our modified version we switch the following line from:

```
bind-address = 127.0.0.1
```

to

```
bind-address = 0.0.0.0
```

Before it was only possible to access the database on the guest machine itself.
We changed this, so we can connect from the host machine as well. Finally we
need to restart the MySQL demon.

``` bash
service mysql restart
```

With which we end up in our final _mysql.sh_ file, which looks like this:

``` bash
#!/usr/bin/env bash

debconf-set-selections <<< "mysql-server mysql-server/root_password password my_pw"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password my_pw"
apt-get install -y mysql-server libapache2-mod-auth-mysql php5-mysql

if [ -f /vagrant/tmp/my.cnf ]; then
    mv /vagrant/tmp/my.cnf /etc/mysql/my.cnf
else
    >&2 echo "Error: my.cnf not found"
fi

service mysql restart
```

Final thing we need to do is adding the shell script as a shell provisioner to our _Vagrantfile_.

``` ruby
config.vm.provision "shell", path: "sh/mysql.sh"
```

Now let's install PHP as well. This is pretty straight forward. Our _php.sh_ looks like this:

``` bash
#!/usr/bin/env bash
apt-get install -y php5 libapache2-mod-php5 php5-mcrypt php5-curl php5-mysql
```

Afterwards we also add it as a provisioner

``` ruby
config.vm.provision "shell", path: "sh/php.sh"
```

Which results in the following _Vagrantfile_

``` ruby
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provision "shell", path: "sh/update.sh"
  config.vm.provision "shell", path: "sh/apache.sh"
  config.vm.provision "shell", path: "sh/mysql.sh"
  config.vm.provision "shell", path: "sh/php.sh"

  config.vm.provision "file", source: "conf/apache2.conf", destination: "/vagrant/tmp/apache2.conf"
  config.vm.provision "file", source: "conf/my.cnf", destination: "/vagrant/tmp/my.cnf"

  config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.synced_folder "../", "/vagrant/projects/"
end
```

This is it. Our LAMP stack is complete. Now we want to check out our
environment. Switch into the directory of the _Vagrantfile_ and execute the
following command

```
vagrant up
```

A lot of stuff is happening now and for the first time it will take some time
to fire up the vagrant machine. If vagrant booted up successfully, we add
an _index.php_ in the directory above our _Vagrantfile_. This is our project
directory. I won't show anything big here, just a proof of concept 

``` php
<?php

phpinfo();
```

If you browse to __192.168.33.10__ on your host system, you should see an info
page about the running PHP version, similar to this 

![[phpinfo.png]]

Congratulations! You've done it! To suspend your vagrant machine just type the
following in the same spot, where you did the _vagrant up_

```
vagrant suspend
```

This saves the state of your machine and will be the way to shut it down in
daily business. If you want to completely reset your machine and install
everything again, you execute

```
vagrant destroy
```

To start your vagrant machine, just do _vagrant up_ again. You could start from
here and put your own PHP project in the parent directory of the _Vagrantfile_
and start programming. But if you want to know some nifty stuff, like how to
access your website via an address like _mysite.dev_ instead of 192.168.33.10
or how you can access your database with the __mysql-workbench__ software on
your host machine, you should definitely check out the next posts in this
series. [[Creating your own PHP dev-env in Vagrant Bonus 1|Goto Bonus 1]]
