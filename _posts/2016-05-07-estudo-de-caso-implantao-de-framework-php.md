---
layout: post
title: "Estudo de caso: Implantação de framework PHP"
description: ""
category: php
tags: [php, laravel, arquitetura]
---
{% include JB/setup %}

# Contextualização

Na empresa em que trabalho usamos um framework MVC de PHP proprietário, desenvolvido
internamente. Como não contamos com uma equipe dedicada à evolução desse framework,
cada membro faz o que pode para conciliar suas demandas com melhorias no framework.

Chegou a um ponto em que esse framework ficou obsoleto, engessado, de difícil
manutenção e demandava muita escrita de código para desenvolvermos as
funcionalidades nos sistemas. Quando viabilizámos o desenvolvimento de algumas
features essenciais, presentes em todos os frameworks de mercado, tais como
proteção contra XSS, CSRF, SQL Injection, utilização de um ORM, ferramenta de
testes, dentre outras, decidimos, ao invés de desenvolver essas features,
viabilizarmos a implantação de um framework.

Até então, uma solução interna havia sido desenvolvida devido às necessidades,
sobretudo de integrações com os sistemas internos. Para implantar um framework,
na época em que a estrutura atual foi desenvolvida - meados de 2011 - os mecanismos
de autenticação e autorização desse framework, dentre outros, deveriam ser
customizados. Como na época a equipe era bastante reduzida, não tinha experiência
com os frameworks disponíveis, tampouco tempo para aprender um, optou-se em
criar uma estrutura bastante espartana, capaz de executar um método de uma classe
controladora, com base nos parâmetros recebidos pela solicitação.

Nesse estudo de caso abordarei o processo de escolha do framework e alguns dos
desafios da equipe envolvida na empreitada, na qual eu fiz parte.

# O processo de escolha

