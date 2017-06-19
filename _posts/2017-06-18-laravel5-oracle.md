---
layout: post
title: "Laravel 5 + Oracle"
description: "Working with Oracle in Laravel 5"
category: development
tags: [php, laravel, oracle, database]
---
{% include JB/setup %}

The primary RDBMS my company uses is Oracle and, as you might know, Laravel
doesn't provide native support to Oracle... Fortunately there's a nice library
that completely solves this problem.

In this guide I'll help you to work with Laravel and Oracle using the
[yajra/laravel-oci8](https://github.com/yajra/laravel-oci8) amazing library.

# Installing

The installation of this library depends on your Laravel version. To make it
easier, the `yajra/laravel-oci8` is versioned according Laravel 5, so, if you're
using Laravel 5.3, you install `yajra/laravel-oci8` 5.3, and so forth.

So, assuming you've got Laravel 5.4 installed in your application:

```
composer require yajra/laravel-oci8:"5.4.*"
```

# Configuring

Then, you must edit your `config/app.php` and add the following line inside the
`providers` key.

```
Yajra\Oci8\Oci8ServiceProvider::class,
```

After that, you can publish the configuration file, running the command `php artisan vendor:publish --tag=oracle`

Publishing the configuration copies a `oracle.php` file into your `config/` directory. This file looks like this:

```
'oracle' => [
    'driver'        => 'oracle',
    'tns'           => env('DB_TNS', ''),
    'host'          => env('DB_HOST', ''),
    'port'          => env('DB_PORT', '1521'),
    'database'      => env('DB_DATABASE', ''),
    'username'      => env('DB_USERNAME', ''),
    'password'      => env('DB_PASSWORD', ''),
    'charset'       => env('DB_CHARSET', 'AL32UTF8'),
    'prefix'        => env('DB_PREFIX', ''),
    'prefix_schema' => env('DB_SCHEMA_PREFIX', ''),
],
```

Then, you can simply define the environment variables according to your server's
configuration and user's credentials and set the default driver to `oracle` in
your `.env` file. Something like this:

```
DB_CONNECTION=oracle
DB_TNS=magrathea
DB_PORT=3306
DB_DATABASE=heartofgold
DB_USERNAME=marvin
DB_PASSWORD=fortytw0
```

If you don't like the approach of having two files handling the database
configuration - `config/database.php` and `config/oracle.php`, you can copy the
configuration above to your `config/database.php`, inside the `connections` key.

# Using

The `yajra/laravel-oci8` library allows you to do everything you usually would
do with a native driver.

## Eloquent

There are a few Oracle particularities you must pay attention when creating your
Eloquent models - sequences and BLOBs, except for this, no difference from
models for any other driver.

### Sequences

It's possible to use sequences - the Oracle obscenity used to auto-increment.
The format defined by the library is `{$table}_{$column}_seq`, but you can
override this behavior if you want or need to use another format.

```
class Person extends Model {

}
```

In the example above we use the default values defined by Laravel and
yajra/laravel-oci8, so, `$table = 'persons'`, `$primaryKey = 'id'` and
`$sequence = 'persons_id_seq'`.

After creating the sequence and defining it in your model, use it normally:

```
class Person extends Model {
    public $table = 'people';
    public $sequence = 'sq_people';
}

$person = new Person();
$person->name = 'Marvin, the Paranoid Android';
$person->save();
```

### BLOBs

In order to use a BLOB or CLOB column, your model's got to extend
yajra/laravel-oci8's Eloquent Model class and define these BLOB/CLOB columns in a
`binaries` property:

```
use yajra\Oci8\Eloquent\OracleEloquent as Model;

class Person extends Model {
    protected $binaries = ['image', 'bio'];
}
```

That's enough.


## Executing procedures

Unhappily, sometimes you've got to call procedures. Fortunately you can do that
`yajra/laravel-oci8`! Let's suppose the following procedure:

```
CREATE PROCEDURE update(
    pid IN NUMBER,
    pname IN VARCHAR,
    pstatus OUT NUMBER) AS
    BEGIN
        UPDATE people SET
        name = pname
        WHERE id = pid;

        pstatus := 1;
    END;
```

Now let's execute it.

## The manual way

```
function update() {
    $pdo = DB::getPdo();
    $status = '';

    $stmt = $pdo->prepare("begin update(:pid, :pname, :pstatus); end;");
    $stmt->bindParam(':pid', $this->id, PDO::PARAM_INT);
    $stmt->bindParam(':pname', $this->name, PDO::PARAM_STR);
    $stmt->bindParam(':pstatus', $status, PDO::PARAM_INT);
    $stmt->execute();

    return $status;
}
```

# The reduced way

function update() {
    $status = '';
    $result = DB::executeProcedure('update', [
        'pid'  => $this->id,
        'pname'  => $this->name,
        'pstatus'  => $this->status,
    ]);

    return $status;
}

Much better, huh?!

# Above and beyond

You can do a lot of more stuff using the `yajra/laravel-oci8` library - migrations,
which allow you to create/alter/delete tables and sequences. You can execute functions,
load triggers, get and increment sequences manually, use the Fluent to insert a
registry automatically incrementing its ID and so on.

This was just an introductory article to the `yajra/laravel-oci8` library. Read
more on [its documentation](https://yajrabox.com/docs/laravel-oci8).

Hope that helps!
