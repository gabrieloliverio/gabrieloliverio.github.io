---
layout: post
title: "Case study: PHP framework implantation"
description: ""
category: php
tags: [php, laravel, architecture]
---
{% include JB/setup %}

# Background

At the company I work, we use a proprietary MVC PHP framework, developed internally,
by ourselves. As we haven't got a dedicated team to the evolution of this framework,
each member do all the possible to conciliate his demands with improvements on
this framework.

We reached a point that this technology became obsolete, difficult to maintain and
required a lot of code to implement features on the systems. When we studied to
implement some essential features, that all the modern frameworks available
nowadays have got out of the box, such as XSS, CSRF and SQL Injection protection,
error handling and logging, testing helpers, among others, we decided
to study a framework implantation, instead of implementing these required features.

An intern solution was developed due to some specific requirements, above all,
of legacy systems integrations and some rules defined about authentication. To
implant a framework, at the time the actual framework was developed - at around
2011 - the authentication and authorization mechanisms, among other features,
must be overwritten. As at the time the team was even smaller than now,
didn't have experience with the available frameworks nor time to learn one,
the team decided to develop a spartan MVC framework, that could barely
instantiate a class and execute a method based on some parameters received
by the request.

In this study, I will approach the framework selection process and some of
the challenges faced by the team involved in this endeavour, in which I belonged to.

# The selection process

We searched the most popular PHP frameworks in 2015 and we reached to the
results of [SitePoint](http://www.sitepoint.com/best-php-framework-2015-sitepoint-survey-results/),
indicating Laravel in the first place, followed by Symfony - with a considerable
advantage.

The criteria adopted for the selection - Laravel or Symfony (or even their micro versions - Lumen and Silex), were:

- Comunity size
- Amount of available libraries
- Project activity level
- Native functionalities
- Amount of available documentation
- Learning curve

Laravel, specially due to its simplicity, low learning curve and its documentation,
seemed to be more attractive than Symfony. This one, seems to require more
design patterns knowledge and has a learning curve higher than Laravel. Symfony
brings Doctrine as its default ORM, which adopts the Data Mapper pattern. This is
ideal for bigger and more complex projects - and goes beyond my company's usual
projects and, probably, would only add complexity in the development. Laravel
brings Eloquent, which adopts the Active Record pattern - it's simpler, requires
less code and probably, could take to a higher productivity.

Laravel's valition mechanism seemed to be far easier than Symfony's, allowing
the combination of validators using pipe. The validation in Symfony
is accomplished using "Assert objects" and the combination of them can be done
using an array of these objects... it appeared to be more verbose.

As we would adopt Angular with Laravel using REST in the middle, the template
engine have not impacted in our decision, although I have liked Twig much more than
Blade, but it's just a matter of preference - Twig was developed based
on the Django template engine, which I use and love :)

With the decisions made - Laravel and Angular - we had the proposed architecture...
now we needed to develop a project composed by the the company's most common
use cases, in order to enable this architecture.

# Executable architecture development

Stuying Laravel, I checked that it haven't supported Oracle databases out of the
box, but there was a library that added this functionality - [laravel-oci8](https://github.com/yajra/laravel-oci8) - I just needed to know
if it was stable enough to support a project in production environment. My company
has a database naming convention that uses prefixes in tables, its columns and sequences.
It would be crucial that this library supported the overwritten of its own convention,
in order to keep using the company's one, as well as supporting auto incrementation
through sequences and allowing procedures execution.

Laravel has its own naming convention for tables (snake case - the table being the model
name in the plural form) and fields, such the table's ID and timestamp fields, but
allows the overwritten of this behavior. The library laravel-oci8 also has its own
sequences naming convention and its documentation said that it allowed its naming
customization.

We picked a set of use cases to develop, in which we would start implementing
from the simplest to the most complex, in order to gain experience with the
framework. In the first, we should retrieve tuples based on the combination
search parameters, as well as insert, update and delete records from table.

The retrieving for easy, Eloquent is fantastic... but the insertion was
challenging. Following the Laravel documentation, everything appeared to be right,
but the ID was not being incremented and binded to the insert! According to the
laravel-oci8 everything was right too, until I finally open an [issue](https://github.com/yajra/laravel-oci8/issues/155) at the Github's project.
Immediately the project's maintainer answered that the insert through sequences
that used a customized name, other than the default naming convention, was not
totally implemented yet and pointed me a 'workaround' to this problem (executing
the sequence manually, attributing the value to the ID field and then saving
the object). In the same week, another project contributor alerts me that the
the lacking functionality was fully implemented, I tested and voilà - it worked
as expected. Updating and deleting a record, logically and physically were
easily implemented.

We used a RESTful resource controller to provide the services the would be
consumed by Angular app, with some caveats. The `index` method that originally
renders a template with all the objects was used to return the objects filtered
based on parameters received by the request, in a JSON form, of course. The
`show` method, instead of rendering a template with a single object retrieved
by its ID, would return this object as a JSON. The `edit` method was not used.


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
