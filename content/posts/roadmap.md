---
title: "Roadmap"
date: 2017-10-02T22:18:53-07:00
---

What will the NetAuth server contain, and when will it contain it.
These are important questions to ask and set expectations for what
will exist at each version. So lets do that now:

Version 0.0.1:
-------------

This is going to be all about the MVP (Minimum Viable Product).  So
for this to work, there needs to be a way to add and remove entities
and to authenticate them using some secret information.  There should
also be some limited meta-data available here, such as internal
permissions for who can create and remove things on the server itself.
There should probably also be some way to see what's going on in the
server, so we'll want the ability to list entities.  Might as well
implement that as basic grouping support.

Version 0.0.2:
--------------

Now its time to start adding some features.  First off, the superuser
from the first version should now be able to change secrets for
unprivileged entities.  Also a good idea to support some real metadata
now.

NetAuth won't go as far as OpenLDAP in terms of data flexibility, but
it will need to store some metadata about entities in the system.
This will be accomplished as a section within the entity structure for
metadata and it needs to be written to.  Since we've got a superuser
concept, they can edit this as well as the entity its written on.

Oh, and this version should probably stop storing everything in
memory, so a way to write things to the disk would be good.  This
should be a pluggable system so that in the future when people
complain that its not good they can write a plugin that implements
whatever storage system will make them go away^W^W happy.

Version 0.0.3:
--------------

Groups should do more in this release.  They should first of all exist
as more than just a single group that contains everything.  Entities
should be movable to put into and take out of groups, and you should
be able to find out what group an entity is in.

Cluster support should come in as well.  After all it won't do much
good to build an authentication source that can't be replicated.  Just
try explaining to the boss that the network is down because your
single, non-redundant authentication server had a problem.  Clustering
is quite hard though, so in this version we'll cheat and just make the
data storage layer safe to sync between multiple servers.

Version 0.0.4:
--------------

Groups should do *more*.  Groups should be nestable.  Not nestable
though in the sense that returning a group in a client shows you it
contains groups though, that makes for complicated clients.  The
groups should do the expansions server-side so that if group foo
includes group bar, then foo contains bar's members directly in the
response to a membership query.

If we can include groups, lets also make it possible to exclude
memberships based on groups.  Thus if you have membership in foo and
in bar, but foo excludes bar, then you don't show up in the membership
of foo even though you have it as a direct group membership.

Version 0.0.5:
--------------

Doing everything with a global superuser doesn't scale, so that should
be fixed.  We already have the capability system that's used to check
if a user is the superuser.  Lets add lots more capabilities to this
and use those to give finer grained control on actions within the
server.  This will let the systems administrator create service users
for things like password reset that have limited permissions.

By the time we make it this far, there should be something releasable.
That being said, don't be surprised if more versions crop up between
here and 0.1.0.
