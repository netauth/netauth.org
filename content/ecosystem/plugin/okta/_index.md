---
title: "Okta"
date: 2019-09-14T15:49:54-07:00
---

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NetAuth/plugin-okta/blob/master/LICENSE)
[![Build Status](https://travis-ci.org/NetAuth/plugin-okta.svg?branch=master)](https://travis-ci.org/NetAuth/plugin-okta)
[![Go Report Card](https://goreportcard.com/badge/github.com/NetAuth/plugin-okta)](https://goreportcard.com/report/github.com/NetAuth/plugin-okta)

Okta is a popular hosted identity solution with support for SAML and
OIDC protocols.  While it has a built in directory it is fairly
primitive compared to NetAuth's group expansion rules which are
evaluated at request time.  Additionally Okta is unable to provide
authentication to the local network except via RADIUS, which requires
a machine on which to run the Okta connector.

The Okta plugin can be obtained from its [GitHub
Page](https://github.com/NetAuth/plugin-okta).

To use the plugin, you will need to obtain an Okta API key.  Follow
the documentation on [this
page](https://developer.okta.com/docs/guides/create-an-api-token/overview/)
if you don't already have one.  Its recommended to create a service
user for NetAuth, make that user a super admin, and then generate the
key as that user.  This is to ensure that actions are logged as
happening by NetAuth since a quirk in the Okta platform logs actions
as though they were taken by a user even when performed via API.

Once you have created your API token, you should obtain an okta plugin
binary.  You can either obtain a binary release from github, or build
it as shown below:

```shell
$ git clone -b <version> git://github.com/NetAuth/plugin-okta.git
$ go build -o okta.treeplugin .
```

Place `okta.treeplugin` in your server's plugin directory, and add the
following stanza to the configuration file:

```toml
  [plugin.okta]
    token = "<token>"
    domain = "<domain>"
    orgurl = "https://<sso_domain>"
    interval = "<sync_period>"
```

Fill in the plugin stanza as follows:

  * token - This is the token value obtained from the Okta console.
  * domain - This value will be appended to entities to produce the
    email and username that is supplied to Okta.  For example if this
    value were `netauth.org` and the username were `maldridge`, then
    the username would be `maldridge@netauth.org`.
  * orgurl - This is the URL on which your organization sign in page
    is served by default.  This value must include the `https://`
    prefix.
  * interval - This value controls how frequently group information is
    synchronized to Okta.  CRUD information is propogated in real
    time, but group information is synchronized on a schedule.  This
    should be set to a reasonable amount of time for your
    organization.  If you have no other idea of what to set this value
    to, set it to `20m`.

The Okta plugin is not supported or recognized as official software by
Okta, so for questions and concerns, please funnel those to the
`#netauth-dev` channel on freenode.

