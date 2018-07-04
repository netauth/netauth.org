---
title: "Plugin"
date: 2018-01-25T20:55:48-07:00
---

Plugins are a convenient way to cheat certain problems by providing a
low power implementation of something and replacing it later.  NetAuth
is no different and uses plugins for many things.

Currently there are plugins in the storage layer and soon in the
cryptography layer.  These allow the implementations to be swapped out
later should they be deemed to be unsatisfactory.

Plugin is a bit of an overloaded term though, so its worth clearing up
what is meant by it here, especially when its being used in the
concept of statically linked binaries.

In this case I'm referring to implementations that are baked in and
are used to provide functionality in a switchable way.  All of the
baked in plugins use the same interface and can be swapped out at any
time, they just have to have been compiled into the server when it was
built.  This may seem strange, but ultimately is because the Go
language doesn't support dynamic loading of Go libraries.  They're
either compiled in or they're not available.
