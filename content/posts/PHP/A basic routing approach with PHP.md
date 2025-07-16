---
title: A basic Routing approach with PHP
tags:
  - mvc
  - php
  - route
  - routing
categories:
  - PHP
date: 2015-10-07
description: Nice and readable URLs are the way to go in modern web applications. More and more people are abandoning the old style URLs, containing question marks and  equal signs, in favor of the slash separation for actions and parameters. Most frameworks are already supporting this new kind of URLs and encapsulate the logic inside of a routing class or module. Everyone is using it and everything is good so far. But even if these new URLs are all over the place, there is very little information on the net about how it is actually implemented. Because I'm currently working on a minimalistic PHP MVC Framework with a friend, I came across this problem. Beside of the source code of the big players in PHP Frameworks I found a small and easy to use snippet to get routes working pretty fast. In this post I want to present the key PHP feature, which allows us to realize it and how to build upon it.
---
Nice and readable URLs are the way to go in modern web applications. More and
more people are abandoning the old style URLs, containing question marks and 
equal signs, in favor of the slash separation for actions and parameters.  Most
frameworks are already supporting this new kind of URLs and encapsulate the
logic inside of a routing class or module. Everyone is using it and everything
is good so far. But even if these new URLs are all over the place, there is
very little information on the net about how it is actually implemented.
Because I'm currently working on a minimalistic PHP MVC Framework with a
[friend](http://randy-schuett.de), I came across this problem. Beside of the
source code of the big players in PHP Frameworks I found a small and easy to
use snippet to get routes working pretty fast. In this post I want to present
the key PHP feature, which allows us to realize it and how to build upon it.
Everything starts of with the PHP super constant
[$_SERVER](http://php.net/manual/en/reserved.variables.server.php).  We are
interested about the 'REQUEST_URI'  key. If we type in the address of our
webserver, followed by and _/index.php_ and this file contains the following
line of code.

``` php
<?php
echo $_SERVER['REQUEST_URI'];
```

we probably would get back _/index.php_. This is exactly what we wanted to
archive, because now we are able to read what the user typed into the address
bar of his browser and now we can react to it. But with this approach comes a
problem. If we would build our Router upon this feature, we are forced to place
a file at each possible path, which we want to cover, containing the request of
the super constant and reroute to the appropriate controller and action. A good
way to solve this problem would be, that whatever the user types into the
address bar, it would open up the same file everytime, like our _index.php_.
Sadly there is no way to solve this in PHP. So this has to be done on the
webserver level itself. Most webservers (like Apache)  support configurations
written down to a _.htaccess_ file, which is placed in the root directory of
the webserver. We write the following content to the file.

``` php
RewriteEngine On
RewriteRule ^(.*) - [E=BASE:%1]
RewriteRule ^(.*)$ %{ENV:BASE}index.php [NC,L]
```

The first directive tells the webserver to turn on the rewrite engine.
Afterwards we give him the rules to link everything to the _index.php_ in the
root directory. Now we can type in _/test/dir/index_ for example and the
_index.php_ in the root directory is called, but with this output
instead: _/test/dir/index_. Based on this behavior we can build our Router. For
a minimalistic example we create a _Router.php_ file which contains the
following class.

``` php
<?php

class Router
{
    public function route($route)
    {
        echo $route;
    }
}

```

and we change our _index.php_.

``` php
<?php

require_once 'Router.php';

$r = new Router();
$r->route($_SERVER['REQUEST_URI']);
```

So far we gained nothing except a new layer of abstraction. But now we are able
to actually give our routing some behavior, similar to modern frameworks, which
are based on controllers and actions. As an example we take the following
path _/default/print_hello_. In theory this should instantiate a
_DefaultController_ class and call the _print_helloAction_ method. As a place
for our controllers we create a new directory called _controllers_ and add a
_DefaultController.php_ file.

``` php
<?php

class DefaultController
{
    public function indexAction()
    {
        echo 'Index';
    }

    public function print_helloAction()
    {
        echo 'Hello';
    }
}
```

The last part of this post will be about how our Router class can actually
instantiate the _DefaultController_ and call the method based on the given
route. One way to do it is to cut down the string at the slashes. Afterwards we
got the controller and action name which we are now able to instantiate and
call. The code for the route function could look like this.  

``` php
<?php

class Router
{
    public function route($route)
    {
        $parts = explode('/', $route);
        //Index 0 is empty because the routes
        //always start with a preceding /
        $controller = array_key_exists(1, $parts) ? $parts[1] : '';
        $action     = array_key_exists(2, $parts) ? $parts[2] : '';

        if (strlen($controller)) {
            //upper case the first letter of the controller name and
            //append the Controller string to it
            $controller = ucfirst($controller) . 'Controller';
            //Include the PHP file for the Controller
            require_once 'controllers/' . $controller . '.php';
            //instantiate the controller
            $object = new $controller;
            //call the action
            if (strlen($action)) {
                return call_user_func([$object, $action . 'Action']);
            }
            //call indexAction if no action is given
            return call_user_func([$object, 'indexAction']);
        }
    }
}
```

If we type in _/default/print_hello_ we get back _Hello_. If no action is
given, we automatically try to route to the index action. So _/default_ would
print
_Index_. [call_user_func](http://php.net/manual/en/function.call-user-func.php)
also supports parameters. So it is very easy to extend the given Router with
parameter handling. Also there should be more handling of special cases, like
if no controller is given in the URL. It's also simple to extend the given
model to use real view files for showing a page, instead of just doing an echo.
Nevertheless serves this basic concept  as a starting point and it should be
straightforward to build upon it.
