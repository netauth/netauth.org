---
title: "Case Study: Void Linux"
date: 2018-09-13T15:49:17-07:00
---

From the very start, there was an intention that NetAuth would be
usable over the internet at large.  This informed many design
decisions and drove a lot of features that were needed in the initial
launch.

Lets dig a bit deeper into how Void Linux is using NetAuth to secure
its servers and provide centralized management of resourced.  An
important note for this article: the principal author of NetAuth
(`maldridge`) is also the infrastructure lead for Void Linux.

Void has a collection of VMs and physical hardware scattered across a
number of providers.  These machines are on 2 different continents,
and serve a large pool of end users by building and mirroring binary
packages.  These machines also host a wikimedia installation, and a
series of ancillary services that keep the distribution running.

Needless to say, it is important to restrict the number of logins
available on these systems, as well as to ensure that only authorized
persons can access the infrastructure.

Previously, account provisioning had been handled by having a project
lead connect to a machine using SSH and provision an account.  After
this account was provisioned, a somewhat clumsy exchange would take
place whereby at least one ssh key would be provisioned on the
machine.  Passwords were another matter, and were set by dropping a
file into the new users home folder which contained an initial
password string.

This process was error proned, difficult to manage, and led to many
people running around with the ability to become root at any time.
This is an unfortunate side effect of having an unmanaged user
structure since it meant that sudo classes couldn't be provisioned
uniformly across the fleet.  This also doesn't even begin to touch on
other systems which need to derive authenticated sessions from
operators.

Void needed a central source of user and group information, but also
needed to operate over the internet at large, which precludes the
ability to use technologies like LDAP and Kerberos, both of which are
not recommended for use across untrusted networks.

The actual infrastructure deployed to run the `netauthd` server is
quite minimal.  A dedicated VM with a restricted login group runs the
server.  The server uses the default ProtoDB storage engine as well as
the default bcrypt hashing plugin.  Certificates are provided by an
offline CA that exists to service the authentication systems of the
project.

Access to this VM is restricted to people who already hold the
`GLOBAL_ROOT` capability, which is in turn restricted to members of
the project that could already obtain this access via ownership of the
GitHub organization.  Only the NetAuth server or services of
equivalent service security class are permitted to run on the same VM.

Void currently has two kinds of clients connected to NetAuth.  The
first kind is the traditional machine type of candidate that accounts
for the bulk of all connections.  The second type is a specialized
client that provides secure state to other services.

### Linux Clients

Linux clients are connected to the NetAuth server with a set of 3
components.  The first two components permit password authentication
and provide authorization information by supporting PAM and NSS.  The
PAM integration is performed via `pam_netauth` which is added to the
`system-authentication` chain.

The other component that needs to be provided to the system is a set
of SSH keys.  These should be consistent across all hosts and looked
up dynamically when requested.  The `NetKeys` binary makes these
specialized lookups and provides the keys in the format that sshd
expects them to be in.

### Terraform

Void makes heavy use of Terraform to update state on GitHub and on
Google Cloud.  One of the great shortcomings of Terraform is that it
has no ability to separate out secure data from the "state" that it
needs to maintain between invocations.  This leads to the situation
where a central location needs to store the state so that multiple
people can apply Terraform state translations, but also must
authenticate before obtaining state since it may contain sensitive
information.

This is accomplished by a separate daemon running on the NetAuth
server called TerraState.  TerraState stores the state and uses
NetAuth credentials to validate that entities attempting to obtain
state are authorized to do so.


## In Closing

Void is a great match for NetAuth right now.  To continue pushing
NetAuth on Void will require the introduction of an oauth gateway and
a web interface for entity management.  This will allow Void to hook
in its existing web services to a single sign on environment rather
than having different accounts on the wiki, forum, and build
interface.


Feel free to reach out on `#voidlinux` on FreeNode if you'd like to
know more about Void's use of NetAuth.
