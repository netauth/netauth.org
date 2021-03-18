---
title: "LDAP"
date: 2021-03-18T01:40:00-06:00
---

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/netauth/ldap/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/netauth/ldap)](https://goreportcard.com/report/github.com/netauth/ldap)

Sometimes adapting an application to query NetAuth directly isn't
practical, and instead its easier to adapt NetAuth to speak to the
application.  The LDAP proxy provides a "meet in the middle" approach
where an application will speak LDAP to the proxy, and the proxy
translates these requests to the native NetAuth2 protocol for
transmission to the server.

## Installation

Precompiled binaries can be obtained from the [GitHub Releases
Page](https://github.com/netauth/ldap/releases).  Additionally, docker
containers are provided from the GitHub Container Registry.

If you wish to install as a system daemon the process must either be
run as root or provided sufficient capabilities to bind port 389 for
cleartext mode, or 636 for TLS encrypted mode.  You are strongly
encouraged to use TLS encryption, and only use unencrypted LDAP for
instances where the communications can be secured via alternate means,
such as binding only to localhost.

## Usage

The proxy reads the standard NetAuth configuration file and serves a
read-only LDAP socket on port 389 (unencrypted) or port 636 (TLS).
Applications may connect to the server and query as though the proxy
were a basic LDAP server, noting that complex filter expressions are
likely to not work.

## Examples

The proxy is regularly tested against Hashicorp Vault and Grafana.
Examples of each configuration are shown below:

### Hashicorp Vault

Vault must be configured to use the LDAP auth backend.  Parameters
should be configured as shown below:

```hcl
  path        = "ldap"
  url         = "ldap://localhost"
  userdn      = "ou=entities,dc=netauth,dc=example,dc=com"
  userattr    = "uid"
  discoverdn  = false
  groupdn     = "ou=entities,dc=netauth,dc=example,dc=com"
  groupfilter = "(uid={{.Username}})"
  groupattr   = "memberOf"
```

### Grafana

Grafana can be configured to authenticate and dynamically provision
LDAP users similar to this example config:

```toml
[[servers]]
host = "localhost"
port = 389
use_ssl = false
bind_dn = "uid=%s,ou=entities,dc=netauth,dc=example,dc=com"
search_filter = "(uid=%s)"
search_base_dns = ["ou=entities,dc=netauth,dc=example,dc=com"]

[servers.attributes]
username = "uid"
member_of = "memberOf"

[[servers.group_mappings]]
group_dn = "cn=grafana-root,ou=groups,dc=netauth,dc=example,dc=com"
org_role = "Admin"
grafana_admin = true # Available in Grafana v5.3 and above

[[servers.group_mappings]]
group_dn = "cn=grafana-editor,ou=groups,dc=netauth,dc=example,dc=com"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```

Note that the above example also shows how you can assign NetAuth
groups to Grafana roles to simplify permissions provisioning.
