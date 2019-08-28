---
title: "Comparison With Keybase SSH CA"
date: 2019-08-27T17:52:49-07:00
---

With the launch of [Keybase SSH
CA](https://keybase.io/blog/keybase-ssh-ca) several days ago, some
questions have come in about what the differences are between NetAuth
and Keybase SSH.  Both services provide a means of managing keys that
is better than copying them around by hand.  Both services are easier
to setup than massive corporate SSO platforms.  However, there are
many differences between them.  This article tries to explain what the
differences are, and when you might use each solution.

Lets start out with a brief overview of what Keybase SSH is.  It is,
as you might guess from the name, an SSH CA provided by and using the
Keybase infrastructure.  What is an SSH CA you might ask?  An SSH CA
is a Certificate Authority that can be used to secure communications
over SSH.  The idea is that rather than using an SSH key which has no
expiration date and is non-trivial to expire from a number of hosts to
use a certificate instead.  Certificates can be expired with the use
of a CRL, they can be time bound to very short intervals, and the
infrastructure that operates a CA is well understood and battle
tested.

Keybase issues the certificates with the use of a custom CA which an
end user would run on a (hopefully) dedicated machine in a very secure
location.  This machine has to be secure, because if it is compromised
then the security of the underlying CA must also be assumed to be
compromised.  Once the CA is known to be breached, it is good practice
to assume that anything else it was securing has been breached, even
if no evidence of this can be produced right away.  Its far better to
be safe than sorry in this case.

Keybase is a very impressive system, and it is the author's opinion
that more organizations should be using SSH certificates than
currently are.  Keybase and NetAuth though are different mechanisms,
so lets now look at what NetAuth provides to handle your SSH keys.

NetAuth doesn't attempt to get into the CA business, and it doesn't
attempt to issue your keys or otherwise provide you a means of
generating new ones.  What it does do is store standard SSH keys and
provide a way to get them back.  Through the use of the
AuthorizedKeysCommand it is possible to retrieve keys from a remote
store to use with a local `sshd`.  NetAuth provides this remote store.
Now that we know what each one does and roughly how it does it, lets
look at how these systems are different.

The biggest difference is what your actual login looks like.  Since
Keybase is only providing a certificate for a team level, everyone on
the team is logged on as a single user to the machine.  From the
machine's perspective, it is impossible to tell who has logged in, or
if multiple people were logged in simultaneously, as they will all be
authenticated as the same user.  You can of course review the CA's
logs to determine who requested a particular certificate for a
particular machine at a particular time, but this is not knowledge
that would be instantly available to a host in the same way as calling
`getpwnam` on a logged in user.

In contrast, NetAuth provides an identity to go with the key, so with
an appropriate NSS configuration, each login is tied to a specific
entity.  This can provide far more granular access than just logging
in as a particular team, but the utility of this granularity is
somewhat environment specific.  This is an advantage in environments
where it is necessary to know at the time of login who is logging in.

Another area the systems differ is how a key is revoked or expired.
For the purposes of this article, both cases will be treated as a key
replacement, with the case of removing a key being treated as
replacement with an empty key.  In the Keybase case, the key will
expire after a limited amount of time, and a new key must be obtained.
This time limit is what makes SSH CAs so useful: you can ensure that
after a key is no longer in use, it is truly dead.

In the NetAuth case, keys can be removed at any time, and they will
disappear within a few hundred milliseconds from the keys available to
client machines.  For replicated NetAuth servers this time also
depends on the replication lag, but it is still within a few
milliseconds after the replication has completed.  Since the keys are
provided to the end system via the AuthorizedKeysCommand directive,
and best practices dictate that users using this mechanism should not
be able to install keys via any other means, it can be assured that
once a key has been removed from the store of usable keys, no copies
of it remain available for use.

A final point that is primarily of interest to organizations: you can
run NetAuth entirely within your security perimeter.  This may not
sound like much, but its worth remembering that while you can audit
the code that provides the SSH CA functionality, you cannot audit the
closed-source servers that Keybase uses to provide the access control
to that CA.  While the central Keybase server hasn't yet been shown to
have bugs, in certain regulatory regimes the lack of visibility into
the server makes it difficult to consume.  When this is combined with
the fact that the SSH CA currently only supports group level
visibility, not individual visibility (which it can't without an NSS
module or some user provisioning system), the Keybase CA seems to be
targeted at hobbyist users or very small organizations.

Hopefully this has shed some light on what is different between the
Keybase SSH CA and NetAuth's own key management solution.  If you have
questions or want to discuss this article, feel free to join the
discussion in `#netauth` on freenode.
