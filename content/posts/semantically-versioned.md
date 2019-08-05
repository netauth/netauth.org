---
title: "Semantically Versioned"
date: 2019-08-05T11:24:01-07:00
---

With commit `08729d8` the ROADMAP file has been updated to show all
future releases on a slightly different versioning scheme than before.
This is to support the use of tools that enforce semantic versioning,
and to correctly show that NetAuth is no longer considered in alpha.
The leading 0 continues to signify that the software is still in beta
and may change at any time, but multiple organizations are now running
NetAuth successfully in production, and its time to recognize that in
the version number.

As alluded to above, this change was prompted by some tooling.  The
project is experimenting with the use of
[GoReleaser](https://goreleaser.com) to provide pre-built binaries and
nicer release messages on GitHub.
