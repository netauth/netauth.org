---
title: "Integration: Okta"
date: 2019-09-08T19:24:57-07:00
---

NetAuth is a complete local identity system, but what happens when you
need an identity that's available "in the cloud"?  Likely in the
format such as a grid of SAML apps that your users can click on and
use their same identity beyond the edge of your network.

One of the biggest names in the market for hosted SAML in today's
market is Okta.  In the 0.2.0 release, NetAuth has a plugin which will
allow you to mirror changes from your NetAuth system into an Okta
account.  This plugin is easy to use and easy to use, as its actions
are completely transparent to users of both systems.  You have "real"
Okta users and "real" NetAuth entities, and you can use NetAuth's
advanced group rules system to make arbitrarily complex membership
trees and then propagate those trees to Okta.

The source code for the Okta plugin is [on
github](https://github.com/NetAuth/plugin-okta).
