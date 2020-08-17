---
title: "pam-helper"
date: 2020-08-17T00:27:45-07:00
---

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/netauth/pam-helper/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/netauth/pam-helper)](https://goreportcard.com/report/github.com/netauth/pam-helper)

Most Linux systems use PAM.  If you want to use NetAuth to provide
accounts to your Linux fleet, `pam-helper` can provide an easy and
secure means to allow PAM to validate credentials against the NetAuth
servers in your environment.

The `pam-helper` tool is implemented as a standalone executable which
is invoked by `pam_exec.so` and is provided with the entered
credential via standard input.  If the supplied credentials are valid,
the exit code will be zero, if they are invalid an error will be
returned.

## Installation

If your distribution provides a packaged binary form of `pam-helper`
you are strongly encouraged to use it.  Verify though that it is a
recent version, especially if you are on a Debian derived
distribution.

Compiling from source follows the standard process for Go executables.

## Configuration

An example `system-auth` file is shown below:

```
#%PAM-1.0

auth    [success=2 default=ignore] pam_unix.so try_first_pass nullok
auth    [success=1 default=die] pam_exec.so expose_authtok quiet /usr/bin/pam-helper
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
