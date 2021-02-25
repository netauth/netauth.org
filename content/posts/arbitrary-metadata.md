---
title: "Better Arbitrary Metadata"
date: 2021-02-24T22:33:18-08:00
---

On entities and groups it is currently possible to store data that is
beyond NetAuth's core metadata concept.  This is to facilitate storing
small amounts of additional arbitrary metadata that may hold value to
other systems that are connected to NetAuth.

Over time, the initial implementation has proven to be inadequate
because it relies on string manipulation to seperate the key from the
value.  This breaks when you want to namespace your keys using colons,
which is a common schema with certain formats such as Oasis.

NetAuth 0.4 will ship with the new kv2 storage available which is able
to use arbitrary strings for keys with no risks of corruption.  Also
shipping is a new cleaner API to access the kv2 storage.  As this is
an interface that is not 1:1 backwards compatible, no automatic
migration is provided.  Be sure to check the patch notes for an
example script that will copy data from the legacy format to the kv2
format.

Also note that v0.4 will be the last release series to support the
original format.  Additional deprecation notices will be posted prior
to the 0.5 release cycle to ensure all users have been appropriately
warned to migrate data.

