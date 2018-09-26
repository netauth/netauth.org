---
title: "NetAuth"
date: 2018-09-25T20:24:08-07:00
---

[![Build Status](https://travis-ci.org/NetAuth/NetAuth.svg?branch=master)](https://travis-ci.org/NetAuth/NetAuth)
[![GoDoc](https://godoc.org/github.com/NetAuth/NetAuth?status.svg)](https://godoc.org/github.com/NetAuth/NetAuth)
[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NetAuth/NetAuth/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/NetAuth/NetAuth)](https://goreportcard.com/report/github.com/NetAuth/NetAuth)

NetAuth is itself a server and a set of command line tools to manage
that server.  It provides a system of record for identity and
authentication information.  This system can then be used as a root of
trust for other downstream systems that speak standard protocols such
as LDAPv3, OAuth 2.0, or other well known protocols.

NetAuth is written in Go and has strict rules surrounding code testing
and release stability.  The releases attempt to follow Semantic
Versioning, but some allowances are made given that gRPC is very
resiliant to missing fields.

Comprehensive documentation is
available [here](https://docs.netauth.org), but this page provides
enough to get up and running to evaluate NetAuth.

## Installation

NetAuth is not available in compiled form from the project directly,
you may compile it yourself or obtain it from your distribution's
package manager.  On Debian derived systems ensure that you are
getting a recent vresion as the NetAuth project does not offer
security backports.

To compile NetAuth you'll need a working Golang compiler at version
1.10 or better.  You'll also need `dep`, the Go package manager.

```
$ mkdir -p $GOPATH/src/github.com/NetAuth
$ cd $GOPATH/src/github.com/NetAuth
$ git clone -b <version> https://github.com/NetAuth/NetAuth
$ cd NetAuth
$ dep ensure
$ go build -o netauthd cmd/netauthd/main.go
$ go build -o netauth cmd/netauth/main.go
```

This will produce a pair of executable files `netauth` and `netauthd`.
Per convention, `netauthd` is the server executable.  You may copy
these to where your system keeps user installed applications for
secure use, usually `/usr/local/sbin`.

## Configuration

Configuration is really best left to the manual linked above, but for
a quickstart there's two important things to note:

### netauth.toml

The `netauth.toml` file is examined by the NetAuth client library
which is linked into all tools and programs that can communicate with
the server.  You should create this file and include some content in
it so that your client can find your server.  An example file is as
shown:

```toml
Server="mynetauthserver.internal"
```

This file is normally read from `/etc/netauth.toml` but for testing
things and for circumstances where you might need to override things,
you can override this path with the environment variable `NACLCONFIG`
which can contain either a relative or absolute path to the config
file you wish to use.

### Certificates

Remember that NetAuth is dealing with secure information.  This
information must be protected while in transit, and NetAuth uses TLS
to ensure this security.  While TLS is a must for a production setup,
it can be annoying to deal with while doing a local demo or otherwise
just trying out NetAuth.  If you want to turn off TLS and understand
the risks that are associated with this the `-PWN_ME` flag can be used
to disable TLS.

This flag is no joke, you will be sending passwords in the clear.
Anyone on your network with the knowhow to do so can sniff these
passwords and use them.  Don't run without TLS for anything more than
a toy demo, and even then consider against it since tools like `cfssl`
make it very easy to generate certificates!

## 5 Minute Demo

Lets do a 5 minute demo where we start a NetAuth server, add an entity
and a group, and authenticate as that entity:

### Starting the server

Open a terminal and start the NetAuth server with the options as shown:

```bash
$ netauthd --PWN_ME \
 --jwt_rsa_generate \
 --jwt_rsa_privatekey token.key \
 --jwt_rsa_publickey token.pem \
 --make_bootstrap root:password
```

This will start your server and generate a set of keys that can be
used for issuing tokens.  The server is bootstrapped with an entity of
ID 'root' and authenticated with the secret 'password'.  An important
note to make here is that the ID 'root' will collide with the
administrative user on most Linux systems.  To combat this collision
if you want to use the ID 'root' it is recommended to instead use
'_root' which will not collide.  The log scroll should look something
like the following:

```text
2018/09/25 21:21:59 NetAuth server is starting!
2018/09/25 21:21:59 Server bound on localhost:8080
2018/09/25 21:21:59 ===================================================================
2018/09/25 21:21:59   WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING
2018/09/25 21:21:59 ===================================================================
2018/09/25 21:21:59
2018/09/25 21:21:59 Launching without TLS! Your passwords will be shipped in the clear!
2018/09/25 21:21:59 Seriously, the option is --PWN_ME for a reason, you're trusting the
2018/09/25 21:21:59 network fabric with your authentication information, and this is a
2018/09/25 21:21:59 bad idea.  Anyone on your local network can get passwords, tokens,
2018/09/25 21:21:59 and other secure information.  You should instead obtain a
2018/09/25 21:21:59 certificate and key and start the server with those.
2018/09/25 21:21:59
2018/09/25 21:21:59 ===================================================================
2018/09/25 21:21:59   WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING
2018/09/25 21:21:59 ===================================================================
2018/09/25 21:21:59 The following DB backends are registered:
2018/09/25 21:21:59   ProtoDB
2018/09/25 21:21:59 The following crypto implementations are registered:
2018/09/25 21:21:59   bcrypt
2018/09/25 21:21:59 The following token services are registered:
2018/09/25 21:21:59   jwt-rsa
2018/09/25 21:21:59 Initializing new Entity Tree with ProtoDB and bcrypt
2018/09/25 21:21:59 Initialized new Entity Manager
2018/09/25 21:21:59 Initializing token service
2018/09/25 21:21:59 Warning: No token implementation selected, using only registered option...
2018/09/25 21:21:59 Loading public key from token.pem
2018/09/25 21:21:59 Blob at token.pem contains no key!
2018/09/25 21:21:59 Generating keys
2018/09/25 21:22:00 Keys successfully generated
2018/09/25 21:22:00 Commencing Bootstrap
2018/09/25 21:22:00 No entity with ID 'root' exists!  Creating...
2018/09/25 21:22:04 Secret set for 'root'
2018/09/25 21:22:04 Created entity 'root'
2018/09/25 21:22:04 Set capability GLOBAL_ROOT on entity 'root'
2018/09/25 21:22:04 Bootstrap phase complete
2018/09/25 21:22:04 Disabling bootstrap
2018/09/25 21:22:04 Bootstrap disabled
2018/09/25 21:22:04 Ready to Serve...
```

### Using NetAuth

Now we'll do a few things with the server that's running.  To make
things easier, lets alias some flags for a new shell:

```shell
$ alias netauth="netauth --PWN_ME --jwt_rsa_publickey=token.pem --entity root"
```

Now we'll add an entity and get information on them:

```shell
$ netauth create-entity --ID foo --secret bar
Secret:
New entity created successfully
$ netauth entity-info --entity foo
ID: foo
Number: 2
```

Now lets add a group and get information on it:

```shell
$ netauth create-group --name myGroup --display_name "My First Group"
New group created successfully
$ netauth group-info --group myGroup
Name: myGroup
Display Name: My First Group
Number: 1
```

Now we will put foo into myGroup and see the membership shows the
entity foo as a member:

```shell
$ netauth entity-membership --entity foo --group myGroup --add
Membership updated successfully
$ netauth list-members --group myGroup
ID: foo
Number: 2
```

Finally we can authenticate as foo and inspect the resulting token:

```shell
$ netauth --entity foo get-token
Secret:
Token obtained
$ netauth --entity foo inspect-token
{foo [] 5}
```

You can see the token was obtained and the token contents dumped in
machine format.

### Wrapping Up

From the server's perspective, here's what happened:

```text
YYYY/MM/DD HH:MM:SS Token requested for root (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Successfully authenticated 'root'
YYYY/MM/DD HH:MM:SS Secret set for 'foo'
YYYY/MM/DD HH:MM:SS Created entity 'foo'
YYYY/MM/DD HH:MM:SS New entity 'foo' created by 'root' (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Info requested on 'foo' (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Allocated new group 'myGroup'
YYYY/MM/DD HH:MM:SS New Group 'myGroup' created by 'root' (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Information on myGroup requested (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Entity 'foo' added to 'myGroup' by 'root' (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Membership of 'myGroup' requested (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Token requested for foo (netauthctl@deepblue)
YYYY/MM/DD HH:MM:SS Successfully authenticated 'foo'
```

There's plenty more that NetAuth is capable of, this is just a 5
minute demo that shows basic usage.  For more information, read the
complete docs at [https://docs.netauth.org](https://docs.netauth.org).
