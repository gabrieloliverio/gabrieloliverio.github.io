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

We got to a point that this technology became obsolete, difficult to maintain and
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

In this study, I will address the framework selection process, executable
architecture development and some of the challenges faced by the team involved
in this endeavour, in which I had the opportunity to belonged to.

# The selection process

We searched the most popular PHP frameworks in 2015 and we reached to the
results of [SitePoint](http://www.sitepoint.com/best-php-framework-2015-sitepoint-survey-results/),
indicating Laravel in the first place, followed by Symfony - with a considerable
advantage.

The criteria adopted for the selection - Laravel or Symfony (or even their micro versions - Lumen and Silex), were:

- Community size
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
projects and, probably, would only add complexity in the development process. Laravel
brings Eloquent, which adopts the Active Record pattern - it's simpler, requires
less code and probably, could take to a higher productivity.

Laravel's validation mechanism seemed to be far easier than Symfony's, allowing
the combination of validators using strings and a pipe, like
`required|max:20|min:10`. The validation in Symfony is accomplished using
"Assert objects" and the combination of them can be done using an array of
these objects... it appeared to be much more verbose. Don't get me wrong
Symfony fan... I believe Symfony is a great framework and I want to have
the opportunity to work with it in a near future.

As we would adopt Angular with Laravel using REST in the midfield, the template
engine have not impacted in our decision, although I have liked Twig much more than
Blade, but it's just a matter of preference - Twig was developed based
on the Django template engine, which I use and love :)

With the decisions made - Laravel and Angular - we had the proposed architecture...
now we needed to develop a project composed by the the company's most common
use cases, in order to enable this architecture.

# Executable architecture development

Studying Laravel, I had already noticed that it doesn't provide native support to Oracle databases, but there was a library that added this functionality - [laravel-oci8](https://github.com/yajra/laravel-oci8) - I just needed to know
if it was stable enough to support a project in production environment. My company
has a database naming convention that uses prefixes in tables, its columns and sequences.
It would be crucial that this library supported the overwritten of its own convention,
in order to keep using the company's one, as well as supporting auto incrementation
through sequences and allowing procedures execution.

Laravel has its own naming convention for tables (snake case - the table being the model
name in the plural form) and fields, such the table's ID and timestamp fields, but
allows the overwritten of this behavior. The library `laravel-oci8` also has its own
sequences naming convention and its documentation said that it allowed its naming
customization.

We picked a set of use cases to develop, in which we would start implementing
from the simplest to the most complex, in order to gain experience with the
framework. In the first, we should retrieve tuples based on the combination of
search parameters, as well as insert, update and delete records from table.

The retrieving for easy, Eloquent is fantastic... but the insertion was, well...
challenging. Following the Laravel documentation, everything appeared to be right,
but the ID was not being incremented and binded to the insert! According to the
`laravel-oci8` everything was right too, until I finally opened an [issue](https://github.com/yajra/laravel-oci8/issues/155) at the project's Github.
Immediately the its maintainer answered that the insert through sequences
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
based on the parameters received by the request, in a JSON form, of course. The
`show` method, instead of rendering a template with a single object retrieved
by its ID, would return this object as a JSON. The `edit` method was not used.

One of the biggest challenges to face in the implantation would be the integration
with the Identification Management system. In this system, there's a database user
for each system user, in which its credentials must be used in every database
connection. To do so, we must develop our own authentication method.

This authentication method was developed as a User Provider, as Laravel calls,
and was surprisingly easier than we expected. As were using a API approach, we
decided to implement the [JWT](https://jwt.io/) (JavaScript Web Token) - a stateless
pattern that uses a token in the requests to keep the user authenticated, using
the (great) library [jwt-auth](https://github.com/tymondesigns/jwt-auth).
Using JWT we wouldn't need to store our session in a Redis or Memcached database
in order to allow us to fully implement Single Sign-On, as well as would enable
us to integrate mobile apps, that are slowly emerging in my company and legacy
desktop applications.

We proceed implementing the more complex use cases, where we tested transactions,
models using tables from other databases and database engines, as well as
executing stored procedures - all of them well succeeded.

# Final considerations

We have finished the executable architecture within the time limit (2 months) and
developed classes for most of the company specific needs, such as the
authentication and authorization methods, which will be incorporated into a
dedicated project. This one will be included as a dependency of all the projects
being developed using this architecture.

Although we have developed this executable architecture and, consequently,
reduced most of the risks in this implantation, there's always a risk of facing
a problem, but, to me, this is a risk that is worth to cope with. My company's
needs aren't too specific that nobody hasn't done something similar and, perhaps,
provide the source at Github. If there is not already an available solution, the
nowadays PHP frameworks, like Laravel and Symfony, are developed to deal with
extensions, as we could note when we needed to develop our authentication method.

We hope that, with this framework implantation, we could improve our systems'  
maintainability, reliability and security, as well as increase our productivity
as a team. I intend to edit this post as soon as we finish our first system
using this architecture, telling our opinions about the process and any
challenges that might occur.
