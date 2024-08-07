---
title: "Ecosystem"
date: 2018-09-25T01:02:56-07:00
---

NetAuth isn't a single isolated daemon and command line tool, there's
a suite of software that plugs into it to create a functional service
landscape.  This page details this software.

## [NetAuth](netauth/)

The core server itself and the command line tools.

## [pam_netauth](pam_netauth/)

Deprecated: See pam-helper instead.

## [pam-helper](pam-helper)

Most Linux and some BSD systems use Pluggable Authentication Modules
to authenticate users.  NetAuth can interface with PAM using
pam-helper.

## [ldap](ldap/)

Connect systems that understand simple LDAP queries to NetAuth.

## [localizer](localizer/)

Map your NetAuth entities and groups into Linux systems.

## [nsscache](nsscache/)

Deprecated: See localizer instead.

Systems that use glibc to access the passwd and group databases can
make use of nsscache to generate appropriate files for libnss-cache.

## [NetKeys](netkeys/)

Need to obtain keys from NetAuth for another system?  NetKeys can do
this and is an ideal drop in to use with OpenSSH's
AuthorizedKeysCommand.

## [Buildbot](https://github.com/classabbyamp/buildbot-netauth)

Need to authenticate to your Buildbot installation? Buildbot-netauth
allows Buildbot to communicate directly with NetAuth for login and
user information.

## [Plugins](plugin)

The core server can be extended by using plugins to augment the
default behavior.  Plugins can either change built-in behaviors, or
can propagate events to external systems.

## [Python](https://github.com/NetAuth/netauth-python)

NetAuth can be easily integrated into Python-based applications with
the `netauth` Python library.

## [Raw gRPC](https://github.com/NetAuth/Protocol)

Don't see what you're looking for here?  You can develop your own
service that speaks NetAuth's protocol in the language of your choice
using the protocol definition.
