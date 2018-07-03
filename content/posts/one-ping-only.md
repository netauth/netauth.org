---
title: "One Ping Only"
date: 2017-10-14T23:34:22-07:00
---

Before going into the mess of authentication and data that needs to be
read from and written to potentially complex data-stores, it would be
nice to be able to just ping the server and know that its okay.

Thinking ahead, this would probably also be something nice for an
admin that got a page at 3 in the morning that something is wrong.

Pinging the server is really simple, we'll just send a ping request to
the server, and it should send back a reply of whether or not its
healthy, and a string with further information.

On the server we'll do this as a module that has an internal value
that is updated atomically and protected by a mutex, and then have a
set of methods that can set this value to true or false based on if
the server is healthy.  The idea here is that its really hard to tell
if the server is good, but its easy to tell if something bad has
happened that's unrecoverable.  Once an unrecoverable error has
occured, the system should be able to expose that not only in the
logs, but in the ping response.

In the future this will likely be fleshed out to a set of system
specific health checks, but for now a single global status will be
more than enough to keep things going.
