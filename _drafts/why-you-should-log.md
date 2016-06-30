---
layout: post
title: "Why you should log"
description: ""
category: programming
tags: [programming]
---
{% include JB/setup %}

Logs have been used for a long time in many areas to record important events.
In shipping, logs are kept to help the crew to navigate when GPS, radio or radar
fail. Logs are used also in marine inquiries, similar to an airplane black box,
registering weather conditions, ports docked at, significant incidents and distance
traveled with respect to a given start position.

We know you're not member of ship crew, but the logs used in programming and
operating systems are VERY similar, obviously, because our logs are based
on the shipping logs!

There are two main purposes for logs in our area - debugging, similar to
the logs used to help the ship crew, and audit, that are used in investigation
processes. We will focus on the first type of log.

# Benefits of logging

- It allows you know exactly the execution flow of your application and can
use it even in production environment
- You can take a proactive approach, acting in cases even  

# What should you log?

Simple, you should log everything you judge as important! No one, unless you
know, what is important for your application. To help you identify what is
important, let's take a look at some log entries I took from `dmesg`:

```
[    0.755157] PM: Hibernation image not present or could not be loaded.
[    0.755848] Freeing unused kernel memory: 1336K (ffffffff81d20000 - ffffffff81e6e000)
[    0.755849] Write protecting the kernel read-only data: 12288k
[    9.544173] Adding 3905532k swap on /dev/sda2.  Priority:-1 extents:1 across:3905532k FS
[    9.552651] IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
[    0.205464] pci 0000:00:1c.0: PME# supported from D0 D3hot D3cold
[    0.205704] pci 0000:00:1c.3: System wakeup disabled by ACPI
[
```

None of the above log entries are error messages, they are useful information
obtained and logged in specific events, such as turning the computer on,
enabling/disabling network adapters, plugging storage devices and so on.
Services such as the Apache Httpd, for example, log every request and its
context data (we will see that later) as well as errors, like restarting the
server with an invalid virtual host, and events as starting, stopping and
restarting the server. All these log entries can be used by system administrators
to possibly find a problem.

## Severity levels
There are usually five severity levels for logs in logging frameworks:
debug, info, notice/warning, error and critical.

**Debug**: It's that one that helps you to find bugs, in which you log an event, such
a condition evaluated to true and a variable value, for example.

**Info**: Allows you knowing your application's execution flow. In
this type, you usually log events, such a user registered, a user that logged
the system, a payment refused and so on.

**Warning/notice**: These are used when something unusual happened in your system,
but it didn't crashed and its user interface was not affected. Up to this point
you are able to add logs manually, placing the log framework calls in catch
blocks or based on specific conditions. However, it's really interesting if you use a
kind of exception handler to deal with events starting from this log severity,
so that you can handle certain exceptions, maybe logging them and showing a default
error message, and deal with unexpected exceptions.

**Error**: Used when something definitely went wrong, such as runtime errors, BUT it
doesn't require immediate action.

**Critical**: Used when something critical happens, such as a system component
unavailable, for example.

Some frameworks go beyond these severity levels. Monolog (PHP), for example,
have two others: Alert - for actions that must be taken immediately, waking
you up on the middle of the night on your vacations, such as entire system down,
database unavailable and so on - and Emergency, when the system is unusable.

Sincerely, Emergency and Alert for me are the same. When the entire system is
down, isn't the system unusable, most of the times? Now it's up to you decide at
what severity level you have to use, I particularly never used Alert and Emergency.
When something really serious happens, such as a database unavailable, I add a
critical log entry and, thus, send an e-mail message for the team, but that's my need.
If you need sending a SMS message for you no matter the time, go ahead if it's
you need/want.

## Context data

Context data is the information about you're logging and not using them is almost
the same as not logging at all! Consider for example the log entry below:

```
[    9.552651] IPv6: link is not ready
```

So you have a problem, is investigating your `dmesg` output and see that a link
isn't ready... This could help you if you knew the network interface!

Other example closer to our lives: you have an event subscription website and
someone is having a problem when tries to subscribe in a specific event, which
log entries you think would help the most?

### These

```
[2016-06-30 12:40:05] [INFO] User registered
[2016-06-30 12:42:11] [INFO] User activated
[2016-06-30 12:45:03] [INFO] Event subscribed by user
[2016-06-30 12:46:32] [ERROR] Fail subscribing to event by user
```

### Or these?

```
[2016-06-30 12:40:05] [INFO] User 'gabrieloliverio' registered
[2016-06-30 12:42:11] [INFO] User 'gabrieloliverio' activated
[2016-06-30 12:45:03] [INFO] Event 'Foo' subscribed by user 'gabrieloliverio'
[2016-06-30 12:46:32] [ERROR] Fail subscribing to event 'Bar' by user 'gabrieloliverio'
```

Now think in a log file with thousands of entries, with several users registering
and subscribing to events simultaneously... You can take your own conclusions :)

# Log handlers

Logging frameworks usually have something they call 'handlers', which are
structures that allow you to handle log entries based on its severity level.
This way, you can use a file handler to store logs with severity Info to Critical
in your filesystem and an email handler, to send an email message to your team
with log entries with Critical severity level.

Most of the logging frameworks support a myriad of handlers that allow you
send your log entries to other tools, such as [Slack](https://slack.com),
[NewRelic](https://newrelic.com/), [Loggly](https://www.loggly.com/) and
[Graylog](www.graylog2.org), as well as storing your entries in databases,
like [Redis](http://redis.io/) and [MongoDB](https://www.mongodb.com/), for
example.
