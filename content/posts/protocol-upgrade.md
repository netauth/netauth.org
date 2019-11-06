---
title: "Protocol Upgrade"
date: 2019-10-01T22:58:27-08:00
---

NetAuth has just turned 2 years old, and in that time its changed a
lot.  The core ideals of being an easy to deploy, easy to reason about
network authentication service haven't changed, but the mechanisms
have changed a lot.  When NetAuth started out there weren't plans for
key management, untyped metadata, or anything beyond CRUD actions.
Searching wasn't even in the original scope for server side
implementation.

Times change though and features were added.  While the scope has been
aggressively defended to keep NetAuth lightweight and low on
complexity, the interface had to grow to support these new features.
As of release 0.2.0 the interface description looks like this:

```
service NetAuth {
  rpc Ping(PingRequest) returns (PingResponse) {}
  rpc AuthEntity(NetAuthRequest) returns (SimpleResult) {}
  rpc GetToken(NetAuthRequest) returns (TokenResult) {}
  rpc ValidateToken(NetAuthRequest) returns (SimpleResult) {}
  rpc LockEntity(NetAuthRequest) returns (SimpleResult) {}
  rpc UnlockEntity(NetAuthRequest) returns (SimpleResult) {}
  rpc ChangeSecret(ModEntityRequest) returns (SimpleResult) {}
  rpc ManageCapabilities(ModCapabilityRequest) returns (SimpleResult) {}
  rpc NewEntity(ModEntityRequest) returns (SimpleResult) {}
  rpc RemoveEntity(ModEntityRequest) returns (SimpleResult) {}
  rpc EntityInfo(NetAuthRequest) returns (Entity) {}
  rpc ModifyEntityMeta(ModEntityRequest) returns (SimpleResult) {}
  rpc ModifyEntityKeys(ModEntityKeyRequest) returns (KeyList) {}
  rpc ModifyUntypedEntityMeta(ModEntityMetaRequest) returns (UntypedMetaResult) {}
  rpc SearchEntities(SearchRequest) returns (EntityList) {}
  rpc NewGroup(ModGroupRequest) returns (SimpleResult) {}
  rpc DeleteGroup(ModGroupRequest) returns (SimpleResult) {}
  rpc ModifyGroupMeta(ModGroupRequest) returns (SimpleResult) {}
  rpc GroupInfo(ModGroupRequest) returns (GroupInfoResult) {}
  rpc ModifyUntypedGroupMeta(ModGroupMetaRequest) returns (UntypedMetaResult) {}
  rpc SearchGroups(SearchRequest) returns (GroupList) {}
  rpc ListGroups(GroupListRequest) returns (GroupList) {}
  rpc ListGroupMembers(GroupMemberRequest) returns (EntityList) {}
  rpc AddEntityToGroup(ModEntityMembershipRequest) returns (SimpleResult) {}
  rpc RemoveEntityFromGroup(ModEntityMembershipRequest) returns (SimpleResult) {}
  rpc ModifyGroupNesting(ModGroupNestingRequest) returns (SimpleResult) {}
}
```

This interface has 26 distinct RPC calls.  These calls take and return
21 different types of messages in variously nested forms.
Additionally, this interface is completely in-band.  Authentication
and stats data are signaled in band as strings and other simple types.
This means lots of duplication for request meta-data.  This is overall
not a maintainable design.

Beyond the type sprawl this API has other problems.  There's no
discernible naming convention, and there's no clear convention for
when rpc's are working on static views of the data, or will have modal
behavior based on input parameters.  Fixing this is of course
possible, but would involve slow changes throughout the rpc layer to
fix.  There's one more architectural problem with the rpc layer and
that has to do with the context.

