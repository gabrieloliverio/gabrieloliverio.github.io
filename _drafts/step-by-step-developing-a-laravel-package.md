---
layout: post
title: "Step by step - Developing a Laravel 5 package"
description: ""
category: development
tags: [php, laravel, development, library, package]
---
{% include JB/setup %}

Some months ago, when I started to develop my first Laravel 5 package, I didn't
even know how to start. It simply lacks good materials introducing this subject,
that could help me to not commit some silly mistakes.

What I'll try to do, is to provide an introductory material, a guideline on
how to develop your first Laravel 5 package.

--------------------------------

# #1 - Learn how to use Composer

The package you're going to develop will be probably a dependency of another
project, right? So, it's indispensable to go a bit deeper on Composer.

# #1.1 Define your composer.json

First thing you should do: fill your `composer.json` - author(s), description,
keywords, license and all of this stuff. The `name` key requires more attention:
it must be in the form 'vendor/package', e.g. `monolog/monolog`. This field is
required when you're going to publish your package on Packagist, where 'vendor'
is your user name on this site.

Another key that requires even more attention is the `psr-4`.

>PSR-4 is a specification for autoloading classes from file paths

Usually, a package consists of a directory, with files like `composer.json`,
`readme.md` and `.gitignore`, for example. The actual source lies in a `src`
directory inside your package. This way, a `psr-4` key could be something like
this:

```
"psr-4": {
    "Vendor\\MyAwesomePackage\\": "src/"
}
```

Note that your classes must be inside this `Vendor\Package` namespace:

```
<?php namespace Vendor\MyAwesomePackage\NestedNamespace;

class AwesomeClass {
    ...
```

The full documentation about the `composer.json` can be found
[here](https://getcomposer.org/doc/04-schema.md).

# #1.2 Require your package from a test project

In order to test your package, you need to have a test project. In this project's
`composer.json` file you can add a `repositories` key pointing to your package,
that's probably a Git/Hg/whatever repository, this way:

```
"repositories": [
    {
        "type": "vcs",
        "url": "/home/user/Projects/my-awesome-package"
    }
],
```

Then require your package, as you do with any other:

```
"require": {
    "php": ">=5.6.4",
    "laravel/framework": "5.4.*",
    "vendor/my-awesome-package": "dev-master"
},
```

Note that the version required by the test project is `dev-master` - it corresponds to a branch `master` (prefixed by `dev`). If you've got a branch named `test`, for example, and want to use this branch instead of `master`, just require `dev-test`.

# #2 Consider to use contracts instead of facades

As recommended in the [Laravel docs](https://laravel.com/docs/5.4/contracts):

>Most applications will be fine regardless of whether you prefer facades or contracts. However, if you are building a package, you should strongly consider using contracts since they will be easier to test in a package context.

Reasons?
# #3 Implement test cases
