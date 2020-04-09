---
title: "On the Localizer"
date: 2020-04-08T23:35:20-07:00
---

One of NetAuth's primary goals has always been authentication and
identity for Linux servers.  For many servers, this means PAM and NSS.
While these are both widespread they are not particularly easy to
interact with.

For example, NSS has no concept of a failure during processing.  This
largely precludes on-line lookups, and even in the offline case the
lookup path needs to be bulletproof.  Bugs in NSS handlers can quickly
turn into security incidents by nature of the information that NSS
handles.  PAM isn't much better, depending on being able to load
shared objects into the target's address space.  This is in stark
contrast to the `bsdauth` system where a program is executed in a
highly restricted context and its return code checked for whether
authentication succeeded.

Over time, its become desirable to support systems where NSS and PAM
are not available.  Where can one find such systems you ask?  The very
popular Alpine Linux ships with this configuration.  There is no NSS
available because that is simply not a part of the musl design
specification, and there is no PAM because it is often unnecessary.

NetAuth will solve this with a pair of new projects.  The first one,
the localizer, will replace the entire NSS integration path for
standard purposes.  It will be able to directly modify the `passwd`,
`group`, and `shadow` files to update users that the system is aware
of.  Additional planned functionality beyond the initial release
includes writing `autofs` mount maps, and even a plugin to the NetAuth
server which would permit synchronizing `shadow` hashes directly to
targeted machines in environments where PAM is unacceptable.

In environments where PAM is acceptable, the existing pam_netauth will
be replaced with a shim executable that can be consumed by
`pam_exec.so`.  This will allow cleaner integration and unit testing
by being able to drop CGO compilation options.  It will also
dramatically simplify the story of running a PAM integration on
non-GLibc systems, where Go's shared object compilation is not
supported.

Look forward to both of these new components in the coming months.
The original NSS integration will be maintained on a best effort basis
moving forwards, and the original pam_netauth will be retired entirely
due to its limited support and security critical function.
