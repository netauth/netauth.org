---
title: "New Integrations: Python and Buildbot"
date: 2024-08-06T00:00:00-04:00
---

It's now easier than ever to write an application that interfaces with NetAuth
in Python, using the new NetAuth Python library. Integrating NetAuth
authentication into a Python application can be as simple as:

```py
import netauth
conn = netauth.NetAuth("netauth.example.org")
def auth(username: str, password: str) -> bool:
    try:
        conn.auth_entity(username, password)
        # success!
        return True
    except netauth.error.UnauthenticatedError:
        # failure :(
        return False
    except netauth.error.NetAuthRpcError as e:
        # uh oh
        print(f"Request failed: {e}")
        return False
```

[Buildbot](https://buildbot.net) is a highly-extensible continuous integration
framework written in Python. Void Linux uses Buildbot to build its packages
for delivery to users. To make it easier for Void's developers to interact with
the Buildbot, the NetAuth Python library was used to write an authentication and
authorization plugin for Buildbot. This plugin handles authentication, passes
user information from NetAuth to Buildbot, and allows setting ACLs based on
NetAuth groups.

The source code for the [Python
library](https://github.com/NetAuth/netauth-python) and [Buildbot
plugin](https://github.com/classabbyamp/buildbot-netauth) are on Github.
