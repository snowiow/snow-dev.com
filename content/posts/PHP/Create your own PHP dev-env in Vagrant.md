---
title: Creating your own PHP dev-env in Vagrant
tags:
  - dev-env
  - php
  - vagrant
categories:
  - PHP
  - Vagrant
date: 2016-06-03
description: In this series of posts we will be creating a development environment(dev-env) for PHP with the help of vagrant. Vagrant is a thin wrapper around different virtualization software projects like virtual box, which we will be using in this tutorial. The so called wrapper of vagrant is a configuration file, written in the ruby programming language. But don't worry, you don't need to be an expert ruby programmer to setup a dev-env in vagrant. Everything we use it for, are some variable assignments. In this configuration file we can tell the virtualization software, which operating system and software to install and how to configure everything. Our ultimate goal here is, if the environment is started, everything is setup already. Finally we want to get a fully configured LAMP stack with a running apache webserver, a mysql database and PHP. As always there are many different ways to glory. For example does vagrant offer different ready to use recipes via chef. There is also support for Puppet, a unified configuration language for different systems. We won't use any of these plugins in this series. Everything we will work with, are some bash scripts and the vagrant file. I have chosen this path, because I want to keep the full control over everything. On the other hand it's also more work, but I think it's worth it. Before we go into the details, lets talk about the advantages of a vagrant based development environments. 
---

