---
layout: post
title: "Primeiras impressões com Ruby e Rails"
description: ""
category: ruby
tags: [ruby, programação]
---
{% include JB/setup %}

Depois de bastante tempo pensando, finalmente decidi me aventurar no Ruby e Rails.
Sou meio que originalmente programador Python, e hoje também trabalho PHP, tanto
o ambiente Ruby/Rails quanto a linguagem, até uma semana atrás, eram completamente
desconhecidos por mim. Aprender a linguagem em si está sendo bem tranquilo, mas
esbarrei em alguns problemas de ambiente, sobretudo quando tentei fazer o passo a
passo do site do Ruby on Rails.

Primeiro porque o Rails - versão  2.3.1, atual na data em que escrevo esse post -
requer o Ruby versão superior a 2, que não está disponível nos repositórios do
Ubuntu 14.04. Como estava com MUITA preguiça de compilar o Ruby, busquei por repositórios
que pudessem fornecer esse pacote, e encontrei o [BrightBox](https://www.brightbox.com/blog/2015/01/05/ruby-2-2-0-packages-for-ubuntu).

Depois de seguir o passo a passo e instalar o Ruby 2.2, segui o tutorial de 20
minutos do site do Ruby e fui instalar o Rails. Quando executei `gem install rails`,
o erro a seguir foi exibido:

```
ERROR:  Error installing rails: 	
ERROR: Failed to build gem native extension.
```

Pô Gabriel, que fase! :)

Enfim descobri que uma das dependências do Rails requer os cabeçalhos do Ruby.
Se você é preguiçoso como eu e instalou o Ruby através do repositório que
mencionei anteriormente, o seguinte pacote deve estar disponível no seu sistema:

```
apt-get install ruby2.2-dev
```

Quando executei o `bundle install`, para instalar as dependências do projeto,
o erro a seguir foi exibido:

```
Make sure that `gem install sqlite3 -v '1.3.11'` succeeds before bundling.
```

Depois de muita procura, finalmente encontrei a solução - os cabeçalhos do SQLite3
precisam ser instalados:

```
sudo apt-get install libsqlite3-dev
```

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
