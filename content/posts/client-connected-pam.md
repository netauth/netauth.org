---
title: "Client Connected: PAM"
date: 2018-06-11T22:19:52-07:00
---

Pulling SSH keys is neat and all, but its much more impressive to be
hooked into the PAM stack.  That's where pam_netauth comes in.  This
is a client that implements the NetAuth protocol in a PAM plugin for
use on Linux and Darwin systems.

The module is written in Go and uses the `-buildmode=c-shared`
mechanism to generate `pam_netauth.so` for use with the PAM stack.
The module implements the 'auth' handler only as the management of
credentials is expected to take place outside the local system via
either `netauthctl` or via a site specific IDP interface.

The code for pam_netauth lives at
[GitHub](https://github.com/NetAuth/pam_netauth).
