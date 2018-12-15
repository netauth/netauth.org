---
title: "Search Mode"
date: 2018-12-14T20:27:01-08:00
---

Sometimes you might know what you want to find, but not the ID of the
entity, or the Name of the group.  That's where the search mode comes
in.  This new capability is implemented by default by the
[bleve](https://blevesearch.com) project.

The search indexes are updated on any insert of an object or an update
to an existing one.  Search temporarily breaks protodb clustering
capabilities, but this will be resolved by the next release.  The
practical upshot of the change though is that the way is now clear for
an RPC to search the NetAuth server.  Actually adding this search API
though will have to wait until some other cleanup work is done first.
