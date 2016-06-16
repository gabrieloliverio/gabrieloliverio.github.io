---
layout: post
category : linux
title: "Fixing the 'sudo: /etc/sudoers.d is world writable' problem"
description: ""
tags : [ubuntu, php]
---
{% include JB/setup %}

I've just helped a colleague with a problem (beyond the mental one), when
he tried to execute `sudo`. For some reason, EVERYTHING in his `/etc/`
had 777 permission - that's why the `sudo: /etc/sudoers.d is world writable`.
I tried to login as root, executing `su`, but `su: Authentication failure` was
shown... how could I change the permissions if I wasn't able to login as root?
Follow the instructions above to solve this problem.

To login as root, without `su` or `sudo`, you can use `pkexec`:

```bash
pkexec su
```

Now change the files' permissions:

```bash
chmod 440 /etc/sudoers
chmod 775 /etc/sudoers.d
chmod 440 /etc/sudoers.d/README
```

That's all - now you must be able to `sudo`.
