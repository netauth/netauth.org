---
title: "Tokens Please"
date: 2018-04-10T12:13:57-07:00
---

NetAuth is a service that provides secure information, therefore it
needs to be secure itself.  This has so far been done by sending
authenticating information in the form of an entityID and entitySecret
with every request from the CLI tools.  This has 2 main problems.

First, this is not the greatest in terms of security since it means
that the secret either has to be entered every time or held in memory
in the client.  Worse, the workaround here could be to put a secret in
a file and read it from disk.

Second, this is really expensive. Every time the server needs to check
identity using an ID/secret combination it needs to do a bunch of
lookups and then run the secret through bcrypt to check the hash.  As
was discussed previously, bcrypt is expensive by design and so this
isn't a good solution long term.

Enter tokens.

Since authentication is usually based on something you know, something
you have, or a combination of these, we really only need to make sure
the something you have isn't only your ID and secret.  The token can
be granted in exchange for the ID and secret, can be time limited, and
can be scoped for use with a particular thing.

NetAuth uses a pluggable token implementation which provides a
standard type (token.Claims) that includes things the token can claim
to be.  Since these tokens are verifiable by the server, they can be
used in place of an ID and secret to provide information about the
holder.

NetAuth's default token implementation uses the JSON web token
standard, which includes fields for when the token was issued, when it
expires, and what its good for, among other things.
