---
title: 'The power of Vim Plugins: Vim-Plug'
categories:
  - The Power of Vim Plugins
date: 2016-04-03
description: A few weeks after my Vundle post, I stumbled upon a plugin manager called vim-plug. It sounded very promising and I checked it out. Until today I didn't go back to Vundle. This is almost half a year ago and I think it's time to write something up about this amazing plugin manager.
---

A few weeks after my
[Vundle](http://snow-dev.com/the-power-of-vim-plugins-vundle/) post, I stumbled
upon a plugin manager called [vim-plug](https://github.com/junegunn/vim-plug).
It sounded very promising and I checked it out. Until today I didn't go back to
Vundle. This is almost half a year ago and I think it's time to write something
up about this amazing plugin manager. When you enter the GitHub page you see a
bunch of pros, why this plugin manager is so awesome. Two points stood out for
me though. Point one is the utilization of asynchronous downloads and
installation, which makes the whole process of installing and updating a whole
lot faster. If you installed your vim with the +python/+python3 or +ruby flag,
you are able to use this feature. The alternative is to install
[neovim](https://neovim.io/), which comes with asynchronous batteries included.
The second point is "On-demand loading for faster startup time". This line
says, you are able to give every plugin some options, when they should be
loaded up. For example, if you program in more than one language, say PHP and
C++. Now you got some language specific plugins for example
[clang-format](https://github.com/rhysd/vim-clang-format), which formats your
C++ Code. Having the plugin loaded up, only makes sense if you are in a C++
file, right? With the help of vim-plug you are now able to delay the startup of
clang-format until you open a C++ file for the first time. This isn't the only
lazy load option. We will cover this feature in depth in a later section. As I
looked through my plugin list I got about one half of my plugins which didn't
need to be loaded on startup. After setting these options, my startup time
increased quite a bit. If you are convinced either by me or because of the rest
of pro list, let's get started with the installation.  

# Installation 
The installation is pretty straight forward. The GitHub page
gives you 3 different commands. If you use normal vim on Unix, you need to
execute the following line:

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs
```

If you use neovim and you don't have your default vim location symlinked, you
need to change the destination path:

```bash
curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs
```

Now that you have vim-plug installed you need a section at the beginning of
your _.vimrc_, which looks like this: 
```vim
call plug#begin('~/.vim/plugged')

call plug#end()
```

With this template you are ready to go for the next steps.  

# Configuration
Adding plugins from GitHub is as easy as the following line.
You need to place these between the 2 lines, which we added at the end of the
last section.

```
Plug 'scrooloose/nerdtree'
```

If you are using another source than GitHub, you can put the full git path
between the quotes. Pay attention to the ending of the URL:

```
Plug 'https://github.com/scrooloose/nerdtree.git'
```

Thats it! Now you can install that Plugin via the _:PlugInstall_ command.
Another tiny benefit is, that you save 2 characters in each command. You just
type _:PlugInstall_ for vim-plug instead of _:PluginInstall_ in Vundle. The
same goes for updating existing packages, which is _:PlugUpdate_ instead of
_:PluginUpdate_. This is only a small thing, but I really like this, since I've
switched. The second important feature I use very often is to remove packages.
For this you just remove the line

```
Plug 'scrooloose/nerdtree'
```

Afterwards you call _:PlugClean_. Now you get a prompt with the plugins, which
should get deleted. You can accept with \[y\]es or \[n\]o. To skip this prompt,
you can also type in the command _:PlugClean!_. Now we are coming to one of the
main advantages: The on-demand loading. As described in the beginning, this
delays the loading of the plugin until something specific happens. This is
great because vim doesn't load up all plugins on startup, which improves your
startup time quite a bit. Let's get to our first example, where I want to show
you the __on__ keyword. The __on__ keyword loads a plugin, when a specific
command was executed. Those options will be written as JSON after the plugin
path. For example, we want to load NERDTree when _:NERDTreeToggle_ was
executed for the first time.

```
Plug 'scrooloose/nerdtree', {'on': 'NERDTreeToggle'}
```

As you can see, we split up the line by a comma. Afterwards we write down a
JSON string, where __on__ is the key and the command(without the colon) is the
value. That's everything! Now NERDTree load will be delayed until this command
is executed. The second keyword I use pretty often is __for__. This keyword
loads plugins for specific filetypes. Let's take the C++ plugin clang-format
from the intro, which formats C++ code according to some sort of standard. This
plugin is only useful to us, if we work on C++ files. That's why we want to
delay the loading of this plugin until we work on a C++ file for the first time

```
Plug 'rhysd/vim-clang-format', {'for': 'cpp'}
```

And here we go. Clang-fromat will be lazy loaded when we open a C++ file for
the first time. The last keyword which is pretty useful is __do__. It's used
for plugins, which need another third party component to be compiled. One
example is [YouCompleteMe](https://github.com/Valloric/YouCompleteMe), an
autocompletion engine for vim. If you want to have C++ semantic completion
support, you need to compile another component, which is used by this plugin.
The compilation is triggered via a python script, which downloads clang and
compiles the component afterwards. Normally you do this manually after each
update, but you are also able to tell vim-plug to execute it after the plugin
update automatically. To make it happen add the following line to your
_.vimrc_.

```
Plug 'Valloric/YouCompleteMe', { 'do': './install.py' }
```

Now everytime YouCompleteMe is updated, the third party component will be
recompiled automatically and you don't have to deal with it anymore. Of course
you can combine the keywords in a single JSON document.

```
Plug 'Valloric/YouCompleteMe', { 'do': './install.py', 'for': 'cpp' }
```

Those are the things I mainly use, but there is much more to discover. For a
full reference head over to the [GitHub page of
vim-plug](https://github.com/junegunn/vim-plug).
