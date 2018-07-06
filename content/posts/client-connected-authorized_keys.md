---
title: "Client Connected: authorized_keys"
date: 2018-06-07T18:35:27-07:00
---

An authentication server is no use without some sort of client to
obtain information from it.  NetKeys is a client to NetAuth that can
retrieve keys for any daemon that wishes to obtain the public keys for
an entity, but most usually this would be sshd.

To pull keys from NetAuth to put into sshd the following two lines
need to be added to sshd_config:

```
AuthorizedKeysCommandUser <some_dedicated_local_user>
AuthorizedKeysCommand /path/to/NetKeys --ID %u
```

An important consideration though is that the keys are not cached
locally.  If there is a network parttion between the end host and the
NetAuth server then the keys will not be available.

The code for NetKeys lives at
[GitHub](https://github.com/NetAuth/NetKeys).
