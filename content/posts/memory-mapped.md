---
title: "Memory Mapped"
date: 2021-07-01T21:08:30-07:00
---

NetAuth uses a key/value system for storage.  This allows the system
to piggyback off of highly performant implementations of this concept
via pluggable backends.  One such implementation that is being
introduced in the v0.5.0 release is a bitcask memory mapped storage
option.

Bitcask comes with a few limitations, it is only suitable for
operation on a single host, and really should be run from a local
filesystem.  It also can be damaged if you do not shut down your
systems cleanly, so always make an effort to make sure you smoothly
shutdown your servers when using the bitcask storage engine.

To enable the bitcask storage engine simply set the `backend` value in
the `db` section of your configuration file to `bitcask`.  If you have
data already in another backend you can use a new tool to migrate the
data.  The brand new `nsutil` (NetAuth Server Utility) can perform
offline tasks for your data storage option.  Importantly it must be
run only when the server is stopped.  To copy from one backend to
another, you can use the tool as shown below:

```
$ nsutil copy <source> <target>
```

To copy from the pre-0.5.0 default (`filesystem`) to the new `bitcask`
backend the command would look like this:

```
$ nsutil copy filesystem bitcask
```

In large deployments this should show a dramatic speed improvement,
and should also improve speed during system startup when in-memory
indexes are generated.  The `filesystem` backend remains supported at
this time, and will not be removed in favor of the `bitcask` backend.
The new `bitcask` backend is not available on Windows builds.

