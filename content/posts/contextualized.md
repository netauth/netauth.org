---
title: "Contextualized"
date: 2021-09-21T20:38:40-06:00
---

Go is a language based around developer experience and tooling
quality.  This doesn't come for free though, some concepts such as
tracing and profiling depend on having cross-service continuity.  In
Go this context is passed in a dedicated type, the `context.Context`
type.  This type is conventionally passed as the first argument to any
function that requires it, and is threaded all the way from incoming
requests all the way to outgoing RPC calls made by NetAuth to other
services such as storage components.

In the next major version of NetAuth one of the major under the hood
changes that will happen is the plumbing of contexts all the way from
incoming RPCs to the bottom of the storage tier.
