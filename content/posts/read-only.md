---
title: "Read Only"
date: 2019-07-27T00:16:23-07:00
---

Just one server running with all your data on it is a good way to
signal that you don't care about that data.  This kind of setup is
really only suitable for lab networks and demonstration environments,
not at all for production use.

Up until now, if you wanted to run 2 NetAuth servers it was
technically possible, but only if they were on a shared filesystem.
This would effectively allow either server to make changes to data
that was part of shared state.  This shared state would then be
instantly visible to the other server.  There are a few problems with
this approach.

First, it assumes that you have the ability to make a shared
filesystem available to both servers, which may not be possible or
practical for any number of reasons.  In environments where there is
not a shared filesystem and instead a one way replication job from a
master server to many slave servers (similar to how Kerberos KDC
replication works) then changes made to slave servers will appear to
revert themselves.  Second, it ignores that the default search
implementation relies on holding the entire search index in memory.
This makes the search very fast, but means that if you swap out files
under the server, the contents of the index will be out of date very
quickly.

The first problem has a clear solution.  There should be a way to tell
a server that it is a read-only replica member.  This setting should
then allow the server to refuse mutations with a PreconditionFailed
message, but continue serving read-only traffic.  This change should
be read from a configuration file, but should also be done in such a
way that it can be changed at runtime, to support intelligent server
clustering in the future.  The config token is as follows:

```
[server]
readonly = true
```

Just like that you can have read only replica servers!  Unfortunately
its also not quite that simple.  Making this work means that there now
needs to be a way to specify the server that should be contacted for
write requests.  To this end a new token has been added to the core
configuration stanza:

```
[core]
master="some-server.example.com"
```

The server specified by master will be used for write requests, while
the server name will continue to be used for read requests, unless an
override is specifically requested to contact the master.  If the
master server is unset, it defaults to the same as the value for
read-only traffic.

As to the second problem, databases that make use of the blevesearch
mechanism will need to be patched to keep an eye on resources and
re-index them if they are changed while the server is running.
