---
title: "Acid Proof"
date: 2018-01-22T20:37:01-07:00
---

A server that purports to be a repository of information isn't much
good if the server can't survive a reboot.  So clearly now that
NetAuth has some data worth storing, it should be stored somewhere.

Databases are usually implemented as a service that is connected to
via the network, and for good reason.  Taking MySQL for example, the
server needs to have some means to manage multiple tables of
information, to handle changes to rules surrounding this information,
to authenticate and authorize readers and writers and more.  It is no
wonder than that MySQL does this as a separate binary and that
Database Administration is a respected profession.

One of the biggest frustrations though for me as a systems engineer is
when something needs some external dependency when it really
shouldn't.  Databases are a good example of this.  They're a fine
thing and solve many problems that would otherwise have to be
implemented in a local system, but they're also extra overhead to run
and almost universally a pain to install, manage, and upgrade.  Some
of the best applications I know provide database plugins which allow
you to use one of many different options, and usually SQLite is one of
these options.  Its a single-file SQL database written in C and it
works quite well.  This though assumes that we're in C when we aren't.

The plugin thing is a good idea though, so we'll use that and just
build a plugin that works for now, and if we need higher performance
then that's just another plugin down the road.

So what is the database doing?  What do we need it to do?  Well, it
needs to be able to store entities and groups.  This is effectively
two seperate tables.  For simplicity we don't really need or want the
fancy SQL rules for deletes or other expansions involving the keyword
CASCADE.  Really we just want something that allows us to send it
objects and get them back later.  A cache would also be nice.  Oh, and
since this is an authentication system the data we send should be
stored securely and only visible to the NetAuth server itself (and a
backup process, but we'll ignore that for now).  Ideally this storage
platform should also have checksums that allow you to recover data in
the event of corruption like an unclean shutdown.

This really sounds like a filesystem.

Since all the data that's in use in NetAuth is just protobuf, why not
just write the protobufs to the disk?  This is a straightforward
solution that should work fairly well, it will permit quick and easy
operation out of the box and if needed data could always be migrated
later to a more capable storage system.

Thus NetAuth shall have a default storage engine that just writes the
protos to disk.  This engine is called ProtoDB, and with a reasonably
fast disk, or an SSD, is probably very usable in installations with a
modest modification rate.
