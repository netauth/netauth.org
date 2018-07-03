---
title: "Git Init"
date: 2017-09-03T21:32:24-07:00
---

Lets build a directory server.  No really, I'm serious.  A service
that will provide directory information to interested applications and
can also authenticate the entities it contains.

Why do this?  After all LDAP is already a thing.  Well if you want to
use LDAP, you have a few options.  You can either use LDAP as both
your authentication (can you bind) and your authorization (what data
is stored on you) source, but it really wasn't designed to work this
way.  In fact most production installations I'm aware of that use LDAP
pair it with MIT Kerberos.  Kerberos is a fantastic authentication
system from MIT's Athena project that is cryptographically secure and
accepted in a staggering number of places.  But its just an
authentication service, so it needs something to go with it to provide
the information about what you can do once authenticated.

But these two services are still very well paired, so why try to do
better?  To understand why this is a worthy endeavor, I invite all
challengers to stand up a replicated OpenLDAP server as the backend to
a multi-master Kerberos KDC, then shove a thousand or more extremely
needy end users in it and live with it for a year.  Come back here
once you've shared my pain and understand that as good as these
technologies are, the only real management frontends for them are
either home-grown for specific sites, Microsoft's AD offering, or
whatever proprietary thing RedHat's cooked up this week.  Because
these frontends only really work if you want to use all the rest of
the accompanying technology and are, at best "high-magic" solutions,
they don't fit well in the portfolio of easily manageable stacks.

Systems Engineers of a certain vintage will recall the tooling from
SunOS and later Solaris.  Prior to the Oracle takeover, Sun systems
had this certain feel to them in terms of quality and reliability.
Even better the documentation quality backed up the engineering
quality to make it a great system to run.  Sun made a run at a
directory service, it was NIS and later NIS+.  NIS was a great
solution but was fundamentally flawed because the network isn't nearly
as trustworthy as it may seem.

NetAuth will be an authentication and authorization system.  It will
learn from the tooling failure that is ldapmodify and kadmin, and it
will learn from the security failures of NIS.  It should have the feel
of quality tooling and be easy to operate and maintain.  Most
importantly, it should be easy to deploy and port data into and out
of.

Perhaps most importantly, NetAuth is to be generic.  Rather than
trying to be a service geared at UNIX hosts or an OpenID Connect
system it will be generic and possible to use as a backing store for
daemons that implement these concrete use cases.

So, lets build a directory server.
