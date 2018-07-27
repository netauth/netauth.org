---
title: "Inconsistency Detected"
date: 2018-07-26T20:56:03-07:00
---

One of the most underappreciated professions is that of the Technical
Writer.  These are the amazing people that generate the documentation
that comes along with high quality software.  They're also, as writers
tend to be, eagle eyed for inconsistencies in the design of textual
interfaces.

I have the fortune of knowing a handful of technical writers, and
recently I received some feedback on NetAuth that the CLI tools have
inconsistent interfaces.  Some things are plural, some things are case
sensitive, some things aren't, and some things can fail in obscure
ways if you don't specify exactly the right combination of attributes.

This is unacceptable.

Clearly then the solution is a fix-it release that will break certain
interface assumptions and fix these things.  Fixing things though
requires some agreement on the end state.  Therefore, it is being laid
down here for how the CLI interfaces will change.  The RPC interfaces
will remain stable, as those are reasonably consistent, but the CLI
interface shall be cleaned up.

## Entities and Groups

Initially, the intent has been that entities had IDs and groups had
Names.  This is carried through the code internally and is a nice
distinction to make there, but the way most people speak about things
is to say "entity foo" or "group bar" which makes sense to reflect in
the tooling.  Thus, across all CLI tools the flags shall be `--entity`
and `--group` to specify an entity and a group.  This is slightly more
characters to type, but it is clearer as to what the flag is for.

## Multi-Mode Options

Currently, there are a handful of commands that take options of the
form `--mode <mode>` or `--action <action>`.  Take for example the
mode selector that is on the group expansion command.  It has options
`INCLUDE`, `EXCLUDE`, and `DROP`.  These options will become separate
flags: `--INCLUDE`, `--EXCLUDE`, and `--DROP`, with appropriate
interlocking checks.

## Case in Option Parameters

This one is easy, all parameters that are fed into commands for
purposes of field printing shall be case insensitive.

## Command Naming

This is one that should have never been an issue, yet if you look at
the output of the commands help screen (abridged):

```
Subcommands for Entity Administration:
    new-entity       Add a new entity to the server
    remove-entity    Add a remove entity to the server
                        
Subcommands for Group Administration:
    new-group        Add a new group to the server
    delete-group     Delete a group existing on the server.
```

There's agreement that new should be for adding things, but not how to
remove them.  Both of these shall be changed to match the naming
elsewhere in the system, the verb 'create' shall be used for additive
state changes, and the word 'destroy' shall be used for destructive
state changes.
