---
title: "Log Level"
date: 2019-09-01T23:44:01-07:00
---

Logging is a difficult problem to solve satisfactorily.  For some
users there will be far too much log output, and for others there
won't be enough.  Recently with the work on plugins it has become
clear that there is insufficient logging in NetAuth.

This realization has led to all the logging infrastructure being
replaced.  The log of choice is Hashicorp's `hclog` which provides
levelled logging, and provides logging that can cross the boundaries
of plugins.  This logging replacement work is now complete, and will
ship in the v0.2.0 release with the rest of plugins.
