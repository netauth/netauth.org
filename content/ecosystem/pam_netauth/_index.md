---
title: "pam_netauth"
date: 2018-09-25T01:12:27-07:00
---

*DEPRECATION NOTICE: The legacy `pam_netauth.so` mechanism is
deprecated and should not be used.  It relies on FFI interfaces that
are difficult to test and maintain and is no longer under active
development.  Additionally it does not work on non-glibc systems.  You
should use [pam-helper](../pam-helper/) instead.*

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NetAuth/pam_netauth/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/NetAuth/pam_netauth)](https://goreportcard.com/report/github.com/NetAuth/pam_netauth)

Most Linux systems use PAM.  If you want to use NetAuth to provide
accounts to your Linux fleet, there are some additional packages that
are highly recommended.

### libpam-policycache

While NetAuth strives to be as reliable as possible, it is still
possible that your NetAuth servers might be down when you're trying to
log in.  To resolve this case it is highly recommended that you have a
password cache on your machines.  This allows the system to keep
working if you've logged in recently.

libpam-policycache can be obtained in source form from [its
repository](https://github.com/google/libpam-policycache).  Binary
packages are available for a number of popular Linux distributions.

## Installation

If your distribution provides a packaged binary form of pam_netauth,
you are strongly encouraged to use this, though if your distribution
happens to be Debian derived, make sure you're getting a version
that's somewhat recent.

If your distribution does not provide pam_netauth, you'll need to
build it from source.  It is assumed that you have a Go installation
of version 1.10 or later and the `dep` Go dependency manager.  You
must also obtain the PAM headers which will usually be in a package
with a name similar to `pam-devel`.

Now you can build pam_netauth:

```
$ git clone -b <version> https://github.com/NetAuth/pam_netauth
$ cd pam_netauth
$ dep ensure
$ go build -buildmode=c-shared -o pam_netauth.so
```

To install `pam_netauth` locate where your system stores security
modules, usually `/usr/lib/security`:

```
$ sudo cp pam_netauth.so /usr/lib/security/
$ sudo chown root:root /usr/lib/security/pam_netauth.so
$ sudo chmod 0755 /usr/lib/security/pam_netauth.so
```

## Configuration

Configuring a service to use `pam_netauth` takes the same form as
configuring any other PAM service.  An example `system-auth` file is
shown below that includes the recommended approach with
libpam-policycache.  `pam_netauth` implements the `auth` service only.

Note that for `pam_netauth` to work, you will need an existing NetAuth
configuration file installed at `/etc/netauth/config.toml`.

Example `system-auth`:

```
#%PAM-1.0

auth    [success=4 default=ignore] pam_unix.so try_first_pass nullok
auth    [success=3 default=ignore] pam_policycache.so try_first_pass action=check
auth    [success=ok default=die] pam_netauth.so try_first_pass
auth    [success=1 default=ignore] pam_policycache.so try_first_pass action=update
auth    required  pam_deny.so
auth    required  pam_env.so
auth    required  pam_permit.so

account   required  pam_unix.so
account   optional  pam_permit.so
account   required  pam_time.so

password  required  pam_unix.so     try_first_pass nullok sha512 shadow
password  optional  pam_permit.so

session   required  pam_mkhomedir.so
session   optional  pam_umask.so    usergroups
session   required  pam_limits.so
session   required  pam_unix.so
session   optional  pam_permit.so
```

Note that in the above file `pam_mkhomedir.so` is used to provide
home directories instead of networked storage.  If your environment
has networked storage substitute an appropriate module such as
`pam_mount.so`
