---
title: "Localizer"
date: 2020-04-18T20:36:27-07:00
---

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NetAuth/localizer/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/NetAuth/localizer)](https://goreportcard.com/report/github.com/NetAuth/localizer)

Linux systems derive user and group information from the `/etc/passwd`
and `/etc/group` files by default.  The localizer utility edits these
files for you to add and remove users that are present in the NetAuth
system to your Linux computers.

## Installation

If your distribution provides a packaged binary form of `localize`,
you are strongly encouraged to use this, though if your distribution
happens to be Debian derived, make sure you're getting a version
that's somewhat recent.

If your distribution does not provide `localize`, you'll need to build
it from source.  It is assumed that you have a Go installation of
version 1.13 or later.

Now you can build `localize`:

```
$ git clone -b <version> https://github.com/NetAuth/localizer
$ cd localizer
$ go build -o localize cmd/localize/main.go
```

Now you can install `localize`.

```
$ sudo cp localize /usr/local/sbin/
$ sudo chown root:root /usr/local/sbin/localize
$ sudo chmod 0755 /usr/local/sbin/localize
```

Remember to update your build periodically to ensure you have
appropriate security fixes.

## Configuration

Running the binary as root will do the right thing, assuming that you
have your certificate located at `/etc/netauth.cert` and your
configuration file at `/etc/netauth.toml`.

`localize` can be configured via flags.  Important flags you may need
to override are provided below:

  * `--base`: This overrides the default location for the `passwd` and
    `group` files to be somewhere other than the standard `/etc`
    directory.  You should not ever need to change, this.
  * `--shell`: If the shell is not provided by the directory, or if
    the shell provided by the directory does not exist on this system,
    this shell will be provided to the passwd map instead.  Choose
    carefully between default security and user friendliness here.
    The secure option is the default, the friendly one is usually
    /bin/bash.
  * `--min-gid` and `--min-uid`: These values control the minimum
    numeric group ID and user ID values to map.  Values below these
    are dropped from the maps.  The defaults should generally be safe,
    but ensure that you don't inadvertently cause a collision with
    local users and groups.

`localize` provides single shot updates to the system.  You must run
`localize` on some sort of job controller if you want to keep the
local system up to date with the information contained in NetAuth.
Choose the update frequency that is right for you.  A good default
choice if you have no idea what to set here is 30 minutes.  This will
be slightly annoying to users that have just been created in the
system, but won't otherwise cause undue load on the NetAuth server.