Pesquisamos os frameworks PHP mais populares em 2015 e chegamos aos resultados
do [SitePoint](http://www.sitepoint.com/best-php-framework-2015-sitepoint-survey-results/),
apontando o Laravel como o mais popular, ficando na frente do segundo colocado -
Symfony - com uma vantagem considerável.

Os critérios adotados para escolha do framework - Laravel ou Symfony (ou ainda
suas versões micro - Lumen e Silex), foram:

- Tamanho da comunidade
- Quantidade de bibliotecas disponíveis
- Atividade do projeto
- Funcionalidades nativas
- Quantidade de material disponível
- Curva de aprendizado

O Laravel, sobretudo devido à simplicidade, baixa curva de aprendizado e
sua documentação, nos pareceu mais atrativo que o Symfony. Este,
parece requerer mais conhecimentos de padrões de design e ter uma curva de
aprendizado mais alta que o Laravel. O Symfony, por padrão usa o Doctrine -
ORM que adota padrão Data Mapper, ideal para projetos maiores e mais complexos,
que vai além das necessidades da empresa em que trabalho. O Laravel, por sua vez, usa o Eloquent,
que adota o padrão Active Record, mais simples e que permite uma maior
produtividade no desenvolvimento. O mecanismo  de validação do Laravel pareceu
bem mais fácil do que o do Symfony, permitindo a combinação de validadores
com pipe, ao estilo Unix.

Como adotaríamos o Angular em conjunto com o Laravel, utilizando REST para
eles conversarem, o mecanismo de template não influenciou na decisão, apesar
de eu ter gostado BEM mais do Twig do que do Blade (opinião sugestionada, por
ele ter sido inspirado no mecanismo de template do Django, que eu curto bastante).

Com as decisões tomadas - Laravel com Angular - tínhamos a arquitetura
proposta, nos faltava elaborar um projeto piloto com os casos de uso mais
comumente encontrados nos projetos desenvolvidos pela empresa, de modo a
viabilizar a arquitetura proposta.

# Elaboração da arquitetura executável

Estudando o Laravel verifiquei que ele não suporta bancos de dados Oracle
nativamente, mas havia a biblioteca [laravel-oci8](https://github.com/yajra/laravel-oci8),
que adicionava essa funcionalidade - restava saber se ela estava madura e estável
o suficiente para suportar um projeto no ambiente de produção. Na empresa em que
trabalho há uma nomenclatura própria, que faz uso de prefixos nos objetos de
banco de dados, de modo que seria imprescindível que essa biblioteca suportasse
esse padrão.

O Laravel possui um padrão na nomenclatura de tabelas (snake case - sendo a
tabela com o mesmo nome do model no plural) e campos, como o identificador da
tabela (id) e os campos de timestamp (created_at e modified_at), mas permite
sobrescrever esse comportamento padrão. O laravel-oci8 também tem seu próprio
padrão de nome de sequences (o Oracle usa sequences para permitir auto-incremento)
e dizia também permitir a customização desse nome.

Selecionamos um conjunto de casos de uso, partindo do mais simples para ganhar
experiência com o framework. No primeiro selecionado, deveríamos obter registros
com base no recebimento ou omissão de parâmetros de busca, inserir, atualizar e
excluir registros de uma tabela auxiliar.

A obtenção de registros foi tranquila, já a inserção foi desafiadora. Seguindo
a documentação do Eloquent, tudo parecia estar certo, mas o ID não estava sendo
incrementado e vinculado no insert. Segundo a documentação do laravel-oci8,
estava certo também, até que enfim abri uma issue no Github do laravel-oci8.
Prontamente o mantenedor do projeto respondeu que o insert com sequence customizada
não estava implementado ainda e me passou uma alternativa. Dias depois a equipe
implementa essa funcionalidade e me alerta na issue. A alteração e exclusão - física
e lógica - de registros foram tranquilas.

Utilizamos um RESTful resource controller para prover os serviços que o Angular
consumiria, com algumas ressalvas. O método `index`, que parecia idealizado na documentação do Laravel para
renderizar um template com a listagem de objetos, foi utilizado para retornar os registros em JSON. O método `show`, ao invés de renderizar
um template com a exibição do objeto, dado um ID, foi utilizado para retornar um
objeto serializado. O método `edit` não foi utilizado.

Um dos maiores desafios a serem enfrentados na implantação seria a integração com
o sistema de gestão de identidades. Nele, cada usuário do sistema possui um usuário
no Oracle. Os sistemas devem executar as consultas ao banco com as credenciais do
usuário do Oracle e, para tal, deveríamos desenvolver nosso próprio mecanismo de
autenticação.

O Laravel, como todo bom framework atual, permite a extensão de suas funcionalidades
com certa facilidade. A criação de um User Provider, como o Laravel denomina,
foi bastante tranquila, mesmo para alguém sem experiência no framework.
Com o mecanismo elaborado, passamos a implementar a autenticação via JWT
(JavaScript Web Token) - padrão stateless que utiliza um token na requisição para manter
o usuário autenticado, usado sobretudo em APIs. Como trabalharíamos com Angular e
faríamos as integrações entre os sistemas via REST, optamos por esse padrão.

Implementamos outros casos de uso mais complexos, fazendo uso de transações,
models utilizando tabelas em outros esquemas e mecanismos de bancos de dados -
MySQL - que usamos na empresa em conjunto com o Oracle, todos bem sucedidos.

# Considerações finais

Finalizamos a arquitetura executável no prazo estipulado, tendo elaborado
classes que suprem as necessidades específicas dos projetos da empresa - tais como
o mecanismo de autenticação - que serão incorporadas em um projeto separado.
Esse projeto seria incluído como dependência de todos os projetos utilizando
essa arquitetura.

Apesar de termos elaborado essa arquitetura executável e, consequentemente
minimizado o risco da implantação do framework, sempre há o risco de nos depararmos
com um problema, mas esse é, ao meu ver, mínimo e que vale a pena enfrentar.
As necessidades da empresa em que trabalho não são tão específicas que ninguém
já tenha feito algo parecido e, talvez, disponibilizado o código.
Além disso, o Laravel foi desenvolvido seguindo bons princípios de engenharia de
software, de modo que permite a modificação e customização de seus comportamentos,
como pudemos comprovar na elaboração do nosso mecanismo de autenticação.

Esperamos que, com a implantação do framework, ganhemos em produtividade e
manutenibilidade, além de nossos sistemas se tornarem mais confiáveis e
seguros. Editarei esse post quando tivermos desenvolvido uma aplicação usando
o framework, passando os pareceres a respeito do processo.
