---
layout: post
title: "Logging with PHP and Monolog"
description: ""
category: php
tags: [php, programming]
---
{% include JB/setup %}

In in the article
[Why should you log?](http://gabrieloliverio.github.io/2016-07-01-why-should-you-log/)
I wrote the logging benefits, what you should log and some key concepts. In this article
we will materialize logging implementing it using [Monolog](https://github.com/Seldaek)
and PHP.

Monolog is the *de facto* standard logging library for PHP and comes out of the
box in the most popular PHP frameworks, such as Laravel and Symfony.  It implements
[PSR-3](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md) -
a common interface for logging libraries defined by [PHP-Fig](http://www.php-fig.org).
Type-hinting `Psr\Log\LoggerInterface` in your application enables interoperability,
allowing you change the logging library for another that implements PSR-3 without
too much headache.

# Installing

Everything is easy using Composer:

```bash
composer require monolog/monolog
```

If you're not using Composer, just download
Monolog's **AND** the [PSR-3' source code](https://github.com/php-fig/log),
make it available to your application and start using it.

If you're not using namespaces (Oh man! I already did that!), you'll have a lot
of work requiring files...

# Creating loggers

This is the minimum code required to log something using Monolog:

```php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$logger = new Logger('channel-name');
$logger->pushHandler(new StreamHandler(__DIR__.'/app.log', Logger::DEBUG));

$logger->addInfo('Hello log!');
```

That's pretty straightforward: we create a `Logger` instance, push handlers and
add the log entry, in this case, with severity `info`.

When you create a `Logger` instance, you must pass a channel name. This name is
included in every log entry and allows you to easily see and filter these entries,
creating one channel for each component or aspect of your application, for example,
channels 'database', 'router', 'security', 'business' and so on.

Loggers, by itself don't know how to handle logs, but the handlers do! You can push
how many handlers you need to a `logger` instance and, each type of handler you
push, decide how it's going to handle the log, for example, the `StreamHandler`
in the above example, record log entries in the file system, in this case, `app.log`.
Monolog provides
[lots of handlers](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md),
that record entries in databases, send to email addresses and specific platforms,
such as [Graylog](https://www.graylog.org/), [Slack](https://slack.com),
[Loggly](https://www.loggly.com) and [NewRelic](https://newrelic.com).

# Severity Levels

Severity levels describe how severe is the entry log added and, as well as the
channel name, allows you to see and filter the entries.

Monolog supports the eight severity levels described by
[RFC 5424](http://tools.ietf.org/html/rfc5424):

- **Debug**: Detailed information used for debugging
- **Info**: Informational messages, used to record normal events
- **Notice**: Informational messages too, but used with more significant events
- **Warning**: Warning conditions, where there isn't an error yet, but you
should take an action or it will become an error
- **Error**: Error conditions, used when something definitely went wrong but
doesn't require immediate action
- **Critical**: Critical conditions, such as a system component unavailable, for
example
- **Alert**: Used for actions that must be taken immediately, waking you up
in the middle of the night
- **Emergency**: Used when the system gets unusable

An `info` log entry of a channel called 'default', for example, looks like this:

```
[2016-07-06 11:54:23] default.INFO: Hey, this is a log entry!

```

Monolog provides one `add` for each type of severity, thus, you should call
`addDebug`, `addInfo`, `addWarning` and so forth.

For detailed information about how you should use the severity levels, see
[Why should you log?](http://gabrieloliverio.github.io/2016-07-01-why-should-you-log/).

# Handler order

The order you push the handler to the `logger` instance matters, cause whenever
you add a log entry, it will traverse the handler stack and be handled by them.
The handler's constructor have got the parameter `$bubble` which, when set to false,
stops the traversing of the handler stack. Let's see a more useful and complex
example to make things clear:

```
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\SwiftMailerHandler;
use Swift_SmtpTransport;
use Swift_Message;
use Swift_Mailer;

$transport = Swift_SmtpTransport::newInstance('smtp.vcela.cz', 25);
$message = new Swift_Message('A CRITICAL log was added');
$message->setFrom('noreply@vcela.cz');
$message->setTo('sara@vcela.cz');
$mailer = Swift_Mailer::newInstance($transport);

$logger = new Logger('default');
$logger->pushHandler(new StreamHandler(__DIR__.'/app.log', Logger::INFO));
$logger->pushHandler(new SwiftMailerHandler($mailer, $message, Logger::CRITICAL, false));

$logger->addCritical('Hey, a critical log entry!');
```

The above example uses the `SwiftMailerHandler`, that explains the increased quantity
of `use`'s. Also, you need to install [SwiftMailer](http://swiftmailer.org/download).
To send emails through SwiftMailer, you need instances of `Swift_SmtpTransport`,
`Swift_Message` and `Swift_Mailer` as well as to bind the `$mailer` and `$message`
to the `SwiftMailerHandler` instance itself.

The handling starts from the handler located at the top, thus, `SwiftMailerHandler`
(push last) is the first to handle the log entry, sending and e-mail, then, the log is
stored in the file system by `StreamHandler`.

Notice the last argument passed to the `SwiftMailer`'s constructor - that's the
`$bubble` parameter. This parameter defaults to `true` and, when set to `false`,
stops the traversing of the handler stack, making the `StreamHandler`, located
at the bottom, never storing logs in the file system.

# Context data

Context data is the information about you’re logging and I wrote lots of things
about it in that article I cited twice, so, if you're interested, you know
the address :)

Monolog provides two ways of including context data in your log entries - passing an
array together the log message and using processors.

## Passing an array

This way you pass an array to the `add` method (`addDebug` or `addInfo`, for example)
after the log message:

```
$username = 'gabrieloliverio';
$logger->addInfo('User registered', ['username' => $username]);
```

## Using processors

Processors include information automatically to the logs. It's a very useful way
to add information, like session and request data, such IP and browser, for example.
Monolog provides a
[bunch of processors](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md#processors), but they can be any callable that receives a parameter corresponding
the entry and must return this entry, eventually changed:

```
$logger = new Logger('default');
$logger->pushHandler(new StreamHandler(__DIR__.'/app.log', Logger::INFO));
$logger->pushProcessor(function ($record) {
    $record['extra']['data'] = 'Hello world!';
    return $record;
});
$logger->addInfo('User registered', ['username'=>'gabrieloliverio']);
```

This would store log entry such as the following:

```
[2016-07-06 11:54:23] default.INFO: User registered {'username':'gabrieloliverio'} {"data":"Hello world!"}
```
