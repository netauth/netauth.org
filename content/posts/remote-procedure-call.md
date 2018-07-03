---
title: "Remote Procedure Call"
date: 2017-10-11T23:04:46-07:00
---

Message Exchange
----------------

So a server written in Go and destined to provide authentication
information must be accessible by some way.  Otherwise it won't be
particularly useful.  We could use HTTP and REST requests to push and
pull JSON to the server, but this has a lot of problems.  For one it
requires a JSON parser for any language that wants to talk to NetAuth,
and it requires the server to understand JSON.  I'm not particularly
fond of JSON and its not the easiest interchange format to do things
with.

So if JSON is out, what other formats are available?  Well, there's
always XML, but as mentioned in the first post (Git Init) the goal is
to build something that feels like its from the Sun days, and XML does
not feel like that at all.

There's always the option of a totally custom format, or of a custom
binary format such as a binary packed string.  But these options are
frustrating to implement and take away from what we're building which
is an auth server.  If I wanted to build an interchange format I would
go do that seperately.  Go has excellent support for Protocol Buffers.
Protocol Buffers, or protobuf as they're more widely known, are
Google's data interchange format and support a few very useful things.

First, protobuf supports reading older message versions transparently.
Second, it supports serialization to a binary format for line
transmission efficiency.  For those that must have JSON, protobuf
encodes the required information to be able to get to JSON.  Perhaps
most importantly, protobuf can generate headers for a myriad of
languages so that the serialization and deserialization of the
messages is done consistently.

RPC
---

The messages shall be protobuf, but how shall they be transferred?  Go
has excellent web support, so they could be sent as the body of POST
requests to a remote server.  This would allow everything to run over
HTTP and pass over annoyingly oversensitive firewalls, but this isn't
really a clean way to do it.  Really, what we want to do is ask the
server to run some arbitrary function with some data that we have,
then return some more data back to us.  Basically we want to remotely
run functions on the server.

Fortunately, Sun had this figured out quite some time ago and built
NFS on it.  Remote Procedure Call, RPC, is a design paradigm where
the caller is on one system and through a well defined and agreed
upon standard will send a request to a server with some packed data
for execution, and will then get results back as a result from the
call.

Golang has an RPC system within the net/rpc package, but this requires
doing a lot of manual processing and futzing around with setting up a
server to handle the requests.  It would be much easier if there was
just a library that did this already so we could get on with the
business of writing an authentication server.

gRPC
----

Well that was easy.  We'll use gRPC.  gRPC is an RPC implementation
designed by Google and using protobuf as the interface defenition
language (IDL).  Unlike Sun's RPC though, gRPC uses HTTP/2 as the
transport.  For those not experienced with SunRPC, this is means that
the port number of the server is a fixed value.  In Sun's system, the
port was a randomly allocated value that you needed to query another
service to find.  In addition to being annoying, this made it
virtually impossible to write a clean and easy to understand firewall
ruleset for Sun RPC servers.  Using HTTP/2 as the transport also means
that we get the firewall crossing capabilities of a well understood
and deployed protocol for free!

Perhaps one of the more important things that gRPC buys though is that
since it generates stubs for the service like protobuf generates stubs
for the types, it can generate stubs for languages other than Go.
There is no intent right now to build a client library in anything
other than Go, but the fact that it is available as an option is
important and should not be overlooked.
