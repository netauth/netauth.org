---
title: "Plugins"
date: 2019-09-10T15:49:54-07:00
---

Plugins are the extensibility mechanism for NetAuth.  They are able to
extend or augment certain behaviors of the NetAuth server and to
extend its functionality beyond what is otherwise possible.

Below is a list of plugins currently available.

## [fail2lock](fail2lock/)

Fail2lock provides the classic X tries in Y time mechanism to limit
password attempts.

## [Okta](okta/)

Okta is a popular hosted SAML and OIDC solution.  You can use this
plugin to maintain all of your Okta users from inside of NetAuth, and
to mirror changes from NetAuth to Okta.
