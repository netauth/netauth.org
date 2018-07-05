---
title: "Permission Denied"
date: 2018-05-28T14:33:16-07:00
---

NetAuth is providing authentication and authorization as a service, but
what provides the authorization info to NetAuth for internal actions?
Clearly the server can authenticate random users no problem, since
after all is has the information to do that on behalf of other
clients.

Internal authorization is done in a few ways, either via admin groups
or via capabilities.

Admin Groups
------------

Admin groups are, as the name implies, groups with administrative
powers.  They don't have power over the entire server, just over
specific groups.  The idea is that you can have an admin group and
point to it from other groups and say "the membership of this group
can be managed by that admin group over there" and those entities will
be allowed to add and remove other entities from this group.

Admin groups can additionally modify all other metadata on a group
including its expansions, the displayed name, and other similar
fields.

Capabilities
------------

Capabilities are the other system of authorization within NetAuth.
Capabilities gate the access to do broad actions on the server and
should not be conferred without careful consideration of the security
implications.  Capabilities grant the ability to do things like
creating and removing objects on the server, changing entity secrets,
and even granting more capabilities.

Capabilities can be assigned to either entities directly or groups,
though preference should be to assign to a group.  This is so that
when an entity is added to a role as defined by a group, they gain the
appropriate powers associated with that role.  Extreme caution should
be taken when using expansion rules on groups that have capabilities
assigned to them as this can lead to everyone in the server suddenly
gaining root authority.

Capabilities are granular and are checked for individually on the
actions that require them, with one exception.  There is a top level
capability that is called "GLOBAL_ROOT" which short-circuits the check
logic and will always authorize any action.  The "GLOBAL_ROOT" power
is also required to be able to assign other capabilities.  Since this
creates a sort of chicken-and-egg problem, the server itself can
assign this capability when launched with special options.
