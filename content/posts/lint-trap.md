---
title: "Lint Trap"
date: 2018-07-05T17:26:20-07:00
---

Most programming languages have linters.  Go is no exception.

Up until now, most of NetAuth has been in the rapid development phase
and code quality has not been kept as high as should be in a release
system.  The linter is one way to improve things.

Changes that have happened since the last release that will make their
way into 0.0.8 or perhaps 0.1.0 involve cleaning up the linter
concerns for all of the packages that are a part of NetAuth itself, as
well as seeking code reviews.  These code reviews have brought forth
valuable feedback for where the code can be made to be more in line
with the Go best practices, as well as places where refactoring passes
had missed sections of the codebase.  One such case dates back to the
very early versions of NetAuth which used dual keyed entities to make
lookups via number faster in some cases.  This had been pulled from
the entity system for a while, but the database still claimed to
support this function.

While the authors of golint make very clear that you should not
blindly strive for a lint-free codebase, I find that all of the
suggestions it made for NetAuth were reasonable and worth
implementing.  NetAuth is now lint free, heavily commented, and
hopefully much easier to extend and service.