In Go, the context is a "magic" type that contains all kinds of
metadata about an incoming RPC and should be threaded through the
entire call chain.  This threading is important because it carries the
other major component of the context as well, which is the
cancellation chain.  This cancellation mechanism allows functions to
stop work if the caller has decided they no longer want the results.
Such a scenario might happen if a client disconnects or times out.
Threading the context through would involve changing the signature on
almost every function in the call chain for the RPC layer, and would
involve changes at the tree layer to thread down to the database
layer.  Since the context is a vital part of most tracing
implementations, it is important to thread it across services as well
as through them.

Finally, the RPC interface is what developers interact with.  Its the
component that exists outside of the sole control of the NetAuth
project, and it needs to be clean and polished to properly reflect the
responsibility it has.  In order to achieve these goals, it has been
decided to release a v2 protocol specification that is cleaner, more
maintainable, and more correctly implemented on the server.  This
specification is shown below:

```
  rpc EntityCreate(EntityRequest) returns (Empty) {}
  rpc EntityUpdate(EntityRequest) returns (Empty) {}
  rpc EntityInfo(EntityRequest) returns (ListOfEntities) {}
  rpc EntitySearch(SearchRequest) returns (ListOfEntities) {}
  rpc EntityUM(KVRequest) returns (ListOfStrings) {}
  rpc EntityKeys(KVRequest) returns (ListOfStrings) {}
  rpc EntityDestroy(EntityRequest) returns (Empty) {}
  rpc EntityLock(EntityRequest) returns (Empty) {}
  rpc EntityUnlock(EntityRequest) returns (Empty) {}
  rpc EntityGroups(EntityRequest) returns (ListOfGroups) {}
  rpc AuthEntity(AuthRequest) returns (Empty) {}
  rpc AuthGetToken(AuthRequest) returns (AuthResult) {}
  rpc AuthValidateToken(AuthRequest) returns (Empty) {}
  rpc AuthChangeSecret(AuthRequest) returns (Empty) {}
  rpc GroupCreate(GroupRequest) returns (Empty) {}
  rpc GroupUpdate(GroupRequest) returns (Empty) {}
  rpc GroupInfo(GroupRequest) returns (ListOfGroups) {}
  rpc GroupUM(KVRequest) returns (ListOfStrings) {}
  rpc GroupUpdateRules(GroupRulesRequest) returns (Empty) {}
  rpc GroupAddMember(EntityRequest) returns (Empty) {}
  rpc GroupDelMember(EntityRequest) returns (Empty) {}
  rpc GroupDestroy(GroupRequest) returns (Empty) {}
  rpc GroupMembers(GroupRequest) returns (ListOfEntities) {}
  rpc GroupSearch(SearchRequest) returns (ListOfGroups) {}
  rpc SystemCapabilities(CapabilityRequest) returns (Empty) {}
  rpc SystemPing(Empty) returns (Empty) {}
  rpc SystemStatus(Empty) returns (ServerStatus) {}
```

The API has actually grown by one function in the rewrite, gaining an
additional more verbose status method `SystemStatus`.  Having grown
though, the names in the interface are much more consistent, and are
much clearer what they do.  Furthermore, rather than the 21 different
message types, there are now only 14 message types, 1 of which is a
literal empty message since error signaling and authentication are now
out of band values.

Another important change came about with the development of the
NetAuth2 protocol.  Since the behavior of the interface was clearly
defined from the beginning, it was possible to develop the module with
100% test coverage for interface behavior.  This now means that the
critical path for serving has greater than 90% test coverage, and
represents a major milestone to the stability promises of NetAuth.

Of course, NetAuth is designed to be base layer infrastructure, and so
just swapping one protocol for another is unacceptable.  The next 2
releases (0.3 and 0.4) will understand both the original protocol, and
this v2 version.  As the original protocol has minimal tests though,
it is recommended to disable it if not required.  A flag is provided
for this purpose.  With the refactoring of the v2 API, there are no
additional major changes planned to the public NetAuth API, and
external developers are welcome to begin developing modules that will
target the stable API in version 0.3.
