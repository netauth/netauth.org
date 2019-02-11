---
title: "Config Files"
date: 2019-01-04T21:49:53-08:00
---

Up until now the primary way to configure NetAuth has been flags
specified to any binary that requires configuration.  Sure, there's
some level of reading in a "config file" with the client library, but
this is a poor facsimile for an actual configuration system.  This
will change with the next release.

The next release will feature configuration courtesy of the Viper
configuration framework.  This framework parses configuration in
various formats and then puts it into a global singleton for later
access.  The changes to pull in configuration files don't amount to an
interesting diff, but the reasoning behind why its taken this long to
get config files do.

When NetAuth started, it started with heavy inspiration of run-time
systems that have no underlying file-system, at least not a POSIX one.
In these systems you can't have a configuration file, you have to read
your config from some kind of API, or pass it in with flags.  On the
surface, this is a really great setup.  It unifies the run-time
information of an application, and it means that you're unlikely to
change the deployment in a way that will break the application.  Since
then, a few key points have arisen around NetAuth.

## Too Many Tunables

One of the things I've seen most administrators have the hardest time
with are tunable values in an application.  Flags are the most obvious
kind of tunnable input, but others exist as well.  If you want to see
the logical conclusion of what happens when there are too many tunable
values, look no further than your nearest Database Administrator.  The
entire profession that DBA's take part in exists because there are
just too many things to be tuned in a database server.  While this is
not a truly fair comparison as tuning databases is a difficult and
application specific process, the fact that it exists at all asks
deeper questions about application design.

Adding tunables to NetAuth would just be begging people to "tune"
them.  In many cases the opinionated defaults are the most prudent
choice to take, and given that NetAuth is critical security
infrastructure, its not a good idea to invite people to make arbitrary
changes.

## Core Infrastructure

One of the biggest reasons to add a traditional configuration file
comes from the logical conclusion that NetAuth represents core
infrastructure at the bottom of the stack.  While there are a number
of places that run their authentication system under the control of a
massive job controller, they also have to contend with the problems
that this creates in terms of authority loops.  An authority loop is
where less than the authentication service's root authority allows you
to gain root on any machine where it is maintaining a state database.
In an ideal world, core NetAuth servers would always be dedicated
physical machines in racks that have separate keys and would only ever
be accessed by persons who hold the `GLOBAL_ROOT` capability.  This is
impractical for a number of reasons, and so the best one can hope for
is a minimal VM.

This requirement to have nothing else really around the NetAuth server
neatly dovetails into the idea that infrastructure is a tree.

## Directed Acyclic Graphs

There are a number of companies and organizations out there that have
learned this one the hard way.  They've got fantastic loops in
infrastructure both production and corporate.  If you want to find
whether your company has these kinds of problems, you can ask what
would happen if everything were to reboot at once.  If you can't
reboot it all at once, you've got a cycle somewhere, and that's very
very bad.

NetAuth can have no cycles if it is to be at the core layers of the
infrastructure of anything more than a garage based lab network, and
so it needs to have everything for startup close at hand.  This means
in practice that it can't depend on some kind of configuration service
to load files from, or a fancy API to load the configuration data from
either.  Doing either of these things would potentially create startup
and authentication loops, and is not permissible.

---

As a result of these realizations, NetAuth now has a config file that
configures not only the core service daemon, but also the libraries
that previously needed to read another file to pick up target
information.  The default format for these files and only supported
format is TOML, but the content should be valid in any of Viper's
parse-able formats.
