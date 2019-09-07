---
title: "Plugged In"
date: 2019-09-06T21:01:29-07:00
---

NetAuth has from the very start been opinionated.  This is what has
allowed the system to work reliably and be developed quickly even with
extremely limited developer resources.  Opinionated systems are,
however, limited by the opinions that have shaped them.  In NetAuth's
case, this is visible in what the primary server does and doesn't do.
It can tell you if an entity is who they claim to be, and what groups
they are in.  This is all, and this is by design.

Of course, this doesn't and won't cover all use cases from all sites
that may wish to run NetAuth.  It was never intended to.  To cover
these additional use cases and to provide the logic in the server that
some sites may require requires extensibility and for NetAuth,
extensibility means plugins.

So what can plugins extend?  Well, right now plugins can hook into 12
key points during entity and group processing.  In order, these points
are:

  * Entity Creation
  * Entity Updates
  * Entity Lock
  * Entity Unlock
  * Entity Destruction
  * Group Creation
  * Group Updates
  * Group Destruction
  * Before secret changes
  * After secret changes
  * Before authentication
  * After successful authentication


These hooks are means to provide extensibility for both policy choices
such as site specific password requirements, as well as mirroring data
from NetAuth into external systems.  Plugins are only useful though if
they're reasonably easy to build, so lets look at what it takes to
build a plugin.

Plugins were designed to be very easy to implement.  To ensure this is
the case plugins are implemented as simple Go interfaces which can be
accessed via Hashicorp's go-plugin library.  This mechanims of plugin
components ensures that each plugin runs as a seperate supervised
process with its own protected memory that can't crash the main server
should it encounter a problem.  To install a new plugin, simply drop
its file into the appropriate directory and restart the server, the
plugin will automatically be added to the list to be invoked during
request procesing.

Plugins aren't without risk though, a plugin that crashes or returns
errors will abort the request that's being processed, so care should
be taken to only load tested and proven plugins on critical
infrastructure.  In the worst case a plugin can be removed temporarily
if its acting up and the server can be run as normal without any
plugins installed.

So this is a lot of how plugins are implemented at a high level, but
what does one actually look like?  One example plugin is part of the
NetAuth codebase to show how the basic idea works: `fail2lock`.  This
plugin is very simple, it implements the "if there are X failures in Y
minutes, refuse authentication" scheme.  The plugin consists of a
`PreAuthCheck` hook which checks if too many recent failures have
occured, and if so it refuses the authentication attempt.  If not, it
pessimistically assumes that the login will fail and adds a record
documenting the failure.  If the login succeeds the `PostAuthCheck`
will be called and the temporary failure markers will be cleared.

In practice, this looks like the following two functions:

```
func (f fail2lock) PreAuthCheck(e, de pb.Entity) (pb.Entity, error) {
	flags := util.PatchKeyValueSlice(e.Meta.UntypedMeta, "READ", "fail2lock", "")

	if len(flags) == 1 && strings.Split(flags[0], ":")[1] == "RESET" {
		delete(f.failDB, e.GetID())
		e.Meta.UntypedMeta = util.PatchKeyValueSlice(e.Meta.UntypedMeta, "CLEARFUZZY", "fail2lock", "")
		return e, nil
	}

	inIntervalFails := 0
	startTime := time.Now().Add(cfg.GetDuration("interval") * -1)
	for _, t := range f.failDB[e.GetID()] {
		if t.After(startTime) {
			inIntervalFails++
		}
	}

	if inIntervalFails >= cfg.GetInt("allowed_fails") {
		appLogger.Warn("fail2lock is locking an entity",
			"entity", e.GetID(),
			"fails", inIntervalFails,
			"allowed", cfg.GetInt("allowed_fails"))
		e.Meta.Locked = proto.Bool(true)
		return e, nil
	}

	f.failDB[e.GetID()] = append(f.failDB[e.GetID()], time.Now())

	return e, nil
}

func (f fail2lock) PostAuthCheck(e, de pb.Entity) (pb.Entity, error) {
	delete(f.failDB, e.GetID())
	return e, nil
}
```

As you can see, the meat of the plugin is actually very simple, even
to have a special flag that lets a help-desk operator instantly clear
failures from the plugin's memory.  This plugin only implements two of
the available hooks though, so how can it satisfy the interface?  Very
simply actually, it embeds the null plugin:

```
type fail2lock struct {
	tree.NullPlugin

	failDB map[string][]time.Time
}
```

This "null" plugin implements all functions and has no affects on
calls to other hooks, it simply returns without errors for all cases.
In this way plugin authors are free to focus on only the code they
require to generate their plugin's behavior.  All told, the example
plugin is only 77 lines of code.

This functionality as well as structured logging from the previous
post will be released in 0.2.0 along with updates to the getting
started guide to showcase this new functionality.
