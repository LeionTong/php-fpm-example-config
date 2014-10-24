# Example configuration of php-fpm

## Introduction 

This is an example configuration [`php-fpm`](http://www.php.net/manual/en/install.fpm.php)
which is the preferred way of serving PHP with FastCGI. The old
`php-cgi` is deprecated. You can even use a
[patch](http://php-fpm.org) for PHP versions prior to 5.3.3, when fpm
become an official PHP project.


## Generalities

The way that php-fpm operates is through a **master** process that
launches children processes (workers) that serve the requests. These
workers can be run under a particular user. Making it appropriate for
environments where there are several users each one running their own
set of PHP apps.

php-fpm is pretty much a **work in progress**, although on one hand is
IMHO the best thing to come out of PHP land in a lot of time, still
lacks a lot of nice features like **graceful restarts**. Currently
adding a new pool or reloading the config requires a **full
restart**. Which causes some downtime. Work is in progress to provide
graceful restarts/reloads like
[Nginx](http://wiki.nginx.org/NginxCommandLine) has.

It has the capacity to adjust the number of workers **dynamically** to
the load, varying from a minimum to a specified maximum.

## Configuration layout

The configuration comes in two flavors:

 1. [unix](https://github.com/perusio/php-fpm-example-config) which is
    the **default**. It uses **UNIX domain sockets** for communication
    between the FCGI responder provided by php-fpm and the server or
    request frontend.
  
 2. [tcp](https://github.com/perusio/php-fpm-example-config/tree/tcp).
    It uses **TCP sockets** for communication between the FCGI
    responder provided by php-fpm and the server or request frontend.

 Choose the one that is more appropriate for your setup. Up until PHP
 5.3.8 TCP sockets, although, theoretically slower, if your setup is
 on the loopback, behave better than UNIX sockets for high-traffic
 sites.
 
 This might have changed in 5.3.10 and 5.4.x. Try it and report back
 please.
 
## Load adequation 

There's no algorithm for determining the number of children. It
depends on your application.

A
[thread](http://groups.google.com/group/highload-php-en/browse_thread/thread/754dbedc5eb841a2)
in the
[highload-php-en](http://groups.google.com/group/highload-php-en)
gives some tips on how to determine the number of children.

 1. If your load is **CPU bound** then the rule is that the number of
    children should be **equal** to number of **CPUs** plus 20 %.
    
    Example: Machine with 8 CPUs. Number of children = 10.
    
 2. If your load is I/O bound then apply the following rule:
 
        number_of_children = 1.2 * total_memory / average_space_per_process 
  
    Example: PHP processes occupying 256 MB of average space in a
    machine with 2GB of RAM that can be used for running this PHP
    application.
  
        number_of_children = 1.2 * 2048 / 256 
  
    giving: `10` children. 
    
    Determine the medium space occupied by a PHP process and apply the
    above formula. The 1.2 factor is just a security factor, to use a
    much abused engineering term.
    
## Features 

 1. It uses `UNIX` sockets for connections from the web server to the
    FastCGI daemon as the *default*. It provides a branch `tcp` that
    relies on TCP sockets. Choose which suits you better.
    
 2. The `php.ini` is modified from the stock one that comes with the
    Debian package. The modifications were made by using a tiny script
    that I wrote for *cleaning up* a PHP config and that's available
    [here](https://github.com/perusio/php-ini-cleanup) on
    github.
 
 3. There are **three** pools on this config that run under the
    `www-data` user. The idea is that the server should load balance
    and distribute the load as you wish over it.
    
 4. Support for the **status** and **ping** functionalities of
    `php-fpm`. See
    [here](https://github.com/perusio/drupal-with-nginx) how to enable
    it for [Nginx](http://wiki.nginx.org).

## Installation

 1. Clone the git repo:
    `git://github.com/perusio/php-fpm-example-config`.

 2. Checkout the `tcp` branch if that's suits you better:
 
         git checkout -b tcp 
         
    If on the contrary the `unix` sockets approach is more suited to
    your site(s) then you can ignore this step and proceed to 3
    directly.

 3. Alter the `php-fpm.conf` and the `pool.d/www.conf` file to your
    liking. Add any pool that you might want.
    
 4. Copy the files to the destination directory:
 
     cp php5-fpm.conf /etc/php5/fpm
     
     cp -a pool.d /etc/php5/fpm
     
 5. (Re)start `php5-fpm` with `service php5-fpm restart` or `service
    php5-fpm start` if starting `php-fpm` anew.   

## Caveats

Remember to **always** do `service php5-fpm restart` after adding a
new pool or modifying the configuration of an existing one.
