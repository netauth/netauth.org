---
title: "What's in a Name?"
date: 2019-11-05T22:37:03-08:00
---

When you refer to a piece of software by name, do you ever think about
what that name really means?  Rarely is a name just a token to refer
to something, usually it is designed to make you feel a certain way or
think about the target in a specific light.  This is certainly the
case with NetAuth.

NetAuth is supposed to make you think about the first two A's in AAA.
That is, authentication and authorization.  This is the 'Auth' part of
the name.  The 'Net' part of the name should tell you where this
'Auth' comes from: the network.  Since network authentication is often
difficult and fiddly to implement, NetAuth aimed to be a quick answer
to a complex problem by being the shortened form of the very problem
people try to solve.

While this name is a great way to connote an idea with a solution, it
doesn't work so well in actually specifying the name in the software
itself.  NetAuth is written in Go, which has some expectations around
package names.  One of the biggest is that package names are lower
case.  This helps to type them since you don't have to reach for the
shift key, and it helps to make the name-space of all Go package names
uniform.  When NetAuth started, the idea was to keep the name
formatted the same in all locations, this turned out to be a mistake
and is being fixed now rather than later.  As the logrus project
learned the hard way, changing the name of a package is a very
difficult thing to do if left for too long.

Additionally, upper case names in Go packages can behave weirdly on
file-systems that are case-insensitive.  In these systems it is
possible to import a title-cased package as all lower case.  The code
will work as expected until it is moved to a host that uses a case
sensitive file-system, where it will break with non-obvious compiler
errors.  By standardizing on the lower case naming convention, this
problem is alleviated.

Starting with commit `710937e`, all references to paths in the source
code will use the fully lower cased format
`github.com/netauth/netauth`.  The proper way to refer to the service
though is still NetAuth with the title-case capitalization.  This
change also coincides with the release of the NetAuth2 protocol, and
should be the last major breaking change to the publicly provided
interface.
