---
title: "Locked Out"
date: 2018-10-31T22:08:37-07:00
---

Sometimes you get a ping along the lines of "kill this user's access
NOW".  The obvious answer is that for things that important physical
security should be dispatched and potentially even partitioning the
network around someone is a good idea.  For the less extreme cases,
such as a lost password, entity locks will suffice.

Entity locks are an easy way to prevent an entity from successfully
authenticating, even with the correct secret.  The lock is a simple
toggle on all entities that when set causes calls to ValidateSecret to
fail.

The locking mechanism will be released with version v0.0.11.
