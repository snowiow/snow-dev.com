---
title: "The power of Vim Plugins: Vundle"
tags:
  - vim
categories:
  - The Power of Vim Plugins
date: 2015-09-29
description: To start of this series properly, we need a plugin manager, to handle all our plugins and keep them up to date. Vundle is one common option, but there are some more to choose from. Two more big players in this business are NeoBundle and Pathogen.
---

To start of this series properly, we need a
plugin manager, to handle all our plugins and keep them up to date.
[Vundle](https://github.com/VundleVim/Vundle.vim) is one common option, but
there are some more to choose from. Two more big players in this business are
[NeoBundle](https://github.com/Shougo/neobundle.vim) and
[Pathogen](https://github.com/tpope/vim-pathogen). I didn't looked too deep
into the last two plugin managers. To be honest, I never tried out something
else than Vundle and I think the reason is that I never felt uncomfortable or
missed something. So I never felt in need of getting another plugin manager and
in this post I want to show you why. First of all you need to have git
installed. You can download git with the package manager of your distribution,
if it isn't installed already. You can easily check this by starting up your
console and type in the following: 

```
git --version 
```

If git version \<some number\> appears you are good to go, but if something
like unknown command pops up, you are in a lot of trouble... Not really. You
just need to install git with your package manager and everything will be fine.
For the most common Linux distributions, the command looks like one of these:


```
apt-get install git
```
```
yum install git
```
```
pacman -S git
```

When git is installed you download Vundle by executing the following command in
your console:

```
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

This command downloads the Vundle code into bundle in your personal vim
directory. The bundle directory will be the place where all your plugins are
saved. If you're done, you should keep git on your computer, because vundle is
working with git commands internally. If it doesn't find git, you aren't able
to use Vundle at all. Now we need to add some stuff to the .vimrc. The .vimrc
is a file, where all your configurations in relation to Vim are placed. If you
didn't edit anything there yet, have a look at your home directory (Make sure
to also show invisible files). If there is no such file, you can create it now
and it will be automatically loaded, when vim is starting the next time. Now we
need to initialize Vundle. To do so we place a code snippet at the beginning of
the .vimrc(It is very important to place the following snippet in front of your
existing configuration, if you already have written down some configurations).

``` viml
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" let Vundle manage Vundle, required

Plugin 'VundleVim/Vundle.vim'
"New plugins go here

call vundle#end()
```

To test if Vundle is working, we add a new plugin. As an example we install
NERDTree, which will be covered in the next post of this series. Under the
comment _"New plugins go here_ the plugin will be added. Because it is hosted
on GitHub we can write the following into the .vimrc

```
Plugin 'scrooloose/nerdtree'
```

To take this change into effect, you either need to restart Vim or re-source
your .vimrc. I've got the following line saved into my .vimrc, which
automatically re-sources my .vimrc everytime it is saved.

```
autocmd! bufwritepost .vimrc source %
```

After restarting/resourcing the .vimrc you enter the following command

```
:PluginInstall
```

A new split view is opened, which shows your plugins and either a dot or a star
in front. The dot says that nothing was done, because the plugin is already
installed and the star says that the new plugin was successfully installed. If
anything goes wrong, you get the possibility to open up the log and look up the
error. To remove a plugin, you need to remove the corresponding line out of
your .vimrc. For example if we want to get rid of the NERDTree Plugin we remove
the line, which we just added and save the .vimrc (Don't forget to
re-source/restart). Afterwards type in the following command into Vim:

```
:PluginClean
```

A new split view opens with the plugins which will be removed. You can accept
it by typing _y_ or abort by typing _n. _ The final command I use on a regular
basis is to update my existing plugins.

```
:PluginUpdate
```

Again a new split view opens and an arrow is moving through your installed
plugins and marks them with a dot or a star, similar to the _:PluginInstall_
command. The symbols also have the same meaning. Dots say that nothing needed
to be done and the star tells you if a update was available and installed. If
you want to search for new plugins you can use the _:PluginSearch_ command like
this:

```
:PluginSearch nerdtree
```

But I often got problems doing it this way. So the normal way I search for new
plugins is using my search engine of trust. I also prefer downloading every
plugin from Github to keep this clean overview of _Plugin
'creator/plugin_name'_ in my .vimrc. But maybe a plugin isn't available on
GitHub(which was never the case for me), the second best source would be [Vim
Scripts](http://vim-scripts.org). To add a Vim Scripts link, you just need to
add the name of the plugin, which is even cleaner:

```
Plugin 'nerdtree'
```

Another option is to include a plugin from a non GitHub Hoster, who also uses
git, like Bitbucket. To add a plugin you need to start writing _git: _followed
by the complete URL of the plugin. As an imaginary example, if NERDTree would
be hosted on Bitbucket, it would look like this:

```
Plugin 'git:bitbucket.com/scrooloose/nerdtree'
```

I hope I could help with this brief overview of Vundle. If you need further
information either head to the GitHub page of the [Vundle
project](https://github.com/VundleVim/Vundle.vim) or type the following command
into Vim:

```
:h vundle
```
