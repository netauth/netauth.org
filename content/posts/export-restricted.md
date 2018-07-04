---
title: "Export Restricted"
date: 2018-02-01T20:21:54-07:00
---

Crypto is hard, but its also really important for an authentication
system.  Lets look at why.

It seems like every week there's news of some new breach or some new
exploit and now some company's password database is leaked to the
internet.  Be it the colossal disaster that was 000Webhost's plaintext
password database, or Adobe's improperly implemented hashing, everyone
seems to find some way to screw up storing passwords.  Lets look at
how passwords can be stored, and how NetAuth stores them.

There are generally two ways to store password databases.  The first
is to encrypt the passwords in such a way that the cipher text is
useless without the decryption key, the other is to hash the password
such that it cannot be recovered from the stored hash.  Hashing sounds
pretty good since it means the password can't be recovered once
hashed, and you might ask yourself why this isn't the only way of
doing things.  After all, why would the password system need to get
your password back in plain text if it can just hash your attempt and
then compare the hashes?

Some systems, such as Kerberos, use the password to sign or encrypt a
token that then needs to be decrypted again.  In the Kerberos system,
this would be the Ticket Granting Ticket, which because it is
encrypted and sent to you locally means that your password never
leaves your workstation.  This is pretty slick, but does mean that the
Kerberos database if compromised could be used to obtain passwords in
plaintext.  Another case where you might want to get the passwords
back out is a password manager such as LastPass or pass(1).  These
systems as a function of storing passwords for other systems need to
be able to return the password as plain text so that it can be fed
into the other system as though a user had typed it themselves.

NetAuth is not a password manager though, so it doesn't need to get
information back out again.  Similarly, the passwords are sent over a
secure channel to the NetAuth server, so there is no need to sign or
encrypt things with an entity secret.  Because the secrets don't need
to be recoverable, we can hash them.

NetAuth's default crypto plugin uses the bcrypt algorithm to hash the
secret and then stores the resultant hash.  The original secret is
never stored or written to disk, so there should be no way to recover
it once the secret has been set.  The bcrypt algorithm was chosen
because Go has good library support for it, and bcrypt is what's
called an "adaptive" hashing algorithm.  This means that you can
change the difficulty to compute a hash to make it less easy to break
the hash.

The interface is of course a plugin based system so other crypto
systems can be swapped in in the future if desired.
