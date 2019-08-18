---
title: "Non Renewable"
date: 2019-08-16T23:38:54-07:00
---

NetAuth makes use of tokens to authenticate many actions.  These
tokens are by default implemented as RSA signed JWT's.  This is a
reliable way of making sure that NetAuth can maintain a lightweight
identity to authorize multiple sequential calls.

In the early days of NetAuth there was also the idea that these tokens
could be passed to other services in exchange for service specific
identity markers.  This turned out to have 2 key flaws, and has been
removed.  The first key flaw with this idea is that the token can be
exchanged.  Tokens represent an internal interface component, and
while they can be reliably parsed with software that has been linked
against the NetAuth client libraries, the tokens themselves are an
unspecified internal interface.  This missing stability guarantee
limits the ability to exchange tokens to other services, since these
services may not be able to parse them.

The second problem with tokens is that they grant too much power back
into NetAuth.  By encouraging other systems to accept tokens, it
increases the risk that one of those systems being compromised could
lead to a compromise of the NetAuth system itself.  Since most
services simply want to perform username/password authentication and
then maintain their own internal tokens, trying to use NetAuth's
tokens on top of this doesn't make sense.

Another key problem with tokens was renewability.  Renewability adds
in lots of questions about the lifetimes of security artifacts, and
makes it much easier for an attacker to move slowly and be careful,
since they can simply renew a token.  Placing a limit on the maximum
lifetime helps, but it still complicates the token issuing logic and
provides another avenue for attack.

Given that tokens are no longer provided as a means of authenticating
to other services, the renewal functionality has been completely
removed.  To further improve the security afforded by the tokens
themselves, the default JWT lifetime has been reduced from 10 hours to
10 minutes.

These changes will be live in release v0.1.2.
