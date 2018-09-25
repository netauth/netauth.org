---
title: "NetKeys"
date: 2018-09-25T15:49:54-07:00
---

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NetAuth/NetKeys/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/NetAuth/NetKeys)](https://goreportcard.com/report/github.com/NetAuth/NetKeys)

Some services provide a way to obtain public keys via a directory.  A
classic example is OpenSSH which can obtain the public keys for a user
via an external executable.  `NetKeys` can provide these applications
with a small executable that can run as a dedicated user.

## Installation

If your distribution provides a packaged binary form of `NetKeys`,
you are strongly encouraged to use this, though if your distribution
happens to be Debian derived, make sure you're getting a version
that's somewhat recent.

If your distribution does not provide `NetKeys`, you'll need to
build it from source.  It is assumed that you have a Go installation
of version 1.10 or later and the `dep` Go dependency manager.

Now you can build `NetKeys`:

```
$ git clone -b <version> https://github.com/NetAuth/NetKeys
$ cd NetKeys
$ dep ensure
$ go build -o netkeys cmd/netkeys/main.go
```

Now you can install `NetKeys`.

```
$ sudo cp netkeys /usr/local/bin/
$ sudo chown root:root /usr/local/bin/netkeys
$ sudo chmod 0755 /usr/local/bin/netkeys
```

Remember to update your build periodically to ensure you have
appropriate security fixes.

## Configuration

`NetKeys` has virtually no configuration of its own beyond the normal
values available in `/etc/netauth.toml`.  The only notable option is
-ID which expects to take an entityID, any keys the entity had of the
specified `-type` will be printed to stdout.  You may wish to
additionall specify `-service` for logging purposes.

An example snippet of how to obtain ssh keys with `NetKeys` is below:

```
AuthorizedKeysCommandUser _sshd_keyuser
AuthorizedKeysCommand /usr/local/bin/netkeys --ID %u
```

Note that `AuthorizedKeysCommand` must be a fully qualified path.  If
you build from source use the path above, otherwise, package manager
binaries usually wind up in `/usr/bin/`.
