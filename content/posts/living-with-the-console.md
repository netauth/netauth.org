---
title: "Living With the Console"
date: 2019-02-18T22:11:59-08:00
---

It is important when designing software to live with it for a while to
determine if its functional or not.  Many organizations call this
"dogfooding" and use this valuable feedback to avert potential
disasters in releasing software to the public.  NetAuth has recently
averted one such disaster in the CLI tooling.

Over a year ago in ["Command Not Found"](../command-not-found/) it was
decided that NetAuth would use subcommands under a single utility.
This worked well for a year and is indeed still how NetAuth structures
its commands.  The primary change is to no longer use the limiting
library that started out NetAuth.  To understand why, lets look at two
different commands to create an entity.

```
$ netauth create-entity --ID foo --secret bar
```

This command is fairly short to type, but requires the use of flags
for items that will always be specified.  Lets take a look at another
command:

```
$ netauth entity-memberships --entity foo --group group1 --add
```

Again, this command works, and once you read the manual it does what
you would expect, but its not easy to remember.  Is it membership,
memberships, or members?  Why am I modifying the entity memberships
and not adding them to a group?

On top of this, these commands are only as good as the documentation
which lives in another repository.  These simple facts conspire to
make a complicated world of CLI tooling.  Its much easier to think
about things with positional arguments in a log of NetAuth's commands
as that makes it much easier to remember the flow of commands for
commonly used items.  Take for example alternate versions of the two
commands above:

```
$ netauth entity create foo --initial-secret bar
$ netauth entity membership foo add group1
```

In this case its much clearer that we're creating an entity named
'foo' and adding it to a group with the name 'group1'.  These commands
are easier to remember.

It goes one step better though, with the change to a new library for
command handling that understands positional arguments, the
documentation for comands now lives with the commands themselves, and
so is more likely to be correctly updated when a command changes its
contents.

Commands additionally now have bash and zsh completions, and can use
the same configuration file that the server uses to pick up many
default options.

While its a small nuisance to break CLI compatibility, it is sometimes
necessary in order to make progress.  The NetAuth team hopes you'll
find the new CLI easier to use and faster to memorize.
