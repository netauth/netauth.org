---
title: "Boolean Algebra"
date: 2021-05-10T22:15:13-07:00
---

NetAuth has a powerful and almost unique group rules system which
allows you to build arbitrary relationships between groups.  This is
discussed in detail in [this post](/posts/bigger-on-the-inside/).
What is great about this is that the logic of handling groups is
entirely on the server, when a client asks for the membership of a
group they receive the complete result set without needing to perform
additional local processing.  What is less great is that the server
needs to handle some pretty expensive operations to compute the result
set.

How expensive?  Well in the implementation that has shipped since the
early days of the NetAuth project until now, computing the members of
any single group involves computing the membership of all groups and
filtering for the one you're interested in.  This involves loading
each group multiple times recursively and expanding the entire
membership graph.  As you might imagine, this is okay for small
numbers of groups, but quickly grows out of hand.  To combat this, it
would be possible to cache the results, but this quickly runs into a
problem of invalidating large amounts of cache whenever relatively
small changes are made.

Instead, the membership resolver that will be included in the next
NetAuth release takes a different approach.  By caching the direct
memberships from every entity and computing an algebraic expression
for every group it is possible to quickly filter which group satisfy a
particular set of direct memberships of an entity and which entities
have direct memberships requisite to satisfy the expression of any
group.  This allows for speedy filtering, and when an entity changes
its direct memberships only one record needs to be updated.  Similarly
when a group changes its rules only the sub-graphs that reference it
must be updated.

This improvement in speed and efficiency is a great win for
maintainability to with critical path test coverage of the resolver
reaching 100% coverage.  Additionally, significant amounts of code
were able to be removed, simplifying the overall architecture in an
already incredibly dense component of the server.

The improved group resolver is slated to ship in version 0.5.0.
