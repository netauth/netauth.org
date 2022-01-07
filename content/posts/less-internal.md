---
title: "Less Internal"
date: 2022-01-06T23:46:52-06:00
---

There's a stance in Open Source that can be summarized as "No is
temporary, Yes is forever."  The intent of this statement is to convey
that feature requests or changes may be vetoed due to the current
understanding of support and complexity, but that no is a temporary
decision.  Once the change is made, the interface is published, and
the release is made, it is almost impossible to retract the features
that were released.

In Go, there is a language enforced way to make an interface internal,
simply include the word internal in the path!  This makes it so that
no packages outside of the one hat defines the code can import the
internal components.  This is a major boon for making unstable
interfaces stay restricted away from external consumers and the need
to support them.  Unfortunately, its a very much binary distinction
and there's no escape hatch, so there's no way for packages that are
part of the same project, but originating with a different package
path to import the restricted contents.

Within NetAuth, many components are part of `internal/` paths, but the
most annoying one to deal with as a consumer is the `token` package
which handles NetAuth's internal authentication tokens.  Having this
package restricted is part of why no web interface exists for NetAuth
at the time of this release.  With the package internal, its almost
impossible to write a good interface which hides and shows buttons
appropriately.

With the next release, the `token` package has moved to the `pkg/`
path, making it available to other consumers.  Similarly, the `cache`
components that were previously part of the top level `pkg/netauth`
package have moved to `pkg/token/cache` to more accurately reflect
their function and relation to the token system.  This change should
be completely transparent to end users, but will make new integrations
easier to build, easier to maintain, and easier to debug.

