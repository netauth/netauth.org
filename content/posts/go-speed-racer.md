---
title: "Go Speed Racer"
date: 2019-08-23T20:43:09-07:00
---

NetAuth makes certain assumptions around the data it works with.
These assumptions are based on the knowledge of how directory and
authentication systems are used, and this deserves some discussion.
In this post we'll look at the types of data that are stored in
NetAuth, the access patterns, and the choices and trade-offs that have
been made.

First and foremost, lets talk about data.  The data that NetAuth
expects to serve is mostly static.  The change horizon for information
such as name, number, home directory, and similar has a change
interval of years, if ever.  This isn't to say it never changes, but
we can make certain assumptions that the information isn't going to
change and use this to our advantage.  Specifically, we can serve
reads without any locking, as its very unlikely that the information
will have changed on disk in a time-frame that matters to any consumer
system.

Other information has similarly slow timelines for changes.
Information such as group membership and group expansions is expected
to change very infrequently.  Group expansions specifically are really
only expected to change once, when they are initially setup.

Some information is allowed to change more frequently, and is expected
to as well.  Information such as entity secrets are expected to change
on a 30 day horizon, though this is of course dependent on
organizational policy and politics.  Some entity metadata as well may
change much more frequently, such as last successful login, or the list
of SSH keys, though organizations are encouraged to use SSH
certificates instead of keys.


This leads to a bug discovered by GitHub user m-wynn.  The bug,
present in versions prior to 0.1.4, makes NetAuth vulnerable to
persisted data corruption in environments with excessively high update
rates to entities and groups.  Without going into deep technical
details, it was previously possible that two different threads within
the server would attempt to write to the same file at the same time,
leading to garbage data being written to the file.

The solution to this problem is simple and involved only a small
change to the PDB library in order to ensure that file writes are
atomic.  This still leaves the risk of two simultaneous writes
fighting with each other, as the last write will "win" the race and
overwrite the other one.  Given the assumptions that are made around
the data in the system, this risk has been deemed acceptable to the
project since there isn't a good way around it without introducing
locks at the RPC layer.
