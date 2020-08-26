---
title: "Masquerade"
date: 2020-08-20T21:42:43-07:00
---

Sometimes, there's no way around speaking a particular protocol.  This
can be the case with legacy software, or software that isn't easy to
modify.  One of the most common protocols understood across a wide
variety of software is LDAP.

LDAP is an incredibly extensive protocol, with an extensions system
that allows it to handle any kind of data that can be encoded as an
octet string.  In order to support this use case, a NetAuth LDAP proxy
is being developed.  This proxy will enable applications that speak
exclusively LDAP to gain limited read-only access to the data held in
the NetAuth system.

Because LDAP is a very complex and extensible protocol, not all of its
features will be available via the proxy.  For the initial development
support will be focused around permitting interoperability with
Hashicorp Vault's LDAP auth backend.
