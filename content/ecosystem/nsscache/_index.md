---
title: "nsscache"
date: 2018-09-25T14:08:15-07:00
---

[![Build Status](https://travis-ci.org/NetAuth/nsscache.svg?branch=master)](https://travis-ci.org/NetAuth/nsscache)
[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NetAuth/nsscache/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/NetAuth/nsscache)](https://goreportcard.com/report/github.com/NetAuth/nsscache)

**Note: nsscache is in maintenance mode.  You are strongly encouraged
to use [`localize`](../localizer).**

Linux systems derive user and group information from a set of
databases.  On some systems it is possible to load multiple databases
via the Name Service Switch.  `nsscache` can write a set of files that
are in the first format for `libnss-cache` to read and supply to the
system.

`libnss-cache` can be obtained from [its
repository](https://github.com/google/libnss-cache).

## Installation

If your distribution provides a packaged binary form of `nsscache`,
you are strongly encouraged to use this, though if your distribution
happens to be Debian derived, make sure you're getting a version
that's somewhat recent.

If your distribution does not provide `nsscache`, you'll need to
build it from source.  It is assumed that you have a Go installation
of version 1.10 or later and the `dep` Go dependency manager.

Now you can build `nsscache`:

```
$ git clone -b <version> https://github.com/NetAuth/nsscache
$ cd nsscache
$ dep ensure
$ go build -o nsscache cmd/nsscache/main.go
```

Now you can install `nsscache`.

```
$ sudo cp nsscache /usr/local/sbin/
$ sudo chown root:root /usr/local/sbin/nsscache
$ sudo chmod 0755 /usr/local/sbin/nsscache
```

Remember to update your build periodically to ensure you have
appropriate security fixes.

## Configuration

Running the binary as root will do the right thing, assuming that you
have your certificate located at `/etc/netauth.cert` and your
configuration file at `/etc/netauth.toml`.

`nsscache` is configured via flags.  Important flags that you may wish
to change are called out below:

  * `--homedir`: The home directory to provide in the passwd map.
    This will perform a string substitution on the string `{UID}`
    which maps to the NetAuth concept of an entity ID.  This can be
    useful for specifying where to mount the home directory into.
  * `--shell`: If the shell is not provided by the directory, or if
    the shell provided by the directory does not exist on this system,
    this shell will be provided to the passwd map instead.  Choose
    carefully between default security and user friendliness here.
    The secure option is the default, the friendly one is usually
    /bin/bash.
  * `--indirects`: Include indirect memberships in the group map.  For
    systems of highly secure nature, you may wish to disable this and
    only include groups that an entity is directly a member of.
  * `--min-gid` and `--min-uid`: These values control the minimum
    numeric group ID and user ID values to map.  Values below these
    are dropped from the maps.  The defaults should generally be safe,
    but ensure that you don't inadvertently cause a collision with
    local users and groups.
  * `--passwd-file`, `--group-file`, and `--shadow-file`: These files
    point to non default locations for the map files.  In general you
    should not modify these unless you have a good reason to do so.

`nsscache` provides single shot updates to the files.  You must run
`nsscache` on some sort of job controller if you want to update and pick
up new values.  Choose the update frequency that is right for you.  A
good default choice if you have no idea what to set here is 15
minutes.  This will be slightly annoying to users that have just been
created in the system, but won't otherwise hammer the NetAuth server.
