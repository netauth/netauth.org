---
title: "fail2lock"
date: 2019-09-12T15:49:54-07:00
---

The fail2lock plugin helps you maintain the security of your system by
limiting the number of attempts that can be made in a short period of
time.

The fail2lock plugin is part of NetAuth and serves as an example
plugin, but is not distributed with the NetAuth tarball by default.
To get a copy you must download the source code and compile it:

```shell
$ git clone git://github.com/NetAuth/NetAuth.git
$ cd NetAuth
$ go build -o fail2lock.treeplugin plugins/tree/fail2lock/main.go
```

The resulting file should be installed in your NetAuth server's plugin
directory.  After installing the plugin binary to your system, you can
append configuration to your config.toml file.  The stanza for NetAuth
is as follows:

```toml
  [plugin.fail2lock]
    interval = "5m"
    allowed_fails = 3
```

The interval is a period of time in which to count bad login attempts.
The allowed_fails is how many bad attempts will be allowed before
returning failures related to exceeding the allowed attempts.

Of course sometimes its necessary to reset the login counter quickly
if a user is aware that they were using, for example caps lock, when
trying to type in a secret.  To facilitate this a the key value pair
of `fail2lock:RESET` will clear the list of failed attempts.  Use of
this flag should be sparing, as it defeats a security protection of
used improperly.
