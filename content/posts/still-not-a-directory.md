---
title: "Still Not a Directory"
date: 2018-10-27T20:37:37-07:00
---

All the way back at the start of the project, it was decided that
NetAuth would not be a general purpose directory.  First, because
general purpose directories are difficult to develop and maintain, and
second because that wasn't the goal of the project.  If you want a
directory, go install OpenLDAP.  This being said, some directory
functions are really useful to have.  NetAuth now has a key/value
store which adds just enough "directory" to be useful.

Starting in version 0.0.10, NetAuth includes a key/value store on
entities and groups.  The semantics of the store are the same for both
entities and groups, so we'll only look at entities in this write-up.

A new command has been added to the CLI:

```shell
$ netauth help untyped-meta
untyped-meta --<entity|group> --<read|clearfuzzy|clearexact|upsert> --key <key> --value [value]

Manage untyped metadata associated with groups and entities.
Clearfuzzy strips ordering values from keys before clearing, whereas
clearexact removes a key with an exact match.  Upsert either inserts a
new value, or updates an existing one, and tries to do the right thing
either way.  Key and Value are always strings, key may not contain the
reserved rune ':'.
  -clear-exact
          Clear keys with exact matching
  -clear-fuzzy
          Clear keys with partial matching
  -entity string
          ID for the entity to modify
  -group string
          Name for the group to modify
  -key string
          Key to act on
  -read
          Read the value specified by 'key' including ordered results
  -upsert
          Insert/Update a value for a particular key
  -value string
          Value to act with
```

The new command is modal, and can read, update, insert, and clear keys
from the store.

An example of basic usage is to set keys on an entity as shown:

```shell
$ netauth untyped-meta --entity root --upsert --key k --value foo
$ netauth untyped-meta --entity root --read --key '*'
k: foo
```

Pretty simple, and the '*' works as expected and returns all the keys
and values (just make sure to quote it so your shell doesn't eat it).

At this point, its worth talking about the caveats and restrictions to
this system.  First, the k/v structure is part of the embedded data
model on entities and groups, if you store thousands of keys on
something, its object will become very large and cause the access
times to spiral upwards.

Second, all keys and values are strings, all the time.  If you link
the native client or use the gRPC definition directly, then when you
get the results from the server, you will obtain a map of strings to
strings.  If you require strict typing, consider using an encoding
scheme of some kind.

Sometimes ordering multiple values for a single key is very useful.
To support this NetAuth supports using indexed keys which are always
sorted and returned by the CLI as a sorted set.

Here's an example of the ordered keys system:

```shell
$ netauth untyped-meta --entity root --upsert --key phone{0} --value "1 (555) 867-5309"
$ netauth untyped-meta --entity root --upsert --key phone{2} --value "1 (555) 090-0461"
$ netauth untyped-meta --entity root --upsert --key phone{1} --value "1 (555) 888-8888"
$ netauth untyped-meta --entity root --read --key phone
phone{0}: 1 (555) 867-5309
phone{1}: 1 (555) 888-8888
phone{2}: 1 (555) 090-0461
```

As you can see, even though the keys are added out of order, they are
returned as rules by an ordering constant at the end of the key.  This
expression is not part of the key, but is instead used to order the
keys within a return.

The key/value system should allow NetAuth to bridge the gap a bit more
easily between being a raw IDP and a full fledged directory.  Just
remember that the additional values live within the data models for
groups and entities, and adding large numbers can cause your server to
slow down.
