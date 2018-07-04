---
title: "Bigger on the Inside"
date: 2018-05-25T13:43:09-07:00
---

NetAuth uses a somewhat unique group system, so lets take a look at
how this works in detail.

In most directory systems, there are groups.  These groups directly
contain members and the members can be listed to have membership in
these groups.  In some directory systems, its possible to nest groups.
This usually has the effect of saying 'foo has these members, plus all
the members of subgroup bar'.  If you want to find out who is in foo,
you should also request the membership of bar.  Similarly, if you are
in group bar, you may not have an easy way to find out that you are
also a member of foo indirectly.  NetAuth uses a different style of
expansion to this.

Direct Memberships
------------------

Direct  memberships work  exactly like  the  group file.   You add  an
entity  to  a group  and  they're  a member  of  it.   This is  likely
sufficient for 90% of all use  cases that most people will care about,
and as such  is the more optimized case.  Memberships  are stored as a
list of groups on each entity, and are queried with extreme speed.

This type of membership would be equivalent to the NIS group map.
Unfortunately, this style of mapping has two key problems.  First, it
requires the client to resolve more things.  As explained above if you
want to find out who is a member of 'foo' you must also know who is a
member of 'bar' and add these members to the list you got from 'foo'.
Second, this type of membership system necessitates a standalone
service to the side of the directory for doing policy based grouping.
For example disabling an account either needs a field on the entity to
disable it, or the consultation of a second group to determine if it
is disabled.

Include Expansions
------------------

The first type of expansion that NetAuth supports is very
straightforward.  You can tell a group to include another group.  This
directly includes the other group's memberships and is transparent to
a client looking up members.  The difference between this system and
the nested groups described above is that the client is not aware an
expansion took place.  The client just asked for the members of 'foo'
and got all of them, even those who have indirect membership in 'foo'
via 'bar'.

These exapansions can be nested arbitrarily deep and can include
arbitrarily many groups at each inclusion level, though for sanity of
the systems administrator, care should be taken not to create overly
complex membership graphs.

Exclude Expansions
------------------

Exclude expansions are the second type of expansion that NetAuth
supports and are a bit more involved to understand.  These expansions
work by 'pruning' members of a group if they are also members of a
subgroup.  Take for example the directory structure shown below:

```
ALL
 |
 |-group1
 |   |
 |   \-foo
 |
 \-group2
     |
     \-foo
```

If you were to query the membership of group1 with no expansion rules,
then as you would expect the members list would contain 'foo'.

```
ALL
 |
 |-group1
 |   |
 |   |-EXCLUDE:group2
 |   \-foo
 |
 \-group2
     |
     \-foo
```

Now if we add an exclude expansion to group1 to exclude the members of
group2 and run the same request, something interesting happens: group1
has no members anymore!  Ok, to clear up what's happening here, lets
modify the directory a little bit so that it now looks like this:

```
ALL
 |
 |-group1
 |   |
 |   |-EXCLUDE:group2
 |   |-foo
 |   \-bar
 |
 \-group2
     |
     \-foo
```

Now if we query the membership of group1 again we have one member
'bar'.  What is happening here?  When the exclude relation was added
to group1 this instructed NetAuth to filter out anyone who was a
member of group2 when returning the membership of group1.  But, and
this is the important bit, the members are still in group1.

```
ALL
 |
 |-group1
 |   |
 |   |-foo
 |   \-bar
 |
 \-group2
     |
     \-foo
```

If we remove the exclude rule and query the membership of group1 again
it now contains 'bar' *and* 'foo'.  The 'foo' entity was never
removed, so they still maintain membership.  Here's a real-world
example that might spark thoughts of how this can be used:

```
ALL
 |
 |-PermitLogin
 |   |
 |   |-INCLUDE:Users
 |   \-EXCLUDE:Restricted
 |
 |-Users
 |   |
 |   |-Alice
 |   |-Bob
 |   |-Eve
 |   |-<Many More Entities>
 |
 \-Restricted
     |
     \-Eve
```

In this example, we have entities that are permitted to log in to a
system as members of the PermitLogin group via an include rule
targeting the Users group (all users are permitted to log in).  Not
surprisingly Eve was caught trying to install a keylogger on Bob's
computer, so Eve is restricted from logging in now.  But this is
temporary as we expect Eve will learn from this experience, so we
don't really want to remove Eve from the Users group, since that's
going to require noting down somewhere and some rules to put back
later.

Instead we can have another group called 'Restricted' and this group's
members shall be excluded from PermitLogin.  By putting Eve in this
group, PermitLogin no longer will return Eve as a member since the
EXCLUDE rule has filtered her out.

Expansions All The Way Down
---------------------------

You can actually nest includes and excludes as deep as you like to
compose arbitrarily complex logical membership graphs based on the
include and exclude rules.  Do be careful though, as each rule has a
non-trivial cost to calculate and the mental cost of maintaining a
large tree like this is also quite high.

