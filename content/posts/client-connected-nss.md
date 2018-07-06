---
title: "Client Connected: NSS"
date: 2018-06-23T22:26:58-07:00
---

No this isn't Mozilla's NSS.  That one is Network Security Services
and is in browsers and other things.  This NSS is the Name Service
Switch from the GNU C library.

Having a PAM implementation isn't very useful if you can't actually
identify the person logging in.  That's where the Name Service comes
in.  On a POSIX system this normally plugs into the files
/etc/{passwd,group,shadow} or less often, a local BerkelyDB containing
the same information.

If you want to implement a service that NSS can consult to locate
accounts that aren't local on the system, you need to build a shared
object that the C library will load at runtime to run the query.  If
you're on alternative libc's this might be loaded by [an external
daemon](https://github.com/pikhq/musl-nscd) but the fact remains that
its a shared object loading and doing the query.

There are a few problems with this, most notably that its another
shared object and marshaling data back and forth to Go is quite a lot
of effort for something that should otherwise be quite simple if it
was done natively in C.  Its also a pain because the NSS API is one of
the most convoluted and difficult to understand, which is why other C
libraries don't implement it.  The other problem is that the NSS API
expects no failures and has to be blazingly fast.  The number of
operations on a system that require the directory to be consulted is
staggering, and if the network is slow, then all those operations slow
down as well.  Since NetAuth is designed to be operable over the
Internet at large where network latency is assured, this is a
non-starter.  An NSS plugin could of course implement a local cache
and that would make things quite fast after the cache warmed up, this
kind of hit-or-miss cache is tricky to get right.

Wouldn't it be easy if we could just shove all the account data into a
set of files alongside the normal ones and look data up from there?
Well, it turns out a few engineers at Google had the same thought and
developed [libnss-cache](https://github.com/google/libnss-cache).
This library implements the interface that NSS expects and consults a
set of caches and separate indexes on the disk.  All NetAuth needs
then is a daemon that can generate these files.  The system
administrator then need only install the libnss-cache library and
setup a crontab to refresh the caches on disk.  This also has the
benefit of continuing to work in the event of a network partition.

Such a daemon is [nsscache](https://github.com/NetAuth/nsscache).
This daemon consults the NetAuth server and does writes out the
various maps that the system expects to see.  This also has the
benefit of making complex expansion rules virtually free for *nix
hosts since the expansions are precomputed.