# Table of Contents
* [[Creating your own PHP dev-env in Vagrant Part 2|Part 2]]
* [[Creating your own PHP dev-env in Vagrant Part 3|Part 3]]
* [[Creating your own PHP dev-env in Vagrant Bonus 1|Bonus 1]]
* [Bonus 2](/posts/creating-your-own-php-dev-env-in-vagrant-bonus-2.html)
* [Full Code Base on GitHub](https://github.com/snowiow/vagrant-template)

In this series of posts we will be creating a development environment(dev-env)
for PHP with the help of [vagrant](https://www.vagrantup.com). Vagrant is a
thin wrapper around different [virtualization
software](https://en.wikipedia.org/wiki/Virtualization) projects like [virtual
box](https://www.virtualbox.org), which we will be using in this tutorial. The
so called wrapper of vagrant is a configuration file, written in the [ruby
programming language](http://www.ruby-lang.org/en/). But don't worry, you don't
need to be an expert ruby programmer to setup a dev-env in vagrant. Everything
we use it for, are some variable assignments. In this configuration file we can
tell the virtualization software, which operating system and software to
install and how to configure everything. Our ultimate goal here is, if the
environment is started, everything is setup already. Finally we want to get a
fully configured LAMP stack with a running apache webserver, a mysql database
and PHP. As always there are many different ways to glory. For example does
vagrant offer different ready to use recipes via
[chef](https://github.com/ShawnMcCool/vagrant-chef). There is also support for
[Puppet](https://www.vagrantup.com/docs/provisioning/puppet_apply.html), a
unified configuration language for different systems. We won't use any of these
plugins in this series. Everything we will work with, are some bash scripts and
the vagrant file. I have chosen this path, because I want to keep the full
control over everything. On the other hand it's also more work, but I think
it's worth it. Before we go into the details, lets talk about the advantages of
a vagrant based development environments. 

# Benefits
The most obvious benefit is that we will have one file, which handles the
complete configuration. As soon as it is written, we can use it over and over
again. We can upload the file to git and download it on other machines to get
the same dev-env everywhere we work. We can share it with other programmers in
a team, so everyone is working in the same environment. Sentences like "...but
it works on my machine" lie in the past now. Best case scenario would be to
replicate the environment of the production system. In this case, you are able
to catch any environmental issues locally, before they go live. The second
benefit is a PHP specific one. Languages like python have built-in features to
run different versions of the language for different projects and also manage
their respective
packages([virtalenv](https://pypi.python.org/pypi/virtualenv)). In PHP it's
pretty difficult to comfortably run different versions on one system. There are
however some approaches like [phpbrew](https://github.com/phpbrew/phpbrew) to
handle this problem. Vagrant is another option. Here you just start the vagrant
machine with the respective version of PHP installed. The final benefit is that
you interact with everything inside of vagrant from your physical machine. You
can use your preferred IDE to code your app. You can reach the webserver via
your normal webbrowser. You can view the database over a ssh connection and so
on. In this domain your physical machine is called the host machine and the
operating system inside vagrant is the guest. So lets list up our final goals,
we want to archieve at the end of this tutorial. 

# Goals
* Having a one line command to fire up the whole dev-env(vagrant up)
* Access the served sites via your preferred web browser on your host system
* Setup Virtual hosts to access the site via a pretty looking URL
* Access the database over a ssh connection with workbench or PHPMyAdmin on your hosts web browser
* Having a shared fdirectory between host and guest system, to be able to code on your host system and let the changes take effect immediately

# Installation
You need to install two things to get started. The first thing is vagrant of
course. You can download it from their [official
homepage](https://www.vagrantup.com/downloads.html). I personally installed
vagrant with my package manager on arch linux. This isn't recommended by
hashicorp, but I never had problems so far. The second thing you will need is
VirtualBox, which will be our virtualization software of choice. Again you can
either download it from [their official
website](https://www.virtualbox.org/wiki/Downloads) or install it via your
package manager. Those are all the things you need to get yourself into the
getting started section. 

# Getting started 
We want to create a vagrant subdirectory in a project directory. Our goal in this tutorial is to have vagrant as a subdirectory of the project folder, where our PHP web app lives. So we can upload anything to git together. Our "project" will just be a phpinfo file, but it can be substituted by any project, you are currently working on.

```shell
cd path-to-project && mkdir vagrant
```

Afterwards we switch into that directory and initiate a new vagrant project.

```
cd vagrant && vagrant init
```

This will create a file called _Vagrantfile_ in your newly created directory.
This is the configuration we talked about in the beginning of this post. If you
open it, you see some ruby code, which consists of some assignments and many
comments. We won't cover anything in detail here, because we want to focus on
our mission and get our goals done. If you want to know more about the other
stuff in the _Vagrantfile_, you can head to the [official
documentation](https://www.vagrantup.com/docs/getting-started/) of vagrant,
which is very well written. So let's get into the first two lines of code.

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "base"
```

The first line stays as it is. It tells vagrant on which version this config is
based. The second line says what operating system image should be used. These
are called boxes in the vagrant domain. We will use a basic ubuntu 14.04
installation as our basis. That's why we change the second line to this:

```ruby
config.vm.box = "ubuntu/trusty64"
```

For a full list of available boxes have a look into
[this](https://atlas.hashicorp.com/boxes/search). Here you can find all
different kinds of boxes. Starting off with very basic ones, up to fully
configured environments. The next line we are interested in is the following:

```ruby
# config.vm.synced_folder "../data", "/vagrant_data"
```

This line tells vagrant which directory of the host system should be mapped to
a directory inside the guest system. These are called synced folders, because
any operation happening in this directory is instantly visible on both systems.
As you may noticed, this will be the location, where our code base will be
placed. For the moment, we will edit this line of code to the following:

```ruby
config.vm.synced_folder "../", "/vagrant/projects/"
```

The first argument is the directory on the host system. The second argument is
the path inside the guest system. It will be created automatically on startup
of the vagrant machine. So we don't need to do anything there for the
moment. The last thing we uncomment for the moment is this one

```ruby
# config.vm.network "private_network", ip: "192.168.33.10"
```

We don't change anything except for deleting the hash tag at the beginning. As
soon as this line is active, vagrant starts up a private network connection,
which is only accessible by the host system. This will also be the IP address
where our web application will be available later on. You can change the IP to
anything you want, but remember to change the later occurrences, where we work
with this particular IP as well. This is the end of our intro. In the following
chapter we will write our first shell scripts to setup the apache webserver.  
[[Creating your own PHP dev-env in Vagrant Part 2|Goto Part 2]]
