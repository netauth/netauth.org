---
title: "NetAuth"
date: 2018-09-25T20:24:08-07:00
---

[![Build Status](https://travis-ci.org/NetAuth/NetAuth.svg?branch=master)](https://travis-ci.org/NetAuth/NetAuth)
[![GoDoc](https://godocs.io/github.com/NetAuth/NetAuth?status.svg)](https://godocs.io/github.com/NetAuth/NetAuth)
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

NetAuth is provided pre-compiled on the release page for each release
on GitHub after 0.1.0.  Running versions prior to 0.1.0 in production
is discouraged.

If you would prefer to compile NetAuth instead, you can compile it
with Go 1.17 or better:

```shell
$ git clone -b <version> https://github.com/netauth/netauth
$ cd netauth
$ go build -o netauthd github.com/netauth/netauth/cmd/netauthd/
$ go build -o netauth github.com/netauth/netauth/cmd/netauth/
$ go build -o nsutil github.com/netauth/netauth/cmd/nsutil/
```

This will produce a pair of executable files `netauth` and `netauthd`.
Per convention, `netauthd` is the server executable.  The third
utility, `nsutil` is a server utility that performs certain
bootstrapping tasks.  You may copy these to where your system keeps
user installed applications for secure use, usually `/usr/local/sbin`.

You can also obtain both the server and command line client in Docker
container format from the GitHub Container Registry.  The containers
may be pulled as shown below:

```
$ docker pull ghcr.io/netauth/netauth:v0.6.1
$ docker pull ghcr.io/netauth/netauthd:v0.6.1
```

To make the containerized versions behave like local binaries, the
following aliases may be employed (adjust the version as needed):

```
$ alias netauth="docker run --rm -it -v $PWD/config.toml:/etc/netauth/config.toml -e USER=admin --network host ghcr.io/netauth/netauth:v0.6.1
$ alias netauthd="docker run --rm -it -v $PWD/config.toml:/etc/netauth/config.toml --network host ghcr.io/netauth/netauthd:v0.6.1"
```

Note that some things don't work normally in the docker containers:

  * User detection doesn't work, you need to set your default user
    explicitly by exporting the USER environment variable.
  * Prompts that manipulate the terminal control registers, such as
    secret prompts, don't work and you must pass secrets via explicit
    flags.

Due to these shortcomings the CLI container is provided mostly for
demos and for the rare case where a container is the only format you
can source the tools from.  In all other environments you should
source the CLI from a package or an unencapsulated binary.  The server
on the other hand is fine to run from a container.

## Configuration

Configuration is really best left to the manual linked above, but for
a quickstart there's two important things to note:

A config file for this demo should be created with the following content:

```toml
[core]
  home = "data"
[crypto]
  backend = "bcrypt"
[db]
  backend = "filesystem"
```

### Certificates

Remember that NetAuth is dealing with secure information.  This
information must be protected while in transit, and NetAuth uses TLS
to ensure this security.  For this quick demo we can use a helper
function from `nsutil` to generate some certificates, but for
production use you should create and manage your certificates with
some more comprehensive certificate management infrastructure.

To create the certificates, run the following commands:

```shell
$ mkdir -p data keys
$ nsutil certgen localhost
$ mv tls.* keys
```

You can pass an arbitrary number of IP addresses and hostnames to the
`nsutil certgen` command and they will be included in the generated
certificate.

## 5 Minute Demo

Lets do a 5 minute demo where we start a NetAuth server, add an entity
and a group, and authenticate as that entity:

### Create the Initial Admin User

To create the initial admin user we will use `nsutil bootstrap` to
create the first user and provide them with the root capability
required to manage the server.

```shell
$ nsutil bootstrap admin
```

You will be prompted for a password to set for the user.  An important
note to make here is that the ID 'root' will collide with the
administrative user on most Linux systems.  To combat this collision
if you want to use the ID 'root' it is recommended to instead use
'_root' which will not collide.  In general though, a user ID that is
not `root` is generally a better choice.

### Generate Signing Keys

You will need to create some keys to use with the token issuing
subsystem.  To generate these keys with `nsutil`, use the following
commands:

```shell
$ nsutil keygen rsa 2048
$ mv rsa.privkey keys/rsa-private.tokenkey
$ mv rsa.pubkey keys/rsa-public.tokenkey
```

### Starting the server

Open a terminal and start the NetAuth server with the options as shown:

```bash
$ netauthd
```

This will start the server and make it ready for serving requests.
The log will look something like this:

```text
2022-06-02T23:10:58.282-0500 [INFO]  netauthd: NetAuth server is starting!
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.crypto: Registered Backend: backend=bcrypt
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.db: Registered KV Store: kv=bitcask
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.db: Registered KV Store: kv=filesystem
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.db: Database callback registered: callback=BleveSearch
2022-06-02T23:10:58.282-0500 [INFO]  netauthd: Database initialized: backend=filesystem
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.crypto: Initializing backend: backend=bcrypt
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.db: Database callback registered: callback=entity-resolver
2022-06-02T23:10:58.282-0500 [INFO]  netauthd.db: Database callback registered: callback=group-resolver
2022-06-02T23:10:58.283-0500 [INFO]  netauthd: Token backend successfully initialized: backend=jwt-rsa
2022-06-02T23:10:58.284-0500 [INFO]  netauthd: Ready to Serve...
```

We can confirm the claim made in the last line by invoking the system
ping command on the CLI:

```shell
$ netauth system ping
Pong
```

While 'Pong' is the response that's expected, its also not the most
helpful.  More output can be obtained by asking for the system's
status:

```shell
$ netauth system status
Server Status: [PASS]
```

### Using NetAuth

Now we'll do a few things with the server that's running.  As long as
you run from the same directory that contains yoru config.toml, this
demo will work.  Otherwise you will encounter errors that your server
does not support TLS, and that your server cannot be located.

To make the following commands shorter, we will export the
`NETAUTH_ENTITY` environment variable so that NetAuth knows we want to
make requests as 'admin'.

First we can check our authentication information:

```shell
$ netauth auth check
Secret:
$ echo $?
0
```

Now that we know our credentials work, lets get a token for future
requests:

```shell
$ netauth auth get-token
Token obtained
```

The token contains some interesting information, so lets take a look
at it:

```shell
$ netauth auth inspect-token
This token was issued to 'admin'
 Capabilities:
   - GLOBAL_ROOT
```

Here you can see that the token contains the `GLOBAL_ROOT` capability
which is what allows us to do interesting things while on the server.


Right now though, the server only has a single entity present, lets
add another one and then inspect it:

```shell
$ netauth entity create entity1
Initial Secret for entity1:
Entity Created

$ netauth entity info entity1
ID: entity1
Number: 2
```

Now lets add a group and get information on it:

```shell
$ netauth group create group1 --display-name "My First Group"
Group Created

$ netauth group info group1
Name: group1
Display Name: My First Group
Number: 1
```

Now we will put entity1 into group1 and see the membership shows the
entity myuser as a member:

```shell
$ netauth entity membership entity1 add group1
Membership Updated

$ netauth group members group1
ID: entity1
Number: 2
```

Finally we can authenticate as myuser and inspect the resulting token:

```shell
$ netauth --entity entity1 auth get-token
Secret:
Token obtained

$ netauth --entity entity1 auth inspect-token
This token was issued to 'entity1'
 Capabilities:
```

### Wrapping Up

From the server's perspective, here's what happened:

```text
2022-06-02T23:15:17.956-0500 [INFO]  netauthd: NetAuth server is starting!
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.crypto: Registered Backend: backend=bcrypt
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.db: Registered KV Store: kv=bitcask
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.db: Registered KV Store: kv=filesystem
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.db: Database callback registered: callback=BleveSearch
2022-06-02T23:15:17.956-0500 [INFO]  netauthd: Database initialized: backend=filesystem
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.crypto: Initializing backend: backend=bcrypt
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.db: Database callback registered: callback=entity-resolver
2022-06-02T23:15:17.956-0500 [INFO]  netauthd.db: Database callback registered: callback=group-resolver
2022-06-02T23:15:17.957-0500 [INFO]  netauthd: Token backend successfully initialized: backend=jwt-rsa
2022-06-02T23:15:17.958-0500 [INFO]  netauthd: Ready to Serve...
2022-06-02T23:15:35.252-0500 [INFO]  netauthd: Authentication Succeeded: entity=admin service=netauth client=theGibson
2022-06-02T23:15:35.255-0500 [INFO]  netauthd: Token Issued: entity=admin capabilities=[0] service=netauth client=theGibson
2022-06-02T23:17:52.123-0500 [INFO]  netauthd: Entity Created: entity=entity1 authority= service=netauth client=theGibson error=<nil>
2022-06-02T23:18:01.122-0500 [INFO]  netauthd: Dumped Entity Info: entity=entity1 service=netauth client=theGibson
2022-06-02T23:18:15.453-0500 [INFO]  netauthd: Group Created: group=group1 authority= service=netauth client=theGibson error=<nil>
2022-06-02T23:18:24.502-0500 [INFO]  netauthd: Group Info: group=group1 service=netauth client=theGibson error=<nil>
2022-06-02T23:19:01.082-0500 [INFO]  netauthd: Authentication Succeeded: entity=entity1 service=netauth client=theGibson
2022-06-02T23:19:01.087-0500 [INFO]  netauthd: Token Issued: entity=entity1 capabilities=[] service=netauth client=theGibson
```

There's plenty more that NetAuth is capable of, this is just a 5
minute demo that shows basic usage.  For more information, read the
complete docs at [https://docs.netauth.org](https://docs.netauth.org).
