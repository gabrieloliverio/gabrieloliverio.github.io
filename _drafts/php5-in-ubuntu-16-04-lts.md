---
layout: post
category : linux
title: "Instalando PHP 5 no Ubuntu 16.04 LTS"
description: ""
tags : [ubuntu, linux, php]
---
{% include JB/setup %}

For reasons beyond my restrict comprehension, Canonical removed PHP 5 from
the Ubuntu 16.04, leaving only the PHP 7 packages in its repositories. In this post
I write how to downgrade to PHP 5.6.

** First of all, sudo, please :) **

```bash
sudo su
```

** Then, remove PHP 5 and 7 that are installed on your system **

```bash
apt-get purge `dpkg -l | grep php| awk '{print $2}' |tr "\n" " "`
```

** Add the following repository that provides PHP 5.[56] e and update **

```bash
add-apt-repository ppa:ondrej/php
apt-get update
```

This repository provides packages for PHP 5.5 and 5.6. Change the version in the
following commands, according to your needs.

PS. I just used this repository in my development environment and you should
think at least twice before using it in production environment. If PHP 5 is
needed for your application, as if it would break in PHP 7, you maybe should
consider not upgrading your server to 16.04...

** Install PHP 5.5 or 5.6 and all the extensions you need **

```bash
apt-get install build-essential libaio1 php5.6-dev php-pear php5.6-soap php5.6-sybase php5.6-gd php5.6-xdebug php5.6-xml
apt-get install libapache2-mod-php5.6
```

The package `php5.6-xml` is needed in order to Pear and Pecl be executed properly.
`libapache2-mod-php5.6` is obviously needed only if you use Apache :)

** Verify if your PHP version o 5.[56] **

```bash
php -v
```

** Enable the PHP 5.[56] for Apache **

```bash
a2enmod php5.6
```

If you execute `dpkg -l | grep php` or noticed in the `apt-get install` outputs,
will see that PHP 7 is installed again. Apparently it is a dependency of
`php-pear`, so, you will not get rid of it so early :)

That's it.
