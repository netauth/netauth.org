---
title: "Keys and Values"
date: 2020-11-26T21:44:35-08:00
---

Storage is a funny thing.  Its something that seems so simple, but
also so complicated.  You want to store some bytes, so you put them on
a disk.  Want to read them now?  No problem, just read them back.  Oh,
you wanted to read them back from another computer?  That's now a bit
harder.

NetAuth's storage architecture was previously based on the idea of
thick databases.  These 'databases' were really a set of largely
duplicated functions that serialized and deserialized entities and
groups, plumbed search indexes, and callbacks between them to enable
the system some concept of "high availability" by watching for changes
across layers.  This makes it unnecessarily difficult to add new
storage mechanisms, and results in lots of copy and paste when doing
refactoring.  It also meant that the tests for most functionality
needed to make use of an in-memory database implementation which
doesn't always have the same failure modes as the current default
mechanism `protodb`.

In effect though, all storage mechanisms for NetAuth are just
key-value systems, so treating that as a distinct abstraction will
allow simplicity both in upkeep of existing implementations, and in
construction of new storage backends.  The key/value store needs only
to be concerned with reading and writing byte arrays to and from byte
keys.

The new storage mechanism will ship in NetAuth version 0.4 alongside
the enhanced entity and group metadata system.

