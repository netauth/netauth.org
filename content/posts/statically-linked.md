---
title: "Statically Linked"
date: 2017-10-10T22:51:29-07:00
---

An authentication service needs to work even when everything else is
broken.  This includes if the host its currently on is somewhat
broken.  Most importantly, this needs to be the case if the host that
runs the authentication service is itself deriving user accounts from
the network authentication database it is hosting.

I have learned this the hard way.

To that end, NetAuth needs to survive anything on the host dying.
This can be done very easily by statically linking in everything that
the server could possibly need so that it doesn't need other libraries
to work.  The only library that it should reasonably depend on will be
libc for system functions, but if that is broken on your systems
you've probably got much bigger problems anyway.

So statically linking is fantastically broken in a great many
languages, and flat impossible in others.  Statically linked C or C++
are always interesting adventures, especially when glibc is involved,
so this would almost need to be linked to musl or something along
those lines.  That's not particularly portable though since musl is
really geared torwards Linux and if one thing is certain, its that as
an open source project, someone will eventually try to run this on
plan9 or Windows or some other obscure system.

Lets use Golang then.  Go is a strongly typed, compiled language
designed by Ken Thomphson and Ian Lance Taylor.  It has excellent
cross compiling support and is statically linked by default.  It has a
decent selection of libraries and a developer culture of correctness
and not hacking things together.

Thus, go shall be the the language that NetAuth is developed in.
