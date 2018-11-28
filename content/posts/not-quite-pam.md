---
title: "Not Quite PAM"
date: 2018-11-27T22:45:47-08:00
---

NetAuth is reaching a fairly advanced level of development, and is now
a reasonable choice to run on networks managed by a skilled
administrator.  Before becoming generally available though, there's a
few more changes that need to be made.  Once such change is increasing
the flexibility of the server to be extended by each administrator
based on policy needs of each installation.

Such flexibility has been achieved through a complete overhaul of the
entity and group management subsystem.  Previously these subsystems
used statically defined procedures to create, retrieve, update, and
destroy the objects they managed.  This doesn't allow for onboard
flexibility unfortunately, and had to be changed.  The tree manager
now makes use of chains of hooks to perform these same actions.

Practically, this means that there is a new go package
`github.com/NetAuth/NetAuth/internal/tree/hooks` which maintains types
that satisfy the hook interface.  These hooks perform functions such
as validating that an entity exists, determining whether or not a
given number is available to be assigned, or ensuring that data has
been written back to the storage system after processing.

What's more, the composition of the chains is not static.  On
initialization, the tree manager will assemble all of its hook chains
based on a configuration of what hooks belong in what chain.  A future
release will enable hooks that are not compiled into the server to be
used, allowing for a previously impossible level of customization and
policy evaluation in the NetAuth server itself.

The hook system has another, less visible benefit.  It dramatically
improves the maintainability of the NetAuth server.  Rather than
needing to have a fake database available in every test and to perform
long tail tests of very complicated abstraction layers, the hooks can
be tested in isolation, with faults and failures injected with greater
ease.  This enables the server to be developed with greater confidence
while allowing greater development speed.

Of course it is important that with flexibility the interface promises
are upheld.  This is ensured by a suite of tests that validate that
interfaces exposed by the tree manager do what they are supposed to do
in the default configuration.  These seperate tests import the hooks,
configure full entity manager instances, and perform in some cases
complex chains of actions on the server.

It is important to note though that this testing can only go so far.
A bad hook installed by an administrator can still cause instability,
insecurity, or a failure to satisfy interface promises.  It is assumed
that a system administrator deploying NetAuth will carefully consider
any 3rd party hooks they wish to install, and the effect these hooks
may have on their installation.
