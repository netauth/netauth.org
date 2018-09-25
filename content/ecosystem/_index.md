---
title: "Ecosystem"
date: 2018-09-25T01:02:56-07:00
---

NetAuth isn't a single isolated daemon and command line tool, there's
a suite of software that plugs into it to create a functional service
landscape.  This page details this software.

## [pam_netauth](pam_netauth/)

Most Linux and some BSD systems use Pluggable Authentication Modules
to authenticate users.  NetAuth can interface with PAM using
pam_netauth.

## [nsscache](nsscache/)

Systems that use glibc to access the passwd and group databases can
make use of nsscache to generate appropriate files for libnss-cache.

## NetKeys

Need to obtain keys from NetAuth for another system?  NetKeys can do
this and is an ideal drop in to use with OpenSSH's
AuthorizedKeysCommand.

## [Raw gRPC](https://github.com/NetAuth/Protocol)

Don't see what you're looking for here?  You can develop your own
service that speaks NetAuth's protocol in the language of your choice
using the protocol definition.
