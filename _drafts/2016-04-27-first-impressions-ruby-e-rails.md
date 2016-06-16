---
layout: post
title: "First impressions with Ruby e Rails"
description: ""
category: ruby
tags: [ruby, rails, programming]
---
{% include JB/setup %}

After a lot of time thinking, I finally decided to play with Ruby and Rails.
I'm originally a Python programmer and, today, I work with PHP too... I've always
heard about Rails, but until a week ago, their syntaxes were completely unknown
to me. Learning Ruby, up to the moment, it's not being that difficult, but I
faced some environment issues when I was following the Ruby on Rails tutorial.

First, because Rails - version 4.2 - the current version when I write this article,
requires Ruby 2.\*, that's unavailable in the Ubuntu 14.04 repositories. How I
was too lazy to compile Ruby and looking forward to learn Rails, I looked for
repositories which provide this package for my Ubuntu, and I found the
[BrightBox](https://www.brightbox.com/blog/2015/01/05/ruby-2-2-0-packages-for-ubuntu).

After reading its step-by-step and installing Ruby 2.2, I followed the Ruby's 20 minutes
tutorial and tried to install Rails. When I executed `gem install rails`, the
error below was shown:

```
ERROR:  Error installing rails: 	
ERROR: Failed to build gem native extension.
```

It happened that one of the Rails dependencies requires the Ruby headers.
If you are lazy like me and installed Ruby through that repository I cited
above, the package `ruby2.2-dev` should be available to download:

```
apt-get install ruby2.2-dev
```

Then, when I execute  `bundle install`, to install the projects dependencies
the following error was shown:

```
Make sure that `gem install sqlite3 -v '1.3.11'` succeeds before bundling.
```

The solution is to install the SQLite3 headers:

```
sudo apt-get install libsqlite3-dev
```

Then I finally could install Rails and follow its first tutorial, in which
you have to create a page to manage articles and their comments. I develop
a few years using Django and I can say that it's very productive... but Rails
is magic... a lot of thing seems to happen behind the scenes - the model
importing, the automatic rendering

Aí consegui instalar o Rails e fui seguindo o tutorial de criar um artigo e
comentários. Tenho algum tempo de desenvolvimento em Django, que é bem produtivo,
mas Rails parece bem mais... O Rails é mais mágico, muita coisa acontece por trás
dos panos, a importação do Model, a renderização automática do template em um
método vazio e a atribuição das variáveis de instância ao mecanismo de template,
para mim, foram os mais notados.

Com pouco tempo, não deu pra saber o "custo" dessa mágica toda - se são só prós
ou se tem contras também. Para quem é acostumado a saber o fluxo da coisa toda,
saber que objetos são passados para o template, que template é renderizado e etc.,
o Rails pode assustar um pouco. Estou prosseguindo com a aplicação e sigo com
mais impressões acerca do framework.
