---
title: "Command Not Found"
date: 2017-12-14T19:59:24-07:00
---

NetAuth is primarily managed by the command line.  Eventually there
might be a web client that can do certain tasks, but that will be
still a ways off.  The question then becomes how to structure the
management interface.

If we look back at other systems, there's a few options.  There's the
command shell of something like kadmin, there's the multiple commands
that make up the ldap suite, and there's a few other systems that use
one binary with multiple shell commands under it.

Command shells are a really cool idea, they give you this ability to
open a console which has special builtins for things that the service
can do and things you want to manipulate.  These shells though are not
particularly convenient in my experience.  Just about the only time I
think I've used this functionality was setting someone's shell to
kadmin so that they could do basic management tasks without needing to
understand the system they were connecting to.  Command shells are
also annoyingly complicated to get right since they are after all
shells.

Ok, so if we don't use a command shell then one can simply call a
binary and it will do something, and the being a shell business will
be left to an actuall shell such as bash or zsh.  This is all good and
well, but now the question is one binary or many?

In the many binaries approach you wind up with something like the ldap
toolset which comprises binaries to add, remove, modify, delete and a
few other specialized actions.  In practice I think I've only ever
used ldapmodify but that's as a result of having an emacs major mode
that would flush the buffer into ldapmodify.  This approach has the
advantage of being easily understood and battle tested.  The shadow
tools work this way, the NIS tools work this way, and even some of
Microsoft's own tooling for AD works this way.

In practice this means that you tend to have a bunch of commands that
all start with the same thing.  These wind up being things like
ldapadd, ldapmodify, ldapdelete and so on.  This works really well to
show what namespace these tools belong to, they all do things with
ldap.  Similarly the NIS tools all start with 'yp' to show they belong
to the same functional set.  What if instead of 'ldapmodify' though we
had 'ldap modify'.  Its a subtle difference, but an important one.  In
the second case, you have a single base command, in this case 'ldap'
which then provides subcommands that perform specific actions.  This
allows you to share a large amount of code in the single binary that
might be difficult to pull off otherwise in the case of many binaries.

This is the option that NetAuth uses.  NetAuth has a single control
binary called netauthctl which exposes subcommands for doing different
things.  In the help output, this shows up as a bunch of groups of
commands around logical tasks.  The netauthctl binary takes its own
top level options, but then each subcommand can also take options that
are relavant to it.  Adding new commands is also easy, its just a
matter of registering a struct with methods that satisfy an interface,
which amounts to about 2 lines of code to register, and around 50
lines of code for each command.

NetAuth's commands live in the internal/ctl package.
