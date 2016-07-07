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

{% highlight php linenos %}
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$logger = new Logger('channel-name');
$logger->pushHandler(new StreamHandler(__DIR__.'/app.log', Logger::DEBUG));

$logger->addInfo('Hello log!');
{% endhighlight %}

When you create a `Logger` instance, you must pass a channel name. This name is
included in every log entry and allows you to easily see and filter these entries,
creating one channel for each component or aspect of your application, for example,
channels 'database', 'router', 'security', 'business' and so on.

Loggers, by itself, don't do much for us, unless we use handlers. You can push
how many handlers you need to a `logger` instance and, each type of handler you
push, decide how it's going to handle the log, for example, the `StreamHandler`
in the above example, record log entries in the file system, in this case, `app.log`.
Monolog provides
[lots of handlers](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md),
that record entries in databases, send to email addresses and specific platforms,
such as [Graylog](https://www.graylog.org/), [Slack](https://slack.com),
[Loggly](https://www.loggly.com) and [NewRelic](https://newrelic.com).

# Severity Levels

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

For detailed information about how you should use the severity levels, see
[Why should you log?](http://gabrieloliverio.github.io/2016-07-01-why-should-you-log/).

#
