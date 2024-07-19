# NATS Docs


### Welcome

NATS is a simple, secure and high performance open source data layer for cloud native
applications, IoT messaging, and microservices architectures.

We feel that it should be the backbone of your communication between services. It
doesn't matter what language, protocol, or platform you are using; NATS is the best way
to connect your services.

```
Publish and subscribe to messages at millions of messages per second. At least once
delivery.
Supports fan-in/out delivery patterns
Request/reply
Every major language is supported
Persistence via JetStream
at most once delivery or exactly once delivery
work queues
stream processing
data replication
data retention
data deduplication
Higher order data structures
Key/Value with watchers, versioning, and TTL
Object storage with versioning
Security
TLS
```
#### The official NATS documentation

##### 10,000 foot view


All of this in a single binary that is easy to deploy and manage. No external dependencies,
just drop it in and add a conguration le to point to other NATS servers and you are
ready to go. In fact, you can even embed NATS in your application (for Go users)!

1. In general we reccomend trying to solve your problems rst using Core NATS.
2. If you need to share state between services, take a look at the KV or Object Store in
    JetStream.
3. When you need lower level access to persistence streams, move on to using
    JetStream directly for more advanced messaging patterns.
4. Learn about deployment strategies
5. Secure your deployments with zero trust security

NATS is Open Source as is this documentation. Please let us know if you have updates
and/or suggestions for these docs. You can also create a Pull Request using the
Edit on GitHub link on each page.

```
JWT-based zero trust security
Clustering
High availability
Fault tolerance
Auto-discovery
Protocols supported
TCP
MQTT
WebSockets
```
#### Guided tour

#### Contribute


Feel free to chat with us on Slack slack.nats.io.

Thank you from the entire NATS Team of Maintainers for your interest in NATS!

#### Additional questions?


## Release Notes


### What's New!

The NATS.io team is always working to bring you features to improve your NATS
experience. Below you will nd feature summaries for new NATS implementations. Check
back often for release highlights and updates.

Check out the:

Please check out the announcement post on the blog and the detailed release notes in
the server repo.

Support for a min_version in the leafnodes{} that would reject servers with a lower
version. Note that this would work only for servers that are v2.8.0 and above.

```
Upgrade guide
Podcast EP06: The journey and features of the NATS.io 2.10 release
Release notes
```
```
Server version in monitoring landing page.
Logging to /healthz endpoint when failure occurs.
```
#### Server release v2.10.

#### Server release v2.9.

#### Server release v2.8.

##### LeafNode

##### Monitoring


For full release information, see links below;

See important note if using LeafNode regarding domains.

Ability to congure account limits (max_connections, max_subscriptions,
max_payload, max_leafnodes) in server conguration le.

```
MQTT and Websocket blocks in the /varz endpoint.
```
```
Consumer check added to healthz endpoint.
Max stream bytes checks.
Ability to limit a consumer's MaxAckPending value.
Allow streams and consumers to migrate between clusters. This feature is considered
"beta".
New unique_tag option in jetstream{} conguration block to prevent placing a
stream in the same availability zone twice.
Stream Alternates eld in StreamInfo response. They provide a priority list of
mirrors and the source in relation to where the request originated.
Deterministic subject tokens to partition mapping.
```
```
Release notes 2.8.
Full list of Changes 2.7.4...2.8.
```
##### JetStream

#### Server release v2.7.

##### Notice for JetStream Users

##### Configuration


Support for websocket protocol. MQTT clients must connect to the opened websocket
port and add /mqtt to the URL path.

Ability to rate-limit the clients connections by adding the
connection_rate_limit: <number of connections per seconds> in the tls{} top-
level block.

For full release information, see links below;

```
Overow placement for streams. A stream can now be placed in the closest cluster
from the origin request if it can be placed there.
Support for ephemeral Pull consumers (client libraries will need to be updated to allow
those).
New consumer conguration options
For Pull Consumers: MaxRequestBatch to limit the batch size any client can
request MaxRequestExpires to limit the expiration any client can request
For ephemeral consumers: InactiveThreshold duration that instructs the server
to cleanup ephemeral consumers that are inactive for that long.
Ability to congure max_file_store and max_memory_store in the jetstream{}
block as strings with the following suxes K , M , G and T , for instance:
max_file_store: "256M".
Support for the JWT eld MaxBytesRequired, which denes a per-account maximum
bytes for assets.
```
```
Release notes 2.7.
Full list of Changes 2.6.6...2.7.
```
##### JetStream

##### MQTT

##### TLS


See important note if upgrading from a version prior to NATS Server v2.4.0.

See important notes if upgrading from a version prior to v2.5.0.

For full release information, see links below;

See important note if upgrading from a version prior to NATS Server v2.4.0.

```
JetStream's reserved memory and memory used from accounts with reservations in
/jsz and /varz endpoints
Hardened systemd service
```
```
Release notes 2.6.
Full list of Changes 2.5.0...2.6.
```
#### Server release v2.6.

##### Notice for JetStream Users

##### Notice for MQTT Users

##### Monitoring

#### Server release v2.5.

##### Notice for JetStream Users

##### MQTT/Monitoring


For full release information, see links below;

With the latest release of the NATS server, we have x ed bugs around queue
subscriptions and have restricted undesired behavior that could be confusing or
introduce data loss by unintended/undened behavior of client applications. If you are
using queue subscriptions on a JetStream Push Consumer or have created multiple push
subscriptions on the same consumer, you may be affected and need to upgrade your
client version along with the server version. We’ve detailed the behavior with different
client versions below.

```
MQTTClient in the /connz connections report and system events CONNECT and
DISCONNECT. Ability to select on mqtt_client.
```
```
Sessions are now all stored inside a single stream, as opposed to individual streams,
reducing resources usage.
```
```
Due to the aforementioned improvement described above, when an MQTT client
connects for the rst time after an upgrade to this server version, the server will
migrate all individual $MQTT_sess_<xxxx> streams to a new $MQTT_sess stream for
the user's account.
```
```
Release notes 2.5.
Full list of Changes 2.4.0...2.5.
```
##### MQTT Improvement

##### MQTT Update

#### Server release v2.4.

##### Notice for JetStream Users


With a NATS Server **prior** to v2.4.0 and client libraries **prior** to these versions: NATS C
client v3.1.0, Go client v1.12.0, Java client 2.12.0-SNAPSHOT, NATS.js v2.2.0, NATS.ws
v1.3.0, NATS.deno v1.2.0, NATS .NET 0.14.0-pre2:

If above client libraries are not updated to the latest but the NATS Server is upgraded to
v2.4.0:

```
It was possible to create multiple non-queue subscription instances for the same
JetStream durable consumer. This is not correct since each instance will receive the
same copy of a message and acknowledgment is therefore meaningless since the rst
instance to acknowledge the message will prevent other instances to control if/when a
message should be acknowledged.
Similar to the rst issue, it was possible to create many different queue groups for one
single JetStream consumer.
For queue subscriptions, if no consumer nor durable name was provided, the libraries
would create ephemeral JetStream consumers, which meant that each member of the
same group would receive the same message as the other members, which was not
the expected behavior. Users assumed that 2 members subscribing to “foo” with the
queue group named “bar” would load-balance the consumption of messages from the
stream/consumer.
It was possible to create a queue subscription on a JetStream consumer congured
with heartbeat and/or ow control. This does not make sense because by denition,
queue members would receive some (randomly distributed) messages, so the library
would think that heartbeats are missed, and ow control would also be disrupted.
```
```
It is still possible to create multiple non-queue subscription instances for the same
JetStream durable consumer. Since the check is performed by the library (with the
help of a new eld called PushBound in the consumer information object set by the
server), this misbehavior is still possible.
Queue subscriptions will not receive any message. This is because the server now has
a new eld DeliverGroup in the consumer conguration, which won’t be set for
existing JetStream consumers and by the older libraries, and detects interest (and
starts delivering) only when a subscription on the deliver subject for a queue
subscription matching the “deliver group” name is found. Since the JetStream
consumer is thought to be a non-deliver-group consumer, the opposite happens: the
server detects a core NATS queue subscription on the “deliver subject”, therefore does
not trigger delivery on the JetStream consumer’s “deliver subject”.
```

The 2 other issues are still present because those checks are done in the updated
libraries.

If the above client libraries are updated to the latest version, but the NATS Server is still to
version prior to v2.4.0 (that is, up to v2.3.4):

For completeness, using the latest client libraries and NATS Server v2.4.0:

Note that if the server v2.4.0 recovers existing JetStream consumers that were created
prior to v2.4.0 (and with older libraries), none of them will have a DeliverGroup, so none

```
It is still possible to create multiple non-queue subscription instances for the same
JetStream durable consumer. This is because the JetStream consumer’s information
retrieved by the library will not have the PushBound boolean set by the server,
therefore will not be able to alert the user that they are trying to create multiple
subscription instances for the same JetStream consumer.
Queue subscriptions will fail because the consumer information returned will not
contain the DeliverGroup eld. The error will be likely to the effect that the user tries
to create a queue subscription to a non-queue JetStream consumer. Note that if the
application creates a queue subscription for a non-yet created JetStream consumer,
then this call will succeed, however, adding new members or restarting the application
with the now existing JetStream consumer will fail.
Creating queue subscriptions without a named consumer/durable will now result in
the library using the queue name as the durable name.
Trying to create a queue subscription with a consumer conguration that has
heartbeat and/or ow control will now return an error message.
```
```
Trying to start multiple non-queue subscriptions instances for the same JetStream
consumer will now return an error to the effect that the user is trying to create a
“duplicate subscription”. That is, there is already an active subscription on that
JetStream consumer. It is now only possible to create a queue group for a JetStream
consumer created for that group. The DeliverGroup eld will be set by the library or
need to be provided when creating the consumer externally.
Trying to create a queue subscription without a durable nor consumer name results in
the library creating/using the queue group as the JetStream consumer’s durable name.
Trying to create a queue subscription with a consumer conguration that has
heartbeat and/or ow control will now return an error message.
```

of them can be used for queue subscriptions. They will have to be recreated.

For full release information, see links below;

```
Domain to the content of a PubAck protocol
PushBound boolean in ConsumerInfo to indicate that a push consumer is already
bound to an active subscription
DeliverGroup string in ConsumerConfig to specify which deliver group (or queue
group name) the consumer is created for
Warning log statement in situations where catchup for a stream resulted in an error
```
```
The ability for normal accounts to access scoped connz information
```
```
Operator option resolver_pinned_accounts to ensure users are signed by certain
accounts
```
```
Release notes 2.4.
Full list of Changes 2.3.4...2.4.
```
```
OCSP support
```
##### JetStream

##### Monitoring

##### Misc

#### Server release v2.3.

##### JetStream


For full release information, see links below;

See NATS 2.2 for new features.

Monitoring endpoints as listed in the table below are accessible as system services using
the following subject pattern:

For more information on monitoring endpoints see NATS Server Congurations System
Events.

```
Richer API errors. JetStream errors now contain an ErrCode that uniquely describes
the error.
Ability to send more advanced Stream purge requests that can purge all messages for
a specic subject
Stream can now be congured with a per-subject message limit
Encryption of JetStream data at rest
```
```
Release notes 2.3.
Full list of Changes 2.2.6...2.3.
```
```
$SYS.REQ.SERVER.<id>.<endpoint-name> (request server monitoring endpoint
corresponding to endpoint name.)
$SYS.REQ.SERVER.PING.<endpoint-name> (from all server request server monitoring
endpoint corresponding to endpoint name - will return multiple messages)
```
#### Server release v2.2.

#### Server release v2.1.

##### Monitoring Endpoints Available via System Services

##### Addition of no_auth_user Configuration


Conguration of no_auth_user allows you to refer to a congured user/account when
no credentials are provided.

For more information and examples, see Securing NATS

For full release information, see links below;

This release adds the ability to specify TLS conguration for the account resolver.

trace_verbose and command line parameters -VV and -DVV added. See NATS
Logging Conguration

We've added the option to include subscription details in monitoring endpoints /routez
and /connz. For instance /connz?subs=detail will now return not only the subjects of
the subscription, but the queue name (if applicable) and some other details.

```
Release notes 2.1.
Full list of Changes 2.1.6...2.1.
```
```
resolver_tls {
cert_file: ...
key_file: ...
ca_file: ...
}
```
#### Server release v2.1.

##### TLS Configuration for Account Resolver

##### Additional Trace & Debug Verbosity Options

##### Subscription Details in Monitoring Endpoints


NATS introduces logfile_size_limit allowing auto-rotation of log les when the size
is greater than the congured limit set in logfile_size_limit as a number of bytes.
You can provide the size with units, such as MB, GB, etc. The backup les will have the
same name as the original log le with the sux .yyyy.mm.dd.hh.mm.ss.micros. For more
information see Conguring Logging in the NATS Server Conguration section.

Queue Permissions allow you to express authorization for queue groups. As queue
groups are integral to implementing horizontally scalable microservices, control of who is
allowed to join a specic queue group is important to the overall security model. Original
PR - https://github.com/nats-io/nats-server/pull/

More information on Queue Permissions can be found in the Developing with NATS
section.

```
Release notes 2.1.
Full list of Changes 2.1.4...2.1.
```
```
Release notes 2.1.
Full list of Changes 2.1.2...2.1.
```
#### Server release v2.1.

##### Log Rotation

#### Server release v2.1.

##### Queue Permissions

#### Server release v2.1.


As services and service mesh functionality has become prominent, we have been looking
at ways to make running scalable services on NATS.io a great experience. One area we
have been looking at is observability. With publish/subscribe systems, everything is
inherently observable, however we realized it was not as simple as it could be. We wanted
the ability to transparently add service latency tracking to any given service with no
changes to the application. We also realized that global systems, such as those NATS.io
can support, needed something more than a single metric. The solution was to allow any
sampling rate to be attached to an exported service, with a delivery subject for all
collected metrics. We collect metrics that show the requestor’s view of latency, the
responder’s view of latency and the NATS subsystem itself, even when requestor and
responder are in different parts of the world and connected to different servers in a NATS
supercluster.

For services, the authorization for responding to requests usually included wildcards for
_INBOX.> and possibly $GR.> with a supercluster for sending responses. What we really
wanted was the ability to allow a service responder to only respond to the reply subject it
was sent.

Exported Services were originally tied to a single response. We added the type for the
service response and now support singletons (default), streams and chunked. Stream

```
Release notes 2.1.
Full list of Changes 2.0.4...2.1.
```
##### Service Latency Tracking

#### Server release v2.0.

##### Response Only Permissions

##### Response Types


responses represent multiple response messages, chunked represents a single response
that may have to be broken up into multiple messages.

```
Release notes 2.0.
Full list of Changes 2.0.2...2.0.
```

### NATS 2.

This guide is tailored for existing NATS users upgrading from NATS version 2.9.x. This
will read as a summary with links to specic documentation pages to learn more about
the feature or improvement.

Although all existing client versions will work, new client versions will expose additional
options used to leverage new features. The minimum client versions that have full 2.10.
support include:

```
CLI - v0.1.
nats.go - v1.30.
nats.rs - v0.32.
nats.deno - v1.17.
nats.js - v2.17.
nats.ws - v1.18.
nats.java - v2.17.
nats.net - v1.1.
nats.net.v2 - Coming soon!
nats.py - Coming soon!
nats.c - Coming soon!
```
```
k8s/nats - v1.1.
k8s/nack - v0.24.
```
#### Upgrade considerations

##### Client versions

##### Helm charts


For critical infrastructure like NATS, zero downtime upgrades are table stakes. Although
the best practice for all infrastructure like this is for users to thoroughly test a new
release against your specic workloads, inevitably there are cases where an upgrade
occurs in production followed by a decision to downgrade. This is never recommended
and can cause more harm than good for most infrastructure and data systems.

Below are a few important considerations if downgrading is required.

2.10.0 brings on-disk storage changes which bring signicant performance
improvements. These are not compatible with previous versions of the NATS Server. If an
upgrade is performed to a server with existing stream data on disk, followed by a
downgrade, the older version server will not understand the stream data in the new
format.

However, being mindful of the possibility of the need to downgrade, a special version of
the 2.9.x series was released with awareness of key changes in the new storage format,
allowing it to startup properly.

The takeaway is that if a downgrade is the only resort, it must be to 2.9.22 or later to
ensure storage format changes are handled appropriately.

There are new stream and consumer conguration options that could be problematic if a
downgrade occurs since previous versions of the server have no awareness of them.
Examples include:

```
Multi-lter consumers - Downgrading would result in no lter being applied since the
new eld is congured as a list rather than a single string.
Subject-transform on streams - Downgrading would result in the subject transform not
being applied since the server has no awareness of it.
```
##### Downgrade warnings

**Storage format changes**

**Stream and consumer config options**


```
Compression on streams - Downgrading when compression is enabled on streams will
cause those streams to become unloadable since the older server versions will not
understand the compression being used.
```
```
Experimental support for IBM z/OS
Experimental support for NetBSD
```
```
A server reload can now be performed by sending a message on
$SYS.REQ.SERVER.<server-id>.RELOAD by a client authenticated in the system
account.
```
```
A new sync_interval server cong option has been added to change the default
sync interval of stream data when written to disk, including allowing all writes to be
ushed immediately. This option is only relevant if you need to modify durability
guarantees.
```
```
Subject mappings can now be cluster-scoped and weighted, enabling the ability to
have different mappings or weights on a per cluster basis.
The requirement to use all wildcard tokens in subject mapping or transforms has been
relaxed. This can be applied to cong or account-based subject mapping, stream
subject transforms, and stream republishing, but not on subject mappings that are
associated with stream and service import/export between accounts.
```
#### Features

##### Platforms

##### Reload

##### JetStream

##### Subject mapping


```
A subject_transform eld has been added enabling per-stream subject transforms.
This applies to standard streams, mirrors, and sourced streams.
A metadata eld has been added to stream conguration enabling arbitrary user-
dened key-value data. This is to supplant or augment the description eld.
A first_seq eld has been added to stream conguration enabling explicitly setting
the initial sequence on stream creation.
A compression eld has been added to stream conguration enabling on-disk
compression for le-based streams.
The ability to edit the republish cong option on a stream after stream creation was
added.
A Nats-Time-Stamp header is now included in republished messages containing the
original message's timestamp.
A ts eld has been added to stream info responses indicating the server time of the
snapshot. This was added to allow for local time calculations relying on the local
clock.
An array of subject-transforms (subject lter + subject transform destination) can be
added to a mirror or source conguration (can not use the single subject lter/subject
transform destination elds at the same time as the array).
A stream congured with sources can source from the same stream multiple times
when distinct lter+transform options are used, allowing for some messages of a
stream to be sourced more than once.
```
```
A filter_subjects eld has been added which enables applying server-side ltering
against multiple disjoint subjects, rather than only one.
A metadata eld has been added to consumer conguration enabling arbitrary user-
dened key-value data. This is to supplant or augment the description eld.
A ts eld has been added to consumer info responses indicating the server time of
the snapshot. This was added to allow for local time calculations without relying on
the local clock.
```
##### Streams

##### Consumers


```
A metadata eld has been added to key-value conguration enabling arbitrary user-
dened key-value data. This is to supplant or augment the description eld.
A bucket congured as a mirror or sourcing from other buckets
```
```
A metadata eld has been added to object store conguration enabling arbitrary
user-dened key-value data. This is to supplant or augment the description eld.
```
```
A pluggable server extension, referred to as auth callout, has been added. This
provides a mechanism for delegating authentication checks against a bring-your-own
(BYO) provider and, optionally, dynamically declaring permissions for the authenticated
user.
```
```
A unique_tag eld has been added to the /varz and /jsz HTTP endpoint
responses, corresponding to the value of unique_tag dened in the server cong.
A slow_consumer_stats eld has been added to the /varz HTTP endpoint
providing a count of slow consumers for clients, routes, gateways, and leafnodes.
A raft=1 query parameter has been added to the /jsz HTTP endpoint which adds
stream_raft_group and consumer_raft_groups elds t o the response.
A num_subscriptions eld has been added to the $SYS.REQ.SERVER.PING.STATZ
NATS endpoint responses.
A system account responder for $SYS.REQ.SERVER.PING.IDZ has been added which
returns info for the server that the client is connected to.
A system account responder for $SYS.REQ.SERVER.PING.PROFILEZ has been added
and works even if a proling port is not enabled in the server conguration.
```
##### Key-value

##### Object store

##### Authn/Authz

##### Monitoring


```
A user account responder for $SYS.REQ.USER.INFO has been added which allows a
connected user to query for the account they are in and permissions they have.
```
```
Support for QoS2 has been added. Check out the new MQTT implementation details
overview.
```
```
When dening routes between servers, a handful of optimizations have been
introduced including a pool of TCP connections between servers, optional pinning of
accounts to connections, and optional compression of trac. There is quite a bit to dig
into, so check out the v2 routes page for details.
```
```
A handshake_first cong option has been added enabling TLS-rst handshakes for
leafnode connections.
```
```
The NATS_STARTUP_DELAY environment variable has been added to allow changing
the default startup for the server of 10 seconds
```
```
The nats-server --signal command now supports a glob expression on the
<pid> argument which would match a subset of all nats-server instances running
on the host.
```
##### MQTT

##### Clustering

##### Leafnodes

##### Windows

#### Improvements

##### Reload


```
Prior to 2.10, setting republish conguration on mirrors would result in an error. On
sourcing streams, only messages that were actively between stored matching
congured subjects would be republished. The behavior has been relaxed to allow
republishing on mirrors and includes all messages on sourcing streams.
```
```
A new header has been added on a fetch response that indicates to clients the fetch
has been fullled without requiring clients to rely on heartbeats. It avoids some
conditions in which the client would issue fetch requests that could go over limits or
have more fetch requests pending than required.
```
```
Previously, a leafnode congured with two or more remotes binding to the same hub
account would be rejected. This restriction has been relaxed since each remote could
be binding to a different local account.
```
```
Previously a dot. in an MQTT topic was not supported, however now it is! Check
out the topic-subject conversion table for details.
```
##### Streams

##### Consumers

##### Leafnodes

##### MQTT


### NATS 2.2

NATS 2.2 is the largest feature release since version 2.0. The 2.2 release provides highly
scalable, highly performant, secure and easy-to-use next generation streaming in the form
of JetStream, allows remote access via websockets, has simplied NATS account
management, native MQTT support, and further enables NATS toward our goal of
securely democratizing streams and services for the hyperconnected world we live in.

JetStream is the next generation streaming platform for NATS, highly resilient, highly
available, and easy to use. We’ve spent a long time listening to our community, learning
from our experiences, looking at the needs of today, and thinking deeply about the needs
of tomorrow. We built JetStream to address these needs.

JetStream:

Get started with JetStream.

```
is easy to deploy and manage, built into the NATS server
simplies and accelerates development
supports wildcard subjects
supports at least once delivery and exactly once within a window
is horizontally scalable at runtime with no interruptions
persists data via streams and delivers or replays via consumers
supports multiple patterns to consume data on the same stream
supports push and pull modes when consuming messages
is account aware
allows for detailed granularity of security, by stream, by consumer, by function
```
#### Next Generation Streaming

#### Security and Simplified Account Management


Account management just became much easier. This version of NATS has a built-in
account management system, eliminating the need to set up an account manager when
not using the memory account resolver. With automated default system account
generation, and the ability to preload accounts, simply enable a set of servers in your
deployment to be account resolvers or account resolver caches, and they will handle
public account information provided to the NATS system through the NATS nsc tooling.
Have enterprise-scale account management up and running in minutes.

By specifying a CIDR block restriction for a user, policy can be applied to limit
connections from clients within a certain range or set of IP addresses. Use this as
another layer of security atop user credentials to better secure your distributed system.
Ensure your applications can only connect from within a specic cloud, enterprise,
geographic location, virtual or physical network.

Scoped to the user, you can now specify a specic block of time during the day when
applications can connect. For example, permit certain users or applications to access the
system during specied business hours, or protect business operations during the
busiest parts of the day from batch driven back-oce applications that could adversely
impact the system when run at the wrong time.

Now you can specify default user permissions within an account. This signicantly
reduces efforts around policy, reduces chances for error in permissioning, and simplies
the provisioning of user credentials.

##### CIDR Block Account Restrictions

##### Time-Based Account Restrictions

##### Default User Permissions

#### WebSockets


Connect mobile and web applications to any NATS server using WebSockets. Built to
more easily traverse r ewalls and load balancers, NATS WebSocket support provides
even more exibility to NATS deployments and makes it easier to communicate to the
edge and endpoints. This is currently supported in NATS server leaf nodes, nats.ts,
nats.deno, and the nats.js clients.

With the Adaptive Edge architecture and the ease with which NATS can extend a cloud
deployment to the edge, it makes perfect sense to leverage existing investments in IoT
deployments. It’s expensive to update devices and large edge deployments. Our goal is to
enable the hyperconnected world, so we added rst-class support for MQTT 3.1.1 directly
into the NATS Server.

Seamlessly integrate existing IoT deployments using MQTT 3.1.1 with a cloud-native
NATS deployment. Add a leaf node that is MQTT enabled and instantly send and receive
messages to your MQTT applications and devices from a NATS deployment whether it be
edge, single-cloud, multi-cloud, on-premise, or any combination thereof.

We’ve added a variety of features to allow you to build a more resilient, secure, and simply
better system at scale.

We’ve added the ability to optionally use headers, following the HTTP semantics familiar
to developers. Headers naturally apply overhead, which was why we resisted adding them
for so long. By creating new internal protocol messages transparent to developers, we
maintain the extremely fast processing of simple NATS messages that we have always
had while supporting headers for those who would like to leverage them. Adding headers
to messages allows you to provide application-specic metadata, such as compression

#### Native MQTT Support

#### Build Better Systems

##### Message Headers


or encryption-related information, without touching the payload. We also provide some
NATS specic headers for use in JetStream and other features.

When taking down a server for maintenance, servers can be signaled to enter Lame Duck
Mode where they do not accept new connections and evict existing connections over a
period of time. Maintainer supported clients will notify applications that a server has
entered this state and will be shutting down, allowing a client to smoothly transition to
another server or cluster and better maintain business continuity during scheduled
maintenance periods.

Why wait for timeouts when services aren’t available? When a request is made to a
service (request-reply) and the NATS Server knows there are no services available the
server will short circuit the request. A “no-responders” protocol message will be sent
back to the requesting client which will break from blocking API calls. This allows
applications to immediately react which further enables building a highly responsive
system at scale, even in the face of application failures and network partitions.

Reduce risk when onboarding new services. Canary deployments, A/B testing, and
transparent teeing of data streams are now fully supported in NATS. The NATS Server
allows accounts to form subject mappings from one subject to another for both client
inbound and service import invocations and allows weighted sets for the destinations.
Map any percentage - 1 to 100 percent of your trac - t o other subjects, and change this
at runtime with a server conguration reload. You can even articially drop a percentage
of trac t o introduce chaos testing into your system. See Conguring Subject Mapping
and Trac Shaping in NATS Server conguration for more details.

##### Seamless Maintenance with Lame Duck Notifications

##### React Quicker with No-Responder Notifications

##### Subject Mapping and Traffic Shaping

##### Account Monitoring - More Meaningful Metrics


NATS now allows for ne-gr ained monitoring to identify usage metrics tied to a particular
account. Inspect messages and bytes sent or received and various connection statistics
for a particular account. Accounts can represent anything - a group of applications, a
team or organization, a geographic location, or even roles. If NATS is enabling your SaaS
solution you could use NATS account scoped metrics to bill users.


### NATS 2.0

NATS 2.0 was the largest feature release since the original code base for the server was
released. NATS 2.0 was created to allow a new way of thinking about NATS as a shared
utility, solving problems at scale through distributed security, multi-tenancy, larger
networks, and secure sharing of data.

NATS 2.0 was created to address problems in large scale distributed computing.

It is dicult at best to combine identity management end-to-end (or end-to-edge), with
data sharing, while adhering to policy and compliance. Current distributed systems
increase signicantly in operational complexity as they scale upward. Problems arise
around service discovery, connectivity, scaling for volume, and application onboarding
and updates. Disaster recovery is dicult, especially as systems have evolved to operate
in silos dened by technology rather than business needs. As complexity increases,
systems become expensive to operate in terms of time and money. They become fragile
making it dicult to deploy services and applications hindering innovation, increasing
time to value and total cost of ownership.

We decided to:

```
Reduce total cost of ownership : Users want reduced TCO for their
distributed systems. This is addressed by an easy to use technology that
can operate at global scale with simple conguration and a resilient
and cloud-native architecture.
Decrease Time to Value : As systems scale, time to value increases.
Operations resist change due to risk in touching a complex and fragile
system. Providing isolation contexts can help mitigate this.
Support manageable large scale deployments : No data silos dened by
software, instead easily managed through software to provide exactly what the
business needs. We wanted to provide easy to congure disaster recovery.
```
#### Rationale


To achieve this, we added a number of new features that are transparent to existing
clients with 100% backward client compatibility.

Accounts are securely isolated communication contexts that allow multi-tenancy
spanning a NATS deployment. Accounts allow users to bifurcate technology from
business driven use cases, where data silos are created by design, not software
limitations. When a client connects, it species an account or will default to
authentication with a global account.

At least some services need to share data outside of their account. Data can be securely
shared between accounts with secure services and streams. Only mutual agreement
between account owners permit data ow, and the import account has complete control
over its own subject space.

This means within an account, limitations may be set and subjects can be used without
worry of collisions with other groups or organizations. Development groups choose any
subjects without affecting the rest of the system, and open up accounts to export or
import only the services and streams they need.

Accounts are easy, secure, and cost effective. There is one NATS deployment to manage,
but organizations and development teams can self manage with more autonomy
reducing time to value with faster, more agile development practices.

Services and streams are mechanisms to share messages between accounts.

```
Decentralize security : Provide security supporting one
technology end-to-end where organizations may self-manage making it
easier to support a massive number of endpoints.
```
#### Accounts

##### Service and Streams


Think of a service as an RPC endpoint into an account. Behind that account there might
be many microservices working in concert to handle requests, but from outside the
account there is simply one subject exposed.

**Service** denitions share an endpoint:

Use cases include most applications - anything that accepts a request and returns a
response.

**Stream** denitions allow continuous data ow between accounts:

Use cases include Observability, Metrics, and Data analytics. Any application or endpoint
reading a stream of data.

Note that services and streams operate with **zero** client conguration or API changes.
Services may even move between accounts, entirely transparent to end clients.

The system account publishes system messages under established subject patterns.
These are internal NATS system messages that may be useful to operators.

Server initiated events and data include:

```
Export a service to allow other accounts to import
Import a service to allow requests to be sent securely and seamlessly to another
account
```
```
Export a stream to allow egress
Import a stream to allow ingress
```
```
Client connection events
Account connection status
Authentication errors
Leaf node connection events
```
##### System Accounts


Tools and clients with proper privileges can request:

Account servers will also publish messages when an account changes.

With this information and system metadata you can build useful monitoring and anomaly
detection tools.

NATS 2.0 supports global deployments, allowing for global topologies that optimize for
WANs while extend to the edge or devices.

While self healing features have been part of NATS 1.X releases, we ensured they
continue to work in global deployments. These include:

```
Server stats summary
```
```
Service statistics
Server discovery and metrics
```
```
Client and server connections automatically reconnect
Auto-Discovery where servers exchange server topology changes with each
other and with clients, in real time with zero conguration changes and
zero downtime while being entirely transparent to clients. Clients can
failover to servers they were not originally congured with.
NATS server clusters dynamically adjust to new or removed servers allowing
for seamless rolling upgrades and scaling up or down.
```
#### Global Deployments

##### Self Healing

##### Superclusters


Conceptually, superclusters are clusters of NATS clusters. Create superclusters to deploy
a truly global NATS network. Superclusters use a novel spline based technology with a
unique approach to topology, keeping one hop semantics and optimizing WAN trac
through optimistic sends with interest graph pruning. Superclusters provide transparent,
intelligent support for geo-distributed queue subscribers.

Superclusters inherently support disaster recovery. With geo-distributed queue
subscribers, local clients are preferred, then an RTT is used to nd the lowest latency
NATS cluster containing a matching queue subscriber in the supercluster.

What does this mean?

Let's say you have a set of load balanced services in US East Coast (US-EAST), another
set in the EU (EU-WEST), and a supercluster consisting of a NATS cluster in US-EAST
connected to a NATS cluster in EU-WEST. Clients in the US would connect to a US-EAST,
and services connected to that cluster would service those clients. Clients in Europe
would automatically use services connected to EU-WEST. If the services in US-EAST
disconnect, clients in US-EAST will begin using services in EU-WEST.

Once the Eastern US services have reconnected to US-EAST, those services will
immediately begin servicing the Eastern US clients since they're local to the NATS cluster.
This is automatic and entirely transparent to the client. There is no extra conguration in
NATS servers.

This is **zero conguration disaster recovery**.

Leaf nodes are NATS servers running in a special conguration, allowing hub and spoke
topologies to extend superclusters.

Leaf nodes can also bridge separate security domains. e.g. IoT, mobile, web. They are
ideal for edge computing, IoT hubs, or data centers that need to be connected to a global

##### Disaster Recovery

##### Leaf Nodes


NATS deployment. Local applications that communicate using the loopback interface
with physical VM or Container security can leverage leaf nodes as well.

Leaf nodes:

NATS 2.0 Security consists of dening Operators, Accounts, and Users within a NATS
deployment.

```
Transparently and securely bind to a remote NATS account
Securely bridge specic local data to a wider NATS deployment
Are 100% transparent to clients which remain simple, lightweight, and easy to develop
Allow for a local security scheme while using new NATS security features globally
Can create a DMZ between a local NATS deployment and external NATS cluster or
supercluster.
```
```
An Operator provides the root of trust for the system, may represent
a company or enterprise
Creates Accounts for account administrators. An account represents
an organization, business unit, or service offering with a secure context
within the NATS deployment, for example an IT system monitoring group, a
set of microservices, or a regional IoT deployment. Account creation
would likely be managed by a central group.
Accounts dene limits and may securely expose services and streams.
Account managers create Users with permissions
Users have specic credentials and permissions.
```
#### Decentralized Security

##### Operators, Accounts, and Users

##### Trust Chain


PKI (NKeys encoded Ed25519) and signed JWTs create a hierarchy of Operators,
Accounts, and Users creating a scalable and exible distributed security mechanism.

This allows for rapid change of permissions, authentication and limits, to a secure multi-
tenant NATS system.

```
Operators are represented by a self signed JWT and is the only thing that
is required to be congured in the server. This JWT is usually signed by a
master key that is kept oine. The JWT will contain valid signing keys that
can be revoked with the master updating this JWT.
Operators will sign Account JWTs with various signing keys.
Accounts sign User JWTs, again with various signing keys.
Clients or leaf nodes present User credentials and a signed nonce when connecting.
The server uses resolvers to obtain JWTs and verify the client trust chain.
```

## NATS Concepts


### Overview

**What is NATS?**
NATS is a connective technology that powers modern distributed systems. A connective
technology is responsible for addressing, discovery and exchanging of messages that
drive the common patterns in distributed systems; asking and answering questions, aka
services/microservices, and making and processing statements, or stream processing.

**Challenges faced by modern distributed systems**
Modern distributed systems are dened by an ever increasing number of hyper-
connected moving parts and the additional data they generate. They employ both
services and streams to drive business value. They are also being dened by location
independence and mobility, and not just for things we would typically recognize as front
end technologies. Today’s systems and the backend processes, microservices and
stream processing are being asked to be location independent and mobile as well, all
while being secure.

These modern systems present challenges to technologies that have been used to
connect mobile front ends to fairly static backends. These incumbent technologies
typically manage addressing and discovery via hostname (DNS) or IP and port, utilize a
1:1 communication pattern, and have multiple different security patterns for
authentication and authorization. Although not perfect, incumbent technologies have
been good enough in many situations, but times are changing quickly. As microservices,
functions, and stream processing are being asked to move to the edge, these
technologies and the assumptions they make are being challenged.

**Effortless M:N connectivity:** NATS manages addressing and discovery based on subjects
and not hostname and ports. Defaulting to M:N communications, which is a superset of
1:1, meaning it can do 1:1 but can also do so much more. If you have a 1:1 system that is
successful in development, ask how many other moving parts are required for production
to work around the assumption of 1:1? Things like load balancers, log systems, and

#### What makes the NATS connective technology

#### unique for these modern systems?


network security models, as well as proxies and sidecars. If your production system
requires all of these things just to get around the fact that the connective technology
being used, e.g. HTTP or gRPC, is 1:1, it’s time to give NATS.io a look.

**Deploy anywhere:** NATS can be deployed nearly anywhere; on bare metal, in a VM, as a
container, inside K8S, on a device, or whichever environment you choose. NATS runs well
within deployment frameworks or without.

**Secure:** Similarly, NATS is secure by default and makes no requirements on network
perimeter security models. When you start considering mobilizing your backend
microservices and stream processors, many times the biggest roadblock is security.

NATS infrastructure and clients communicate all topology changes in real-time. This
means that NATS clients do not need to change when NATS deployments change. Having
to change clients with deployments would be like having to reboot your phone every time
your cell provider added or changed a cell tower. This sounds ridiculous of course, but
think about how many systems today have their front ends tied so closely to the backend,
that any change requires a complete front end reboot or at least a reconguration. NATS
clients and applications need no such change when backend servers are added and
removed and changed. Even DNS is only used to bootstrap rst contact, after that, NATS
handles endpoint locations transparently.

Another advantage to utilizing a NATS is that it allows a hybrid mix of SaaS/Utility
computing with separately owned and operated systems. Meaning you can have a shared
NATS service with core microservices, streams and stream processing be extended by
groups or individuals who have a need to run their own NATS infrastructure. You are not
forced to choose one or the other.

##### Scalable, Future-Proof Deployments

##### Hybrid Deployments

##### Adaptability


Today’s systems will fall short with new demands. As modern systems continue to evolve
and utilize more components and process more data, supporting patterns beyond 1:1
communications, with addressing and discovery tied to DNS is critical. Foundational
technologies like NATS promise the most return on investment. Incumbent technologies
will not work as modern systems unify cloud, Edge, IoT and beyond. NATS does.

NATS can run anywhere, from large servers and cloud instances, through edge gateways
and even IoT devices. Use cases for NATS include:

```
Cloud Messaging
Services (microservices, service mesh)
Event/Data Streaming (observability, analytics, ML/AI)
Command and Control
IoT and Edge
Telemetry / Sensor Data / Command and Control
Augmenting or Replacing Legacy Messaging Systems
```
##### Use Cases


### Compare NATS

NATS Comparison to Kafka, Rabbit, gRPC, and others

This feature comparison is a summary of a few of the major components in several of the
popular messaging technologies of today. This is by no means an exhaustive list and
each technology should be investigated thoroughly to decide which will work best for
your implementation.

In this comparison, we will be featuring NATS, Apache Kafka, RabbitMQ, Apache Pulsar,
and gRPC.

```
Project Client Languages and Platforms
```
```
NATS
```
```
Core NATS: 48 known client types, 11 supported by maintainers,
18 contributed by the community. NATS Streaming: 7
client types supported by maintainers, 4 contributed by the
community. NATS servers can be compiled on architectures
supported by Golang. NATS provides binary distributions.
gRPC 13 client languages.
```
```
Kafka 18 client types supporKafka servers can run on platforms supported across the community and bting java; very wide suppory Conuent.^ t.
```
```
Pulsar 7 client languages, 5 third-party clients - tested on macOS and Linux.
```
```
Rabbit
```
```
At least 10 client platforms that are maintainer-supported
with over 50 community supported client types. Servers are
supported on the following platforms: Linux Windows, NT.
```
#### Language and Platform Coverage

#### Built-in Patterns


```
Project Supported Patterns
```
```
NATS
```
```
Streams and Services through built-in publish/subscribe, request-
reply, and load-balanced queue subscriber patterns. Dynamic
request permissioning and request subject obfuscation is supported.
```
```
gRPC One service, which maBalancing for a service can be done either client-side or by have streaming semantics, per channel. Loady using a proxy.^
```
```
Kafka
```
```
Streams through publish/subscribe. Load balancing can be achieved
with consumer groups. Application code must correlate requests
with replies over multiple topics for a service (request-reply) pattern.
```
```
Pulsar
```
```
Streams through publish/subscribe. Multiple competing consumer
patterns support load balancing. Application code must correlate requests
with replies over multiple topics for a service (request-reply) pattern.
```
```
Rabbit
```
```
Streams through publish/subscribe, and services with a direct
reply-to feature. Load balancing can be achieved with a Work
Queue. Applications must correlate requests with replies
over multiple topics for a service (request-reply) pattern.
```
```
Project Quality of Service / Guarantees
NATS At most once, at least once, and exactly once is available in JetStream.
gRPC At most once.
Kafka At least once, exactly once.
Pulsar At most once, at least once, and exactly once.
Rabbit At most once, at least once.
```
#### Delivery Guarantees

#### Multi-tenancy and Sharing


```
Project Multi-tenancy Support
NATS NATS supporthrough accounts and dening sharts true multi-tenancy and decentred streams and seralized securityvices.^
```
```
gRPC N/A
Kafka Multi-tenancy is not supported.
```
```
Pulsar
```
```
Multi-tenancy is implemented through tenants; built-in data
sharing across tenants is not supported. Each tenant can
have its own authentication and authorization scheme.
Rabbit Multi-tenancy is supported with vhosts; data sharing is not supported.
```
```
Project Authentication
NATS NATS supporED25519 keys), username and passworts TLS, NATS credentials, NKEYd, or simple tS (NATSoken.^
```
```
gRPC TLS, ALT, Token, channel and call credentials, and a plug-in mechanism.
```
```
Kafka Supports Kerberos and TLS. Supporimplementation that uses ZooKeeper to store connection and subject.ts JAAS and an out-of-box authorizer^
```
```
Pulsar TLS Authentication, Athenz, Kerberos, JSON Web Token Authentication.
Rabbit TLS, SASL, username and password, and pluggable authorization.
```
```
Project Authorization
NATS Account limits including number of connections, message
size, number of imports and exports. User-level publish
```
#### AuthN

#### AuthZ


```
Project Authorization
and subscribe permissions, connection restrictions, CIDR address
restrictions, and time of day restrictions.
```
```
gRPC Users can congurne-gr ained individual calls on a sere call credentials tvice.o authorize^
```
```
Kafka Supports JAAS, ACLs for a rich set of Kafka rincluding topics, clusters, groups, and others.esources^
```
```
Pulsar Permissions malists of operations such as pry be granted to specic roduce and consume.oles for^
```
```
Rabbit
```
```
ACLs dictate permissions for congure, write, and
readoperationsonresourceslikeexchangesqueues
```
```
Project Message Retention and Persistence Support
```
```
NATS
```
```
Supports memory and le persistence. Messages can be replayed by time,
count, or sequence number, and durable subscriptions are supported. With
NATS streaming, scripts can archive old log segments to cold storage.
gRPC N/A
```
```
Kafka
```
```
Supports le-based persistence. Messages can be replayed
by specifying an offset, and durable subscriptions are
supported. Log compaction is supported as well as KSQL.
```
```
Pulsar
```
```
Supports tiered storage including le, Amazon S3 or Google
Cloud Storage (GCS). Pulsar can replay messages from a specic
position and supports durable subscriptions. Pulsar SQL and
topic compaction is supported, as well as Pulsar functions.
```
```
Rabbit Supports le-based persistence. Rabbit supporbased semantics (vs log), so no message replay is available.ted queue-
```
#### Message Retention and Persistence


```
Project HA and FT Support
```
```
NATS
```
```
Core NATS supports full mesh clustering with self-healing features
to provide high availability to clients. NATS streaming has warm
failover backup servers with two modes (FT and full clustering).
JetStream supports horizontal scalability with built-in mirroring.
gRPC N/A. gRPC relies on external resources for HA/FT.
Kafka Fully replicated cluster members are coordinated via Zookeeper.
Pulsar Pulsar supports clustered brokers with geo-replication.
```
```
Rabbit Clustering SupporClusters require low-latency networks whert with full data replication via fe network parederation plugins.titions are rare.^
```
```
Project Supported Deployment Models
```
```
NATS
```
```
The NATS network element (server) is a small static binary that can be
deployed anywhere from large instances in the cloud to resource
constrained devices like a Raspberry PI. NATS supports the Adaptive Edge
architecture which allows for large, exible deployments. Single servers, leaf
nodes, clusters, and superclusters (cluster of clusters) can be combined in
any fashion for an extremely exible deployment amenable to cloud, on-
premise, edge and IoT. Clients are unaware of topology and can connect to
any NATS server in a deployment.
```
```
gRPC gRPC is point tmanage, but alwao point and does not hays requires additional pieces for prve a server or broker to deploy oroduction deployments.^
```
```
Kafka
```
```
Kafka supports clustering with mirroring to loosely coupled remote
clusters. Clients are tied to partitions dened within clusters.
Kafka servers require a JVM, eight cores, 64 GB to128 GB of
RAM, two or more 8-TB SAS/SSD disks, and a 10-Gig NIC. (1)__
```
#### High Availability and Fault Tolerance

#### Deployment


```
Project Supported Deployment Models
```
```
Pulsar
```
```
Pulsar supports clustering and built-in geo-replication between
clusters. Clients may connect to any cluster with an appropriately
congured tenant and namespace. Pulsar requires a JVM and
requires at least 6 Linux machines or VMs. 3 running ZooKeeper.
3 running a Pulsar broker and a BookKeeper bookie. (2)__
```
```
Rabbit
```
```
Rabbit supports clusters and cross-cluster message propagation through
a federation plugin. Clients are unaware of topology and may connect
to any cluster. The server requires the Erlang VM and dependencies.
```
```
Project Monitoring Tooling
```
```
NATS
```
```
NATS supports exporting monitoring data to Prometheus and has Grafana
dashboards to monitor and congure alerts. There are also development
monitoring tools such as nats-top. Robust side car deployment or a
simple connect-and-view model with NATS surveyor is supported.
gRPC External components such as a service mesh are required to monitor gRPC.
```
```
Kafka Kafka has a number of management tConuent Control Center, Kafka, Kafka Wools and consoles includingeb Console, Kafka Offset Monit^ or.
```
```
Pulsar CLI tools, per-topic dashboards, and third-party tools.
```
```
Rabbit CLI tools, a plugin-based managementsystem with dashboards and third-party tools.^
```
```
Project Management Tooling
```
```
NATS
```
```
NATS separates operations from security. User and Account management
in a deployment may be decentralized and managed through a CLI. Server
(network element) conguration is separated from security with a command
line and conguration le which can be reloaded with changes at runtime.
```
#### Monitoring

#### Management


```
Project Management Tooling
gRPC External components such as a service mesh are required to manage gRPC.
```
```
Kafka Kafka has a number of management tConuent Control Center, Kafka, Kafka Wools and consoles includingeb Console, Kafka Offset Monit^ or.
```
```
Pulsar CLI tools, per-topic dashboards, and third-party tools.
```
```
Rabbit CLI tools, a plugin-based managementsystem with dashboards and third-party tools.^
```
```
Project Built-in and Third Party Integrations
```
```
NATS
```
```
NATS supports WebSockets, a Kafka bridge, an IBM MQ
Bridge, a Redis Connector, Apache Spark, Apache Flink,
CoreOS, Elastic, Elasticsearch, Prometheus, Telegraf, Logrus,
Fluent Bit, Fluentd, OpenFAAS, HTTP, and MQTT, and more.
```
```
gRPC There are a number of thirHTTP, JSON, Prometheus, Grift and others. d party integrations including(3)__^
```
```
Kafka
```
```
Kafka has a large number of integrations in its ecosystem,
including stream processing (Storm, Samza, Flink), Hadoop,
database (JDBC, Oracle Golden Gate), Search and Query
(ElasticSearch, Hive), and a variety of logging and other integrations.
```
```
Pulsar Pulsar has many integrDebezium, Flume, Elasticsearations, including Activch, Kafka, Redis, and others.eMQ, Cassandra,^
```
```
Rabbit RabbitMQ has many plugins, including prWebSockets, and various authorization and authentication plugins.otocols (MQTT, STOMP),^
```
#### Integrations

#### References


1. https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.1.0/bk_planning-your-
    deployment/content/ch_hardware-sizing.html
2. https://pulsar.apache.org/docs/v1.21.0-incubating/deployment/cluster/
3. https://github.com/grpc-ecosystem


### What is NATS

Software applications and services need to exchange data. NATS is an infrastructure that
allows such data exchange, segmented in the form of messages. We call this a " **message
oriented middleware** ".

With NATS, application developers can:

Developers use one of the NATS client libraries in their application code to allow them to
publish, subscribe, request and reply between instances of the application or between
completely separate applications. Those applications are generally referred to as 'client
applications' or sometimes just as 'clients' throughout this manual (since from the point
of view of the NATS server, they are clients).

The NATS services are provided by one or more NATS server processes that are
congured to interconnect with each other and provide a NATS service infrastructure. The
NATS service infrastructure can scale from a single NATS server process running on an
end device (the nats-server process is less than 20 MB in size!) all the way to a public
global super-cluster of many clusters spanning all major cloud providers and all regions
of the world such as Synadia's NGS.

To connect a NATS client application with a NATS service, and then subscribe or publish
messages to subjects, it only needs to be congured with:

```
Effortlessly build distributed and scalable client-server applications.
Store and distribute data in realtime in a general manner. This can exibly be achieved
across various environments, languages, cloud providers and on-premises systems.
```
##### NATS Client Applications

##### NATS Service Infrastructure

##### Connecting NATS Client applications to the NATS servers


1. **URL:** A 'NATS URL'. This is a string (in a URL format) that species the IP address and
    port where the NATS server(s) can be reached, and what kind of connection to
    establish (plain TCP, TLS, or Websocket).
2. **Authentication** (if needed): Authentication details for the application to identify itself
    with the NATS server(s). NATS supports multiple authentication schemes
    (username/password, decentralized JWT, token, TLS certicates and Nkey with
    challenge).

NATS makes it easy for applications to communicate by sending and receiving
messages. These messages are addressed and identied by subject strings, and do not
depend on network location.

Data is encoded and framed as a message and sent by a publisher. The message is
received, decoded, and processed by one or more subscribers.

With this simple design, NATS lets programs share common message-handling code,
isolate resources and interdependencies, and scale by easily handling an increase in
message volume, whether those are service requests or stream data.

NATS offers multiple qualities of service, depending on whether the application uses just
the Core NATS functionality or also leverages the added functionalities enabled by NATS

###### Application 1

###### NATS Publisher

###### Application 3

###### NATS Subscriber

###### Application 2

#### Simple messaging design

##### NATS Quality of service (QoS)


JetStream (JetStream is built into nats-server but may not be enabled on all service
infrastructures).

```
At most once QoS: Core NATS offers an at most once quality of service. If a
subscriber is not listening on the subject (no subject match), or is not active when the
message is sent, the message is not received. This is the same level of guarantee that
TCP/IP provides. Core NATS is a r e-and-forget messaging system. It will only hold
messages in memory and will never write messages directly to disk.
At-least / exactly once QoS: If you need higher qualities of service ( at least once and
exactly once ), or functionalities such as persistent streaming, de-coupled ow control,
and Key/Value Store, you can use NATS JetStream, which is built in to the NATS server
(but needs to be enabled). Of course, you can also always build additional reliability
into your client applications yourself with proven and scalable reference designs such
as acks and sequence numbers.
```

### Walkthrough Setup

We have provided Walkthroughs for you to try NATS (and JetStream) on your own. In
order to follow along with the walkthroughs, you could choose one of these options:

For MacOS:

For Arch Linux:

For other versions of Linux and for Windows:
The .deb or .rpm les and Windows binaries (even for ARM) are available here GitHub
Releases.

If you are going to run a server locally you need to rst install it and start it. Alternatively if
you already know how to use NATS on a remote server, you only need to pass the server
URL to nats using the -s option or preferably create a context using

```
The nats CLI tool must be installed, and a local NATS server must be installed (or
you can use a remote server you have access to).
You can use Synadia's NGS.
You could even use the demo server from where you installed NATS. This is accessible
via nats://demo.nats.io (this is a NATS connection URL; not a browser URL. You
pass it to a NATS client application).
```
```
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
```
```
yay natscli
```
#### Installing the nats CLI Tool

#### Installing the NATS server locally (if needed)


nats context add, to specify the server URL(s) and credentials le containing your
user JWT.

On Mac OS:

On Windows:

On Arch Linux:

For other versions of Linux or other architectures, you can install a release build as shown
below.

You can nd the latest release of nats-server here.

You could manually download the zip le matching your systems architecture, and unzip
it. You could also use curl to download a specic version. The example below shows
for example, how to download version 2.6.2 of the nats-server for Linux AMD64:

```
brew install nats-server
```
```
choco install nats-server
```
```
yay nats-server
```
```
curl -L https://github.com/nats-io/nats-server/releases/download/v2.6.5/n
```
```
unzip nats-server.zip -d nats-server
```
##### Installing the NATS server via a Package Manager

##### Downloading a Release Build


and nally, copy it to the bin folder (this allows you to run the executable from
anywhere in the system):

To start a simple demonstration server locally, simply run:

(or nats-server -m 8222 if you want to enable the HTTP monitoring functionality)

When the server starts successfully, you will see the following messages:

The NATS server listens for client connections on TCP Port 4222.

```
Archive: nats-server.zip
creating: nats-server-v2.6.2-linux-amd64/
inflating: nats-server-v2.6.2-linux-amd64/README.md
inflating: nats-server-v2.6.2-linux-amd64/LICENSE
inflating: nats-server-v2.6.2-linux-amd64/nats-server
```
```
sudo cp nats-server/nats-server-v2.6.2-linux-amd64/nats-server /usr/bin
```
```
nats-server
```
```
[14524] 2021/10/25 22:53:53.525530 [INF] Starting nats-server
[14524] 2021/10/25 22:53:53.525640 [INF] Version: 2.6.1
[14524] 2021/10/25 22:53:53.525643 [INF] Git: [not set]
[14524] 2021/10/25 22:53:53.525647 [INF] Name: NDAUZCA4GR3FPBX4IFLB
[14524] 2021/10/25 22:53:53.525650 [INF] ID: NDAUZCA4GR3FPBX4IFLB
[14524] 2021/10/25 22:53:53.526392 [INF] Starting http monitor on 0.0.0.0
[14524] 2021/10/25 22:53:53.526445 [INF] Listening for client connections
[14524] 2021/10/25 22:53:53.526684 [INF] Server is ready
```
##### Start the NATS server (if needed)


### Subject-Based Messaging

NATS is a system for publishing and listening for messages on named communication
channels we call Subjects. Fundamentally, NATS is an interest-based messaging
system, where the listener has to subscribe to a subset of subjects.

In other middleware systems subjects may be called topics, channels, streams
(Note that in NATS the term stream is used for a Jetstream message storage).

**What is a subject?**
At its simplest, a subject is just a string of characters that form a name the publisher and
subscriber can use to nd each other. More commonly subject hierarchies are used to
scope messages into semantic namespaces.

```
Please check the constraint and conventions on naming for subjects here.
```
**Location transparency** Through subject-based addressing, NATS provides location
transparency across a (large) cloud of routed NATS servers.

```
Subject subscriptions are automatically propagated within the server cloud.
Messages will be automatically routed to all interested subscribers, independent of
location.
Messages with no subscribers to their subject are automatically discarded (Please see
the JetStream feature for message persistency).
```
###### PUB time.us msg nats-server

###### msg SUB time.us

###### SUB time.us

###### msg

#### Wildcards


NATS provides two wildcards that can take the place of one or more elements in a dot-
separated subject. Publishers will always send a message to a fully specied subject,
without the wildcard. While subscribers can use these wildcards to listen to multiple
subjects with a single subscription.

The. character is used to create a subject hierarchy. For example, a world clock
application might dene the following to logically group related subjects:

There is no hard limit to subject size, but it is recommended to keep the maximum
number of tokens in your subjects to a reasonable value. E.g. a maximum of 16 tokens
and the subject length to less than 256 characters.

NATS can manage 10s of millions of subjects eciently, therefore, you can use ne-
grained addressing for your business entities. Subjects are ephemeral resources, which
will disappear when no longer subscribed to.

Still, subject subscriptions need to be cached by the server in memory. Consider when
increasing your subscribed subject count to more than one million you will need more
than 1GB of server memory and it will grow linearly from there.

```
time.us
time.us.east
time.us.east.atlanta
time.eu.east
time.eu.warsaw
```
#### Subject hierarchies

#### Subject usage best practices

##### Number of subjects

##### Subject-based filtering and security


The message subject can be ltered with various means and through various
conguration elements in your NATS server cluster. For example, but not limited to:

A well-designed subject hierarchy will make the job a lot easier for those tasks.

```
There are only two hard problems in computer science: cache invalidation, naming things,
and off-by-one errors. -- Unknown author
```
A subject hierarchy is a powerful tool for addressing your application resources. Most
NATS users therefore encode business semantics into the subject name. You are free to
choose a structure t for your purpose, but you should refrain from over-complicating
your subject design at the start of the project.

**Some guidelines:**

```
Security - allow/deny per user
Import/export between accounts
Automatic transformations
When inserting messages into JetStream streams
When sourcing/mirroring JetStream streams
When connecting leaf nodes (NATS edge servers)
...
```
```
Use the rst token(s) to establish a general namespace.
```
```
factory1.tools.group42.unit17
```
```
Use the nal token(s)for identiers
```
```
service.deploy.server-acme.app123
```
```
A subject should be used for more than one message.
```
##### Naming things


Pragmatic:

Maybe not so useful:

The rst wildcard is * which will match a single token. For example, if an application
wanted to listen for eastern time zones, they could subscribe to time.*.east, which
would match time.us.east and time.eu.east. Note that * can not match a
substring within a token time.New*.east will

```
Subscriptions should be stable (exist for receiving more than one message.
Use wildcard subscriptions over subscribing to individual subjects whenever feasible.
Name business or physical entities. Refrain from encoding too much data into the
subject.
Encode (business) intent into the subject, not technical details.
```
```
orders.online.store123.order171711
```
```
orders.online.us.server42.ccpayment.premium.store123.electronics.deliver-
```
```
NATS messages support headers. These can be used for additional metadata. There
are subscription modes, which deliver headers only, allowing for ecient scanning of
metadata in the message ow.
```
###### PUB time.us.east msg nats-server

###### msg SUB time.*.east

###### SUB time.us.east

###### msg

##### Matching a single token

##### Matching multiple tokens


The second wildcard is > which will match one or more tokens, and can only appear at
the end of the subject. For example, time.us.> will match time.us.east and
time.us.east.atlanta, while time.us.* would only match time.us.east since it
can't match more than one token.

Subject to your security conguration, wildcards can be used for monitoring by creating
something called a wire tap. In the simplest case, you can create a subscriber for >.
This application will receive all messages -- again, subject to security settings -- sent on
your NATS cluster.

The wildcard * can appear multiple times in the same subject. Both types can be used
as well. For example, *.*.east.> will receive time.us.east.atlanta.

For compatibility across clients and ease of maintaining conguration les, we
recommend using alphanumeric characters, - (dash) and _ (underscore) ASCII
characters for subject and other entity names created by the user.

```
PUB time.us.east.atlanta msg nats-server
```
```
SUB time.us.east.atlanta
msg
SUB time.us.*
```
```
SUB time.us.>
```
```
msg
```
##### Monitoring and wire taps

##### Mixing wildcards

#### Characters allowed and recommended for

#### subject names


UTF-8 (UTF8) characters are supported in subjects. Please use UTF-8 characters at your
own risk. Using multilingual names for technical entities can create many issues for
editing, conguration les, display, and cross-border collaboration.

The rules and recommendations here apply to ALL system names, subjects, streams,
durables, buckets, keys (in key-value stores), as NATS will create API subjects that
contain those names. NATS will enforce these constraints in most cases, but we
recommend not relying on this.

Good names

Deprecated subject names

Forbidden stream names

```
Allowed characters : Any Unicode character except null, space,. , * and >
Recommended characters: ( a - z ), ( A - Z ), ( 0 - 9 ), - amd _ (names
are case sensitive, and cannot contain whitespace).
Naming Conventions If you want to delimit words, use either CamelCase as in
MyServiceOrderCreate or - and _ as in my-service-order-create
Special characters: The period. (which is used to separate the tokens in the
subject) and * and also > (the * and > are used as wildcards) are reserved
and cannot be used.
Reserved names: By convention subject names starting with a $ are reserved for
system use (e.g. subject names starting with $SYS or $JS or $KV, etc...). Many
system subjects also use _ (underscore) (e.g. _INBOX , KV_ABC, OBJ_XYZ etc.)
```
```
time.us
time.us2.east1
time.new-york
time.SanFrancisco
```
```
location.Malmö
$location.Stockholm
_Subjects_.mysubject
```

By default, for the sake of eciency, subject names are not veried during message
publishing. In particular when generating subjects programmatically, this will result in
illegal subjects which cannot be subscribed to. E.g. subjects containing wildcards may be
ignored.

To enable subject name verication, activate pedantic mode in the client connection
options.

```
all*data
<my_stream>
service.stream.1
```
```
//Java
Options options = Options.builder()
.server("nats://127.0.0.1:4222")
.pedantic()
.build();
Connection nc = Nats.connect(options)
```
##### Pedantic mode


### Core NATS

Core NATS is the foundational functionality in a NATS system. It operates on a publish-
subscribe model using subject/topic-based addressing. This model offers two signicant
advantages: location independence and a default many-to-many (M:N) communication
pattern. These fundamental concepts enable powerful and innovative solutions for
common development patterns, such as microservices, without requiring additional
technologies like load balancers, API gateways, or DNS conguration.

NATS systems can be enhanced with JetStream, which adds persistence capabilities.
While Core NATS provides best-effort, at-most-once message delivery, JetStream
introduces at-least-once and exactly-once semantics.


### Publish-Subscribe

NATS implements a publish-subscribe message distribution model for one-to-many
communication. A publisher sends a message on a subject and any active subscriber
listening on that subject receives the message. Subscribers can also register interest in
wildcard subjects that work a bit like a regular expression (but only a bit). This one-to-
many pattern is sometimes called a fan-out.

Messages are composed of:

1. A subject.
2. A payload in the form of a byte array.
3. Any number of header elds.
4. An optional 'reply' address eld.

Messages have a maximum size (which is set in the server conguration with
max_payload). The size is set to 1 MB by default, but can be increased up to 64 MB if
needed (though we recommend keeping the max message size to something more
reasonable like 8 MB).

###### Publisher msg1 Subject

###### Subscriber

###### msg1

###### msg1 Subscriber

###### Subscriber

###### msg1

#### Publish-Subscribe

#### Messages


### Pub/Sub Walkthrough

NATS is a publish subscribe messaging system based on subjects. Subscribers listening
on a subject receive messages published on that subject. If the subscriber is not actively
listening on the subject, the message is not received. Subscribers can use the wildcard
tokens such as * and > to match a single token or to match the tail of a subject.

This simple walkthrough demonstrates some ways in which subscribers listen on
subjects, and publishers send messages on specic subjects.

If you have not already done so, you need to install the nats CLI Tool and optionally the
nats-server on your machine.

In a shell or command prompt session, start a client subscriber program.

Here, <subject> is a subject to listen on. It helps to use unique and well thought-
through subject strings because you need to ensure that messages reach the correct

```
com.msg.oneSUB com.msg.onePUB
```
```
NATS
```
```
Non-ActiveSubscriber
```
```
com.msg.oneSUB com.msg.twoSUB com.msg.*SUB
```
```
nats sub <subject>
```
#### NATS Pub/Sub Walkthrough

##### Walkthrough prerequisites

**1. Create Subscriber 1**


subscribers even when wildcards are used.

For example:

You should see the message: Listening on [msg.test]

In another shell or command prompt, create a NATS publisher and send a message.

Where <subject> is the subject name and <message> is the text to publish.

For example:

You'll notice that the publisher sends the message and prints: Published [msg.test] :
'NATS MESSAGE'.

The subscriber receives the message and prints: [#1] Received on [msg.test]: 'NATS
MESSAGE'.

If the receiver does not get the message, you'll need to check if you are using the same
subject name for the publisher and the subscriber.

```
nats sub msg.test
```
```
nats pub <subject> <message>
```
```
nats pub msg.test "NATS MESSAGE"
```
```
nats pub msg.test "NATS MESSAGE 2"
```
**2. Create a Publisher and publish a message
3. Verify message publication and receipt
4. Try publishing another message**


You'll notice that the subscriber receives the message.
Note that a message count is incremented each time your subscribing client receives a
message on that subject.

In a new shell or command prompt, start a new NATS subscriber.

Verify that both subscribing clients receive the message.

In a new shell or command prompt session, create a new subscriber that listens on a
different subject.

Subscriber 1 and Subscriber 2 receive the message, but Subscriber 3 does not. Why?
Because Subscriber 3 is not listening on the message subject used by the publisher.

Change the last subscriber to listen on msg.* and run it:

```
nats sub msg.test
```
```
nats pub msg.test "NATS MESSAGE 3"
```
```
nats sub msg.test.new
```
```
nats pub msg.test "NATS MESSAGE 4"
```
```
nats sub msg.*
```
**5. Create Subscriber 2
6. Publish another message using the publisher client
7. Create Subscriber 3
8. Publish another message
9. Alter Subscriber 3 to use a wildcard**


Note: NATS supports the use of wildcard characters for message subscribers only. You
cannot publish a message using a wildcard subject.

This time, all three subscribing clients should receive the message.

Do try out a few more variations of substrings and wildcards to test your understanding.

Publish-subscribe pattern with the NATS CLI

```
nats pub msg.test "NATS MESSAGE 5"
```
```
Publish-subscribe pattern - NATS CLI
```
```
Publish-Subscribe PPublish-Subscribe Pattern using the new Nattern using the new NATS CLIATS CLI
```
**10. Publish another message**

#### See Also


### Request-Reply

Request-Reply is a common pattern in modern distributed systems. A request is sent, and
the application either waits on the response with a certain timeout, or receives a
response asynchronously.

The increased complexity of modern systems necessitates features like location
transparency, scale-up and scale-down, observability (measuring a system's state based
on the data it generates) and more. In order to implement this feature-set, various other
technologies needed to incorporate additional components, sidecars (processes or
services that support the primary application) and proxies. NATS on the other hand,
implemented Request-Reply much more easily.

```
NATS supports the Request-Reply pattern using its core communication mechanism —
publish and subscribe. A request is published on a given subject using a reply subject.
Responders listen on that subject and send responses to the reply subject. Reply
subjects are called " inbox ". These are unique subjects that are dynamically directed
back to the requester, regardless of the location of either party.
Multiple NATS responders can form dynamic queue groups. Therefore, it's not
necessary to manually add or remove subscribers from the group for them to start or
stop being distributed messages. It’s done automatically. This allows responders to
scale up or down as per demand.
NATS applications "drain before exiting" (processing buffered messages before
closing the connection). This allows the applications to scale down without dropping
requests.
Since NATS is based on publish-subscribe, observability is as simple as running
another application that can view requests and responses to measure latency, watch
for anomalies, direct scalability and more.
The power of NATS even allows multiple responses, where the rst response is utilized
and the system eciently discards the additional ones. This allows for a sophisticated
pattern to have multiple responders, reduce response latency and jitter.
```
##### NATS makes Request-Reply simple and powerful

##### The pattern


Try NATS request-reply on your own, using a live server by walking through the request-
reply walkthrough.

When a request is sent to a subject that has no subscribers, it can be convenient to know
about it right away. For this use-case, a NATS client can opt-into no_responder messages.
This requires a server and client that support headers. When enabled, a request sent to a
subject with no subscribers will immediately receive a reply that has no body, and a 503
status.

Most clients will represent this case by raising or returning an error. For example:

###### Publisher

###### Subject

###### msg1

###### Reply

###### Subscriber

###### msg1

###### msg1 Subscriber

###### Subscriber

###### msg1

```
m, err := nc.Request("foo", nil, time.Second);
# err == nats.ErrNoResponders
```
##### No responders


### Request-Reply Walkthrough

NATS supports request-reply messaging. In this tutorial you explore how to exchange
point-to-point messages using NATS.

If you have not already done so, you need to install the nats CLI Tool and optionally, the
nats-server on your machine.

Start two terminal sessions. These will be used to run the NATS request and reply clients.

You should see the message: Listening on [help.please]

This means that the NATS receiver client is listening for request messages on the
"help.please" subject. In NATS, the receiver is a subscriber.

The NATS requestor client makes a request by sending the message "I need help!" on the
“help.please” subject.

```
nats reply help.please 'OK, I CAN HELP!!!'
```
```
nats request help.please 'I need help!'
```
#### Prerequisites

#### Walkthrough

##### In one terminal, run the reply client listener

##### In the other terminal, run the request client


The NATS receiver client receives the message, formulates the reply ("OK, I CAN HELP!!!"),
and sends it to the inbox of the requester.


### Queue Groups

When subscribers register themselves to receive messages from a publisher, the 1:N fan-
out pattern of messaging ensures that any message sent by a publisher, reaches all
subscribers that have registered. NATS provides an additional feature named "queue",
which allows subscribers to register themselves as part of a queue. Subscribers that are
part of a queue, form the "queue group".

As an example, consider message delivery occurring in the 1:N pattern to all subscribers
based on the subject name (delivery happens even to subscribers that are not part of a
queue group). If a subscriber is registered based on a queue name, it will always receive
messages it is subscribed to, based on the subject name. However, if more subscribers
are added to the same queue name, they become a queue group, and only one randomly
chosen subscriber of the queue group will consume a message each time a message is
received by the queue group. Such distributed queues are a built-in load balancing feature
that NATS provides.

**Advantages**

Queue group names follow the same naming rules as subjects. Foremost, they are case
sensitive and cannot contain whitespace. Consider structuring queue groups
hierarchically using a period.. Some server functionalities can use wildcard matching
on them.

Queue subscribers are ideal for scaling services. Scale up is as simple as running another
application, scale down is terminating the application with a signal that drains the in ight

```
Ensures application fault tolerance
Workload processing can be scaled up or down
No extra conguration required
Queue groups are dened by the application and their queue subscribers, rather than
the server conguration
```
#### How queue groups function


requests. This exibility and lack of any conguration changes makes NATS an excellent
service communication technology that can work with all platform technologies.

When a request is made to a service (request/reply) and the NATS Server knows there are
no services available (since there are no client applications currently subscribing to the
subject in a queue-group) the server will send a “no-responders” protocol message back
to the requesting client which will break from blocking API calls. This allows applications
to react immediately. This further enables building a highly responsive system at scale,
even in the face of application failures and network partitions.

With JetStream a stream can also be used as a queue by setting the retention policy to
WorkQueuePolicy and leveraging pull consumers to get easy horizontal scalability of
the processing (or using an explicit ack push consumer with a queue group of
subscribers).

When connecting to a globally distributed NATS super-cluster, there is an automatic
service geo-anity due to the fact that a service request message will only be routed to
another cluster (i.e. another region) if there are no listeners on the cluster available to
handle the request locally.

###### Publisher msgs 1,2,3 Queue

###### Subscriber

###### msg 2

###### msg 1 Subscriber

###### Subscriber

###### msg 3

##### No responder

#### Stream as a queue

##### Queuing geo-affinity


Try NATS queue subscriptions on your own, using a live server by walking through the
queueing walkthrough.

##### Tutorial


### Queueing Walkthrough

NATS supports a form of load balancing using queue groups. Subscribers register a
queue group name. A single subscriber in the group is randomly selected to receive the
message.

If you have not already done so, you need to install the nats CLI Tool and optionally the
nats-server on your machine.

The nats reply instances don't just subscribe to the subject but also automatically join
a queue group ("NATS-RPLY-22" by default)

In a new window

In a new window

```
nats reply foo "service instance A Reply# {{Count}}"
```
```
nats reply foo "service instance B Reply# {{Count}}"
```
```
nats reply foo "service instance C Reply# {{Count}}"
```
#### Walkthrough prerequisites

##### 1. Start the first member of the queue group

##### 2. Start a second member of the queue-group

##### 3. Start a third member of the queue-group


You should see that only one of the my-queue group subscribers receives the message
and replies it, and you can also see which one of the available queue-group subscribers
processed the request from the reply message received (i.e. service instance A, B or C)

You should see that a different queue group subscriber receives the message this time,
chosen at random among the 3 queue group members.

You can also send any number of requests back-to-back. From the received messages,
you'll see the distribution of those requests amongst the members of the queue-group.
For example: nats request foo --count 10 "Request {{Count}}"

You can at any time start yet another service instance, or kill one and see how the queue-
group automatically takes care of adding/removing those instances from the group.

Queue groups using the NATS CLI

```
nats request foo "Simple request"
```
```
nats request foo "Another simple request"
```
##### 4. Publish a NATS message

##### 5. Verify message publication and receipt

##### 6. Publish another message

##### 7. Stop/start queue-group members

#### See Also


```
Queue Groups NATS CLI
```
Publish-Subscribe PPublish-Subscribe Pattern using the new Nattern using the new NATS CLIATS CLI


### JetStream

NATS has a built-in persistence engine called JetStream which enables messages to be
stored and replayed at a later time. Unlike NATS Core which requires you to have an
active subscription to process messages as they happen, JetStream allows the NATS
server to capture messages and replay them to consumers as needed. This functionality
enables a different quality of service for your NATS messages, and enables fault-tolerant
and high-availability congurations.

JetStream is built into nats-server. If you have a cluster of JetStream-enabled servers
you can enable data replication and thus guard against failures and service disruptions.

JetStream was created to address the problems identied with streaming technology
today - complexity, fragility, and a lack of scalability. Some technologies address these
better than others, but no current streaming technology is truly multi-tenant, horizontally
scalable, or supports multiple deployment models. No other technology that we are
aware of can scale from edge to cloud using the same security context while having
complete deployment observability for operations.

The JetStream persistence layer enables additional use cases typically not found in
messaging systems. Being built on top of JetStream they inherit the core capabilities of
JetStream, replication, security, routing limits, and mirroring.

```
Key Value Store A map (associative array) with atomic operations
Object Store File transfer, replications and storage API. Uses chunked transfers for
scalability.
```
#### JetStream

##### JetStream

**Additional capabilities enabled by JetStream**


Key/Value and File transfer are capabilities are commonly found in in-memory databases
or deployment tools. While NATS does not intend to compete with the feature set of such
tools, it is our goal to provide the developer with reasonable complete set of data storage
and replications features for use cases like micro service, edge deployments and server
management.

To congure a nats-server with JetStream refer to:

For runnable JetStream code examples, refer to NATS by Example.

JetStream was developed with the following goals in mind:

```
Conguring JetStream
JetStream Clustering
```
```
The system must be easy to congure and operate and be observable.
The system must be secure and operate well with NATS 2.0 security models.
The system must scale horizontally and be applicable to a high ingestion rate.
The system must support multiple use cases.
The system must self-heal and always be available.
The system must allow NATS messages to be part of a stream as desired.
The system must display payload agnostic behavior.
The system must not have third party dependencies.
```
**Configuration**

**Examples**

**Goals**

##### JetStream capabilities

**Streaming: temporal decoupling between the publishers and subscribers**


One of the tenets of basic publish/subscribe messaging is that there is a required
temporal coupling between the publishers and the subscribers: subscribers only receive
the messages that are published when they are actively connected to the messaging
system (i.e. they do not receive messages that are published while they are not
subscribing or not running or disconnected). The traditional way for messaging systems
to provide temporal decoupling of the publishers and subscribers is through the 'durable
subscriber' functionality or sometimes through 'queues', but neither one is perfect:

However, in many use cases, you do not need to 'consume exactly once' functionality but
rather the ability to replay messages on demand, as many times as you want. This need
has led to the popularity of some 'streaming' messaging platforms.

JetStream provides both the ability to consume messages as they are published (i.e.
'queueing') as well as the ability to replay messages on demand (i.e. 'streaming'). See
retention policies below.

**Replay policies**

JetStream consumers support multiple replay policies, depending on whether the
consuming application wants to receive either:

```
durable subscribers need to be created before the messages get published
queues are meant for workload distribution and consumption, not to be used as a
mechanism for message replay.
```
```
all of the messages currently stored in the stream, meaning a complete 'replay' and
you can select the 'replay policy' (i.e. the speed of the replay) to be either:
instant (meaning the messages are delivered to the consumer as fast as it can take
them).
original (meaning the messages are delivered to the consumer at the rate they were
published into the stream, which can be very useful for example for staging
production trac).
the last message stored in the stream, or the last message for each subject (as
streams can capture more than one subject).
starting from a specic sequence number.
starting from a specic start time.
```

**Retention policies and limits**

JetStream enables new functionalities and higher qualities of service on top of the base
'Core NATS' functionality. However, practically speaking, streams can't always just keep
growing 'forever' and therefore JetStream supports multiple retention policies as well as
the ability to impose size limits on streams.

**Limits**

You can impose the following limits on a stream

You must also select a **discard policy** which species what should happen once the
stream has reached one of its limits and a new message is published:

**Retention policy**

You can choose what kind of retention you want for each stream:

```
Maximum message age.
Maximum total stream size (in bytes).
Maximum number of messages in the stream.
Maximum individual message size.
You can also set limits on the number of consumers that can be dened for the stream
at any given point in time.
```
```
discard old means that the stream will automatically delete the oldest message in the
stream to make room for the new messages.
discard new means that the new message is discarded (and the JetStream publish call
returns an error indicating that a limit was reached).
```
```
limits (the default) is to provide a replay of messages in the stream.
work queue (the stream is used as a shared queue and messages are removed from it
as they are consumed) is to provide the exactly-once consumption of messages in the
stream.
interest (messages are kept in the stream for as long as there are consumers that
haven't delivered the message yet) is a variation of work queue that only retains
```

Note that regardless of the retention policy selected, the limits (and the discard policy)
always apply.

**Subject mapping transformations**

JetStream also enables the ability to apply subject mapping transformations to
messages as they are ingested into a stream.

You can choose the durability as well as the resilience of the message storage according
to your needs.

JetStream uses a NATS optimized RAFT distributed quorum algorithm to distribute the
persistence service between NATS servers in a cluster while maintaining immediate
consistency (as opposed to eventual consistency) even in the face of failures.

For writes (publications to a stream), the formal consistency model of NATS JetStream is
Linearizable. On the read side (listening to or replaying messages from streams) the
formal models don't really apply because JetStream does not support atomic batching of
multiple operations together (so the only kind of 'transaction' is the persisting, replicating
and voting of a single operation on the stream) but in essence, JetStream is serializable
because messages are added to a stream in one global order (which you can control
using compare and publish).

JetStream can also provide encryption at rest of the messages being stored.

In JetStream the conguration for storing messages is dened separately from how they
are consumed. Storage is dened in a Stream and consuming messages is dened by
multiple Consumers.

```
messages if there is interest (consumers currently dened on the stream) for the
message's subject.
```
```
Memory storage.
File storage.
Replication (1 (none), 2, 3) between nats servers for Fault Tolerance.
```
**Persistent and Consistent distributed storage**


**Stream replication factor**

A stream's replication factor (R, often referred to as the number 'Replicas') determines
how many places it is stored allowing you to tune to balance risk with resource usage and
performance. A stream that is easily rebuilt or temporary might be memory-based with a
R=1 and a stream that can tolerate some downtime might be le-based R-1.

Typical usage to operate in typical outages and balance performance would be a le-
based stream with R=3. A highly resilient, but less performant and more expensive
conguration is R=5, the replication factor limit.

Rather than defaulting to the maximum, we suggest selecting the best option based on
the use case behind the stream. This optimizes resource usage to create a more resilient
system at scale.

**Mirroring and Sourcing between streams**

JetStream also allows server administrators to easily mirror streams, for example
between different JetStream domains in order to offer disaster recovery. You can also
dene a stream that 'sources' from one or more other streams.

JetStream provides decoupled ow control over streams, the ow control is not 'end to
end' where the publisher(s) are limited to publish no faster than the slowest of all the

```
Replicas=1 - Cannot operate during an outage of the server servicing the stream.
Highly performant.
Replicas=2 - No signicant benet at this time. We recommend using Replicas=3
instead.
Replicas=3 - Can tolerate the loss of one server servicing the stream. An ideal balance
between risk and performance.
Replicas=4 - No signicant benet over Replicas=3 except marginally in a 5 node
cluster.
Replicas=5 - Can tolerate simultaneous loss of two servers servicing the stream.
Mitigates risk at the expense of performance.
```
**De-coupled flow control**


consumers (i.e. the lowest common denominator) can receive but is instead happening
individually between each client application (publishers or consumers) and the nats
server.

When using the JetStream publish calls to publish to streams there is an
acknowledgment mechanism between the publisher and the NATS server, and you have
the choice of making synchronous or asynchronous (i.e. 'batched') JetStream publish
calls.

On the subscriber side, the sending of messages from the NATS server to the client
applications receiving or consuming messages from streams is also ow controlled.

Because publications to streams using the JetStream publish calls are acknowledged by
the server the base quality of service offered by streams is 'at least once', meaning that
while reliable and normally duplicate free there are some specic failure scenarios that
could result in a publishing application believing (wrongly) that a message was not
published successfully and therefore publishing it again, and there are failure scenarios
that could result in a client application's consumption acknowledgment getting lost and
therefore in the message being re-sent to the consumer by the server. Those failure
scenarios while being rare and even dicult to reproduce do exist and can result in
perceived 'message duplication' at the application level.

Therefore, JetStream also offers an 'exactly once' quality of service. For the publishing
side, it relies on the publishing application attaching a unique message or publication ID
in a message header and on the server keeping track of those IDs for a congurable
rolling period of time in order to detect the publisher publishing the same message twice.
For the subscribers a double acknowledgment mechanism is used to avoid a message
being erroneously re-sent to a subscriber by the server after some kinds of failures.

JetStream consumers are 'views' on a stream, they are subscribed to (or pulled) by client
applications to receive copies of (or to consume if the stream is set as a working queue)
messages stored in the stream.

**Exactly once semantics**

**Consumers**


**Fast push consumers**

Client applications can choose to use fast un-acknowledged push (ordered) consumers
to receive messages as fast as possible (for the selected replay policy) on a specied
delivery subject or to an inbox. Those consumers are meant to be used to 'replay' rather
than 'consume' the messages in a stream.

**Horizontally scalable pull consumers with batching**

Client applications can also use and share pull consumers that are demand-driven,
support batching and must explicitly acknowledge message reception and processing
which means that they can be used to consume (i.e. use the stream as a distributed
queue) as well as process the messages in a stream.

Pull consumers can and are meant to be shared between applications (just like queue
groups) in order to provide easy and transparent horizontal scalability of the processing
or consumption of messages in a stream without having (for example) to worry about
having to dene partitions or worry about fault-tolerance.

Note: using pull consumers doesn't mean that you can't get updates (new messages
published into the stream) 'pushed' in real-time to your application, as you can pass a
(reasonable) timeout to the consumer's Fetch call and call it in a loop.

**Consumer acknowledgments**

While you can decide to use un-acknowledged consumers trading quality of service for
the fastest possible delivery of messages, most processing is not idem-potent and
requires higher qualities of service (such as the ability to automatically recover from
various failure scenarios that could result in some messages not being processed or
being processed more than once) and you will want to use acknowledged consumers.
JetStream supports more than one kind of acknowledgment:

```
Some consumers support acknowledging all the messages up to the sequence
number of the message being acknowledged, some consumers provide the highest
quality of service but require acknowledging the reception and processing of each
message explicitly as well as the maximum amount of time the server will wait for an
acknowledgment for a specic message before re-delivering it (to another process
attached to the consumer).
```

The JetStream persistence layer enables the Key Value store: the ability to store, retrieve
and delete value messages associated with a key into a bucket.

You can subscribe to changes in a Key Value on the bucket or individual key level with
watch and optionally retrieve a history of the values (and deletions) that have
happened on a particular key.

The Key Value store supports atomic create and update operations. This enables
pessimistic locks (by creating a key and holding on to it) and optimistic locks (using CAS

- compare and set).

The Object Store is similar to the Key Value Store. The key being replaced by a le name
and value being designed to store arbitrarily large objects (e.g. les, even if they are
very large) rather than 'values' that are message-sized (i.e. limited to 1Mb by default).
This is achieved by chunking messages.

```
You can also send back negative acknowledgements.
You can even send in progress acknowledgments (to indicate that you are still
processing the message in question and need more time before acking or nacking it).
```
```
Concepts
Walkthrough
API and details
```
```
Concepts
Walkthrough
```
##### Key Value Store

**Watch and History**

**Atomic updates and locking**

##### Object Store


Note that JetStream completely replaces the STAN legacy NATS streaming layer.

```
API and details
```
#### Legacy


### Streams

Streams are message stores, each stream denes how messages are stored and what
the limits (duration, size, interest) of the retention are. Streams consume normal NATS
subjects, any message published on those subjects will be captured in the dened
storage system. You can do a normal publish to the subject for unacknowledged delivery,
though it’s better to use the JetStream publish calls instead as the JetStream server will
reply with an acknowledgement that it was successfully stored.

The diagram above shows the concept of storing all ORDERS.* in the Stream even
though there are many types of order related messages. We’ll show how you can
selectively consume subsets of messages later. Relatively speaking the Stream is the
most resource consuming component so being able to combine related data in this
manner is important to consider.

Streams can consume many subjects. Here we have ORDERS.* but we could also
consume SHIPPING.state into the same Stream should that make sense.

```
Orders
```
#### Stream limits and message retention


Streams support various retention policies which dene when messages in the stream
can be automatically deleted, such as when stream limits are hit (like max count, size or
age of messages), or also more novel options that apply on top of the limits such as
interest-based retention or work-queue semantics (see Retention Policy).

Upon reaching message limits, the server will automatically discard messages either by
removing the oldest messages to make room for new ones (DiscardOld) or by refusing
to store new messages (DiscardNew). For more details, see Discard Policy.

Streams support deduplication using a Nats-Msg-Id header and a sliding window
within which to track duplicate messages. See the Message Deduplication section.

For examples on how to congure streams with your preferred NATS client, see NATS by
Example.

Below are the set of stream conguration options that can be dened. The Version
column indicates the version of the server the option was introduced. The Editable
column indicates the option can be edited after the stream created. See client-specic
examples here.

```
Field Description Version Editable
```
```
Name
```
```
Identies the stream and has to
be unique within JetStream
account. Names cannot contain
whitespace,. , * , > , path
separators (forward or backwards
slash), and non-printable
characters.
```
```
2.2.0 No
```
```
Storage The storage type for stream data. 2.2.0 No
```
```
Subjects
```
```
A list of subjects to bind.
Wildcards are supported. Cannot
be set for mirror streams.
```
```
2.2.0 Yes
```
#### Configuration


**Field Description Version Editable**

Replicas

```
How many replicas to keep for
each message in a clustered
JetStream, maximum 5.
```
```
2.2.0 Yes
```
MaxAge

```
Maximum age of any
message in the Stream,
expressed in nanoseconds.
```
```
2.2.0 Yes
```
MaxBytes

```
Maximum number of bytes
stored in the stream. Adheres to
Discard Policy, removing oldest
or refusing new messages if
the Stream exceeds this size.
```
```
2.2.0 Yes
```
MaxMsgs

```
Maximum number of messages
stored in the stream. Adheres
to Discard Policy, removing
oldest or refusing new
messages if the Stream exceeds
this number of messages.
```
```
2.2.0 Yes
```
MaxMsgSize

```
The largest message that will
be accepted by the Stream.
The size of a message is a
sum of payload and headers.
```
```
2.2.0 Yes
```
MaxConsumers

```
Maximum number of Consumers
that can be dened for a given
Stream, -1 for unlimited.
```
```
2.2.0 No
```
NoAck

```
Disables acknowledging
messages that are
received by the Stream.
```
```
2.2.0 Yes
```
Retention Declares the retentionpolicy for the stream.^ 2.2.0 No

Discard

```
The behavior of discarding
messages when any streams’
limits have been reached.
```
```
2.2.0 Yes
```

**Field Description Version Editable**

DuplicateWindow

```
The window within which to
track duplicate messages,
expressed in nanoseconds.
```
```
2.2.0 Yes
```
Placement

```
Used to declare where the
stream should be placed via tags
and/or an explicit cluster name.
```
```
2.2.0 Yes
```
Mirror If set, indicates this stris a mirror of another streameam.^ 2.2.0 No (if dened)

Sources

```
If dened, declares one or
more streams this stream
will source messages from.
```
```
2.2.0 Yes
```
MaxMsgsPerSubj
ect

```
Limits maximum number
of messages in the stream
to retain per subject.
```
```
2.3.0 Yes
```
Description A verbose descriptionof the stream.^ 2.3.3 Yes

Sealed

```
Sealed streams do not allow
messages to be deleted via limits
or API, sealed streams can not be
unsealed via conguration
update. Can only be set on
already created streams via the
Update API.
```
```
2.6.2 Yes (once)
```
DenyDelete

```
Restricts the ability to
delete messages from
a stream via the API.
```
```
2.6.2 No
```
DenyPurge

```
Restricts the ability to
purge messages from
a stream via the API.
```
```
2.6.2 No
```
AllowRollup Allows the use of the
Nats-Rollup header to
replace all contents of a

```
2.6.2 Yes
```

**Field Description Version Editable**
stream, or subject in a stream,
with a single new message.

RePublish

```
If set, messages stored
to the stream will be
immediately republished
to the congured subject.
```
```
2.8.3 Yes
```
AllowDirect

```
If true, and the stream has
more than one replica, each
replica will respond to direct
get requests for individual
messages, not only the leader.
```
```
2.9.0 Yes
```
MirrorDirect

```
If true, and the stream is
a mirror, the mirror will
participate in a serving direct
get requests for individual
messages from origin stream.
```
```
2.9.0 Yes
```
DiscardNewPerSu
bject

```
If true, applies discard new
semantics on a per subject
basis. Requires DiscardPolicy
to be DiscardNew and the
MaxMsgsPerSubject to be set.
```
```
2.9.0 Yes
```
Metadata

```
A set of application-dened
key-value pairs for associating
metadata on the stream.
```
```
2.10.0 Yes
```
Compression

```
If le-based and a compression
algorithm is specied, the
stream data will be compressed
on disk. Valid options are
nothing (empty string) or s2
for Snappy compression.
```
```
2.10.0 Yes
```
FirstSeq

```
If specied, a new stream
will be created with its initial
sequence set to this value.
```
```
2.10.0 No
```

The storage types include:

Note: a stream congured as a mirror cannot be congured with a set of subjects. A
mirror implicitly sources a subset of the origin stream (optionally with a lter), but does
not subscribe to additional subjects.

If no explicit subject is specied, the default subject will be the same name as the
stream. Multiple subjects can be specied and edited over time. Note, if messages are
stored by a stream on a subject that is subsequently removed from the stream cong,
consumers will still observe those messages if their subject lter overlaps.

The retention options include:

```
Field Description Version Editable
```
```
SubjectTransform
```
```
Applies a subject transform
(to matching messages)
before storing the message.
```
```
2.10.0 Yes
```
```
ConsumerLimits
```
```
Sets default limits for consumers
created for a stream. Those can
be overridden per consumer.
```
```
2.10.0 Yes
```
```
File (default) - Uses le-based storage for stream data.
Memory - Uses memory-based storage for stream data.
```
```
LimitsPolicy (default) - Retention based on the various limits that are set including:
MaxMsgs, MaxBytes, MaxAge, and MaxMsgsPerSubject. If any of these limits are
set, whichever limit is hit rst will cause the automatic deletion of the respective
message(s). See a full code example.
```
##### StorageType

##### Subjects

##### RetentionPolicy


```
If the InterestPolicy or WorkQueuePolicy is chosen for a stream, note that any limits, if
dened, will still be enforced. For example, given a work-queue stream, if MaxMsgs are set
and the default discard policy of old, messages will be automatically deleted even if the
consumer did not receive them.
```
```
WorkQueuePolicy streams will only delete messages enforced by limits or when a
message has been successfully Ack’d by its consumer. Messages that have attempted
redelivery and have reached MaxDelivery attempts for the consumer will remain in the
stream and must be manually deleted via the JetStream API.
```
The discard behavior applies only for streams that have at least one limit dened. The
options include:

```
InterestPolicy - Retention based on the consumer interest in the stream and
messages. The base case is that there are zero consumers dened for a stream. If
messages are published to the stream, they will be immediately deleted so there is no
interest. This implies that consumers need to be bound to the stream ahead of
messages being published to the stream. Once a given message is ack’ed by all
consumers, the message is deleted. See a full code example.
WorkQueuePolicy - Retention with the typical behavior of a FIFO queue. Each
message can be consumed only once. This is enforced by only allowing one consumer
to be created for a work-queue stream. Once a given message is ack’ed, it will be
deleted from the stream. See a full code example.
```
```
DiscardOld (default) - This policy will delete the oldest messages in order to
maintain the limit. For example, if MaxAge is set to one minute, the server will
automatically delete messages older than one minute with this policy.
DiscardNew - This policy will reject new messages from being appended to the
stream if it would exceed one of the limits. An extension to this policy is
DiscardNewPerSubject which will apply this policy on a per-subject basis within the
stream.
```
##### DiscardPolicy


Refers to the placement of the stream assets (data) within a NATS deployment, be it a
single cluster or a supercluster. A given stream, including all replicas (not mirrors), are
bound to a single cluster. So when creating or moving a stream, a cluster will be chosen
to host the assets.

Without declaring explicit placement for a stream, by default, the stream will be created
within the cluster that the client is connected to assuming it has sucient storage
available.

By declaring stream placement, where these assets are located can be controlled
explicitly. This is generally useful to co-locate with the most active clients (publishers or
consumers) or may be required for data sovereignty reasons.

Placement is supported in all client SDKs as well as the CLI. For example, adding a
stream via the the CLI to place a stream in a specic cluster looks like this:

For this to work, all servers in a given cluster must dene the name eld within the
cluster server conguration block.

If you have multiple clusters that form a supercluster, then each is required to have a
different name.

Another placement option are tags. Each server can have its own set of tags, dened in
conguration, typically describing properties of geography, hosting provider, sizing tiers,
etc. In addition, tags are often used in conjunction with the jetstream.unique_tag
cong option to ensure that replicas must be placed on servers having different values
for the tag.

```
nats stream add --cluster aws-us-east1-c1
```
```
cluster {
name: aws-us-east1-c1
# etc..
}
```
##### Placement


For example, a server A, B, and C in the above cluster might all the same conguration
except for the availability zone they are deployed to.

Now we can create a stream by using tags, for example indicating we want a stream in
us-east1.

If we had a second cluster in Google Cloud with the same region tag, the stream could be
placed in either the AWS or GCP cluster. However, the unique_tag constraint ensures
each replica will be placed in a different AZ in the cluster that was selected implicitly by
the placement tags.

Although less common, note that both the cluster and tags can be used for placement.
This would be used if a single cluster contains servers have different properties.

```
// Server A
server_tags: ["cloud:aws", "region:us-east1", "az:a"]
jetstream: {
unique_tag: "az"
}
// Server B
server_tags: ["cloud:aws", "region:us-east1", "az:b"]
jetstream: {
unique_tag: "az"
}
// Server C
server_tags: ["cloud:aws", "region:us-east1", "az:c"]
jetstream: {
unique_tag: "az"
}
```
```
nats stream add --tag region:us-east1
```
##### Sources and Mirrors


When a stream is congured with a source or mirror, it will automatically and
asynchronously replicate messages from the origin stream. There are several options
when declaring the conguration.

A source or mirror stream can have its own retention policy, replication, and storage type.
Changes to to the source or mirror,e.g. deleting messages or publishing, do not reect on
the origin stream.

```
Sources is a generalization of the Mirror and allows for sourcing data from one or more
streams concurrently. We suggest to use Sources in new congurations. If you require the
target stream to act as a read-only replica:
```
A stream dening Sources is a generalized replication mechanism and allows for
sourcing data from one or more streams concurrently as well as allowing direct
write/publish by clients. Essentially the source streams and client writes are aggregated
into a single interleaved stream. Subject transformation and ltering allow for powerful
data distribution architectures.

A mirror can source its messages from exactly one stream and a clients can not directly
write to the mirror. Although messages cannot be published to a mirror directly by clients,
messages can be deleted on-demand (beyond the retention policy), and consumers have
all capabilities available on regular streams.

**For details see:**

```
Congure the stream without listen subjects or
Temporarily disable the listen subjects through client authorizations.
```
```
Source and Mirror
```
**Stream sources**

**Mirrors**

##### AllowRollup


If enabled, the AllowRollup stream option allows for a published message having a
Nats-Rollup header indicating all prior messages should be purged. The scope of the
purge is dened by the header value, either all or sub.

The Nats-Rollup: all header will purge all prior messages in the stream. Whereas the
sub value will purge all prior messages for a given subject.

A common use case for rollup is for state snapshots, where the message being published
has accumulated all the necessary state from the prior messages, relative to the stream
or a particular subject.

If enabled, the RePublish stream option will result in the server re-publishing messages
received into a stream automatically and immediately after a successful write, to a
distinct destination subject.

For high scale needs where, currently, a dedicated consumer may add too much
overhead, clients can establish a core NATS subscription to the destination subject and
receive messages that were appended to the stream in real-time.

The elds for conguring republish include:

For each message that is republished, a set of headers are automatically added.

```
Source - An optional subject pattern which is a subset of the subjects bound to the
stream. It defaults to all messages in the stream, e.g. >.
Destination - The destination subject messages will be re-published to. The source
and destination must be a valid subject mapping.
HeadersOnly - If true, the message data will not be included in the re-published
message, only an additional header Nats-Msg-Size indicating the size of the
message in bytes.
```
##### RePublish

##### SubjectTransform


If congured, the SubjectTransform will perform a subject transform to matching
subjects of messages received by the stream and transform the subject, before storing it
in the stream. The transform conguration species a Source and Destination eld,
following the rules of subject transform.


### Source and Mirror Streams

When a stream is congured with a source or mirror, it will automatically and
asynchronously replicate messages from the origin stream.

source or mirror are designed to be robust and will recover from a loss of
connection. They are suitable for geographic distribution over high latency and unreliable
connections. E.g. even a leaf node starting and connecting intermittently every few days
will still receive or send messages over the source/mirror link.

There are several options available when declaring the conguration.

The stream using a source or mirror conguration can have its own retention policy,
replication, and storage type.

```
Name - Name of the origin stream to source messages from.
StartSeq - An optional start sequence of the origin stream to start mirroring from.
StartTime - An optional message start time to start mirroring from. Any messages
that are equal to or greater than the start time will be included.
FilterSubject - An optional lter subject which will include only messages that
match the subject, typically including a wildcard. Note, this cannot be used with
SubjectTransforms.
SubjectTransforms - An optional set of subject transforms to apply when sourcing
messages from the origin stream. Note, in this context, the Source will act as a lter
on the origin stream and the Destination can optionally be provided to apply a
transform. Since multiple subject transforms can be used, disjoint subjects can be
sourced from the origin stream while maintaining the order of the messages. Note,
this cannot be used with FilterSubject.
Domain - An optional JetStream domain of where the origin stream exists. This is
commonly used in a hub cluster and leafnode topology.
```

```
Sources is a generalization of the Mirror and allows for sourcing data from one or more
streams concurrently. If you require the target stream to act as a read-only replica:
```
A stream dening Sources is a generalized replication mechanism and allows for
sourcing data from **one or more streams** concurrently. A stream with sources can still act
as a regular stream allowing direct write/publish by local clients to the stream. Essentially
the source streams and local client writes are aggregated into a single interleaved
stream. Combined with subject transformation and ltering sourcing allows to design
sophisticated data distribution architectures.

```
Changes to to the stream using source or mirror,e.g. deleting messages or
publishing, do not reect back on the origin stream from which the data was
received.
Deletes in the origin stream are NOT replicated through a source or mirror
agreement.
```
```
Congure the stream without listen subjects or
Temporarily disable the listen subjects through client authorizations.
```
```
All congurations are made on the receiving side. The stream from which data is
sourced and mirrored does not need to be congured. No cleanup is required on the
origin side if the receiver disappears.
A stream can be the origin (source) for multiple streams. This is useful for geographic
distribution or for designing "fan out" topologies where data needs to be distributed
reliable to a large number (up to millions) of client connections.
Leaf nodes and leaf node domains are explicitly supported through the API prefix
```
#### General behavior

#### Source specific


```
Sourcing messages does not retain sequence numbers. But it retain the in stream sequence
of messages. Between streams sourced to the same target, the sequence of messages is
undened.
```
A mirror can source its messages from **exactly one stream** and a clients can not directly
write to the mirror. Although messages cannot be published to a mirror directly by clients,
messages can be deleted on-demand (beyond the retention policy), and consumers have
all capabilities available on regular streams.

```
Mirrored messages retains the sequence numbers and timestamps of the
origin stream.
Mirrors can be used for for (geographic) load distribution with the
MirrorDirect stream attribute. See: https://docs.nats.io/nats-
concepts/jetstream/streams#conguration
```
```
Source and mirror contracts are designed with one-way (geographic) data replication
in mind. Neither conguration provides a full synchronization between streams, which
would include deletes or replication of other stream attributes.
The content of the stream from which a source or mirror is drawn needs to be
reasonable stable. Quickly deleting messages after publishing them may result in
inconsistent replication due to the asynchronous nature of the replication process.
Sources and Mirror try to be be ecient in replicating messages and are lenient
towards the source/mirror origin being unreachable (event for extended periods of
time), e.g. when using leaf nodes, which are connected intermittently. For sake of
eciency the recovery interval in case of a disconnect is 10-20s.
Mirror and source agreements do not create a visible consumer in the origin stream.
```
#### Mirror specific

#### Expected behavior in edge conditions


Source and mirror work with origin stream with workqueue retention. The source/mirror
will act as a consumer removing messages from the origin stream.

The intended usage is for moving messages conveniently from one location to another
(e.g. from a leaf node). It does not trivially implement a distributed workqueue (where
messages are distributed to multiple streams sourcing from the same origin), although
with the help of subject ltering a distributed workqueue can be approximated.

```
If you try to create additional (conicting) consumers on the origin workqueue stream the
behavior becomes undened. A workqueue allows only one consumer per subject. If the
source/mirror connection is active local clients trying to create additional consumers will
fail. In reverse a source/mirror cannot be created when there is already a local consumer for
the same subjects.
```
```
Source and mirror for interest based streams is not supported. Jetstream does not forbid
this conguration but the behavior is undened and may change in the future.
```
##### WorkQueue retention

##### Interest base retention


### Example

Streams with source and mirror congurations are best managed through a client API. If
you intend to create such a conguration from command line with NATS CLI you should
use a JSON conguration.

**Minimal example**

```
nats stream add --config stream_with_sources.json
```
#### Example stream configuration with two sources


**With additional options**

```
{
"name": "SOURCE_TARGET",
"subjects": [
"foo1.ext.*",
"foo2.ext.*"
],
"discard": "old",
"duplicate_window": 120000000000,
"sources": [
{
"name": "SOURCE1_ORIGIN",
},
],
"deny_delete": false,
"sealed": false,
"max_msg_size": -1,
"allow_rollup_hdrs": false,
"max_bytes": -1,
"storage": "file",
"allow_direct": false,
"max_age": 0,
"max_consumers": -1,
"max_msgs_per_subject": -1,
"num_replicas": 1,
"name": "SOURCE_TARGET",
"deny_purge": false,
"compression": "none",
"max_msgs": -1,
"retention": "limits",
"mirror_direct": false
}
```

{
"name": "SOURCE_TARGET",
"subjects": [
"foo1.ext.*",
"foo2.ext.*"
],
"discard": "old",
"duplicate_window": 120000000000,
"sources": [
{
"name": "SOURCE1_ORIGIN",
"filter_subject": "foo1.bar",
"opt_start_seq": 42,
"external": {
"deliver": "",
"api": "$JS.domainA.API"
},

},
{
"name": "SOURCE2_ORIGIN",
"filter_subject": "foo2.bar"
}
],
"consumer_limits": {

},
"deny_delete": false,
"sealed": false,
"max_msg_size": -1,
"allow_rollup_hdrs": false,
"max_bytes": -1,
"storage": "file",
"allow_direct": false,
"max_age": 0,
"max_consumers": -1,
"max_msgs_per_subject": -1,
"num_replicas": 1,
"name": "SOURCE_TARGET",
"deny_purge": false,
"compression": "none",
"max_msgs": -1,
"retention": "limits",
"mirror_direct": false
}


**Minimal example**

**With additional options**

```
{
"name": "MIRROR_TARGET"
"discard": "old",
"mirror": {
"name": "MIRROR_ORIGIN"
},
"deny_delete": false,
"sealed": false,
"max_msg_size": -1,
"allow_rollup_hdrs": false,
"max_bytes": -1,
"storage": "file",
"allow_direct": false,
"max_age": 0,
"max_consumers": -1,
"max_msgs_per_subject": -1,
"num_replicas": 1,
"name": "MIRROR_TARGET",
"deny_purge": false,
"compression": "none",
"max_msgs": -1,
"retention": "limits",
"mirror_direct": false
}
```
#### Example stream configuration with mirror


{
"name": "MIRROR_TARGET"
"discard": "old",
"mirror": {
"opt_start_time": "2024-07-11T08:57:20.4441646Z",
"external": {
"deliver": "",
"api": "$JS.domainB.API"
},
"name": "MIRROR_ORIGIN"
},
"consumer_limits": {

},
"deny_delete": false,
"sealed": false,
"max_msg_size": -1,
"allow_rollup_hdrs": false,
"max_bytes": -1,
"storage": "file",
"allow_direct": false,
"max_age": 0,
"max_consumers": -1,
"max_msgs_per_subject": -1,
"num_replicas": 1,
"name": "MIRROR_TARGET",
"deny_purge": false,
"compression": "none",
"max_msgs": -1,
"retention": "limits",
"mirror_direct": false
}


### Consumers

A consumer is a stateful **view** of a stream. It acts as interface for clients to consume a
subset of messages stored in a stream and will keep track of which messages were
delivered and acknowledged by clients.

Unlike with core NATS which provides an **at most once** delivery guarantee of a message,
a consumer can provide an **at least once** delivery guarantee. This is achieved by the
combination of published messages being persisted to the stream as well as the
consumer tracking delivery and acknowledgement of each individual message as clients
receive and process them. JetStream consumers support multiple kinds of
acknowledgements and multiple acknowledgement policies. They will take care of
automatically re-deliver un-acked (or 'nacked') messages up to a user specied maximum
number of delivery attempts (there is an advisory being emitted when a message reaches
this limit).

Consumers can be **push** -based where messages will be delivered to a specied subject
or **pull** -based which allows clients to request batches of messages on demand. The
choice of what kind of consumer to use depends on the use-case.

If there is a need to process messages in an application controlled manner and easily
scale horizontally, you would use a 'pull consumer'. A simple client application that wants
a replay of messages from a stream sequentially you would use an 'ordered push
consumer'. An application that wants to benet from load balancing or acknowledge
messages individually will use a regular push consumer.

```
We recommend pull consumers for new projects. In particular when scalability, detailed ow
control or error handling are a concern.
```
#### Dispatch type - Pull / Push

##### Ordered Consumers


Ordered consumers are the convenient default type of push consumers designed for
applications that want to eciently consume a stream for data inspection or analysis.

In addition to the choice of being push or pull, a consumer can also be **ephemeral** or
**durable**. A consumer is considered durable when an explicit name is set on the Durable
eld when creating the consumer, otherwise it is considered ephemeral.

Durables and ephemeral have the same message delivery semantics but an ephemeral
consumer will not have persisted state or fault tolerance (server memory only) and will be
automatically cleaned up (deleted) after a period of inactivity, when no subscriptions are
bound to the consumer.

By default, durables will have replicated persisted state saved in the cluster and will
remain even when there are periods of inactivity (unless InactiveThreshold is set
explicitly). Durable consumers can recover from server and client failure.

```
Always ephemeral
Auto acknowledgment (no re-delivery)
Automatic ow control
Single-threaded dispatching
No load balancing
```
#### Persistence - Durable / Ephemeral


Below are the set of consumer conguration options that can be dened. The Version
column indicates the version of nats-server in which the option was introduced. The
Editable column indicates the option can be edited after the consumer is created.

NATS JS Consumers - The ONE feature that makes NATS more powerful than Kafka, Pulsar, RabbitMQ,
& redis

```
The ONE fThe ONE featureature that make that makes NATS mores NATS more powere powerful than Kafka, Pulsarful than Kafka, Pulsar......
```
```
Field Description Version Editable
```
```
Durable
```
```
If set, clients can have
subscriptions bind to the
consumer and resume until the
consumer is explicitly deleted. A
durable name cannot contain
whitespace,. , * , > , path
separators (forward or backward
slash), and non-printable
characters.
```
```
2.2.0 No
```
#### Configuration

##### General


**Field Description Version Editable**

FilterSubject

```
A subject that overlaps with the
subjects bound to the stream
to lter delivery to subscribers.
Note this cannot be used with the
FilterSubjects (multiple) eld.
```
```
2.2.0 Yes
```
AckPolicy

```
The requirement of client
acknowledgements,
either AckExplicit
, AckNone, or AckAll.
```
```
2.2.0 No
```
AckWait

```
The duration that the server will
wait for an ack for any individual
message once it has been
delivered to a consumer. If an
ack is not received in time, the
message will be redelivered.
```
```
2.2.0 Yes
```
DeliverPolicy

```
The point in the stream to receive
messages from, either
DeliverAll, DeliverLast,
DeliverNew,
DeliverByStartSequence,
DeliverByStartTime, or
DeliverLastPerSubject.
```
```
2.2.0 No
```
OptStartSeq

```
Used with the
DeliverByStartSequence
deliver policy.
```
```
2.2.0 No
```
OptStartTime

```
Used with the
DeliverByStartTime
deliver policy.
```
```
2.2.0 No
```
Description

```
A description of the consumer.
This can be particularly useful
for ephemeral consumers to
indicate their purpose since the
durable name cannot be provided.
```
```
2.3.3 Yes
```

**Field Description Version Editable**

InactiveThreshol
d

```
Duration that instructs the
server to cleanup consumers
that are inactive for that long.
Prior to 2.9, this only applied
to ephemeral consumers.
```
```
2.2.0 Yes
```
MaxAckPending

```
Denes the maximum number of
messages, without an
acknowledgement, that can be
outstanding. Once this limit is
reached message delivery will be
suspended. This limit applies
across all of the consumer's
bound subscriptions. A value of -1
means there can be any number of
pending acks (i.e. no ow control).
This does not apply when the
AckNone policy is used. The
default is 1000.
```
```
2.2.0 Yes
```
MaxDeliver

```
The maximum number of times a
specic message delivery will be
attempted. Applies to any
message that is re-sent due to ack
policy (i.e. due to a negative ack,
or no ack sent by the client). The
default is -1, redeliver until
acknowledged. Messages, which
have reached the maximum
delivery count will stay in the
stream.
```
```
2.2.0 Yes
```
Backoff A sequence of durations
controlling the redelivery of
messages on nak or
acknowledgment timeout.
Overrides ackWait. The
sequence length must be less or
equal to MaxDelivery. If backoff
is not set a nak will result in an
immediate re-delivery. E.g.
MaxDelivery=5

```
2.7.1 Yes
```

**Field Description Version Editable**
backoff=
[5s,30s,300s,3600s,84000s]
will re-deliver a message 5 times
over the course of one day.

ReplayPolicy

```
If the policy is ReplayOriginal,
the messages in the stream will be
pushed to the client at the same
rate that they were originally
received, simulating the original
timing of messages. If the policy
is ReplayInstant (the default),
the messages will be pushed to
the client as fast as possible while
adhering to the Ack Policy, Max
Ack Pending and the client's ability
to consume those messages.
```
```
2.2.0 No
```
Replicas

```
Sets the number of replicas for
the consumer's state. By default,
when the value is set to zero,
consumers inherit the number
of replicas from the stream.
```
```
2.8.3 Yes
```
MemoryStorage

```
If set, forces the consumer state
to be kept in memory rather than
inherit the storage type of the
stream (where le st orage is the
default). This reduces I/O from
acks, useful for ephemeral
consumers.
```
```
2.8.3 No
```
SampleFrequenc
y

```
Sets the percentage of
acknowledgements that should
be sampled for observability,
0-100 This value is a string
and for example allows both
30 and 30% as valid values.
```
```
2.2.0 Yes
```
Metadata

```
A set of application-dened
key-value pairs for associating
metadata on the consumer.
```
```
2.10.0 Yes
```

The policy choices include:

If an ack is required but is not received within the AckWait window, the message will be
redelivered.

```
The server may consider an ack arriving out of the window. If a rst process fails to ack
within the window it's entirely possible, for instance in queue situation, that the message has
been redelivered to another consumer. Since this will technically restart the window, the ack
from the rst consumer will be considered.
```
The policy choices include:

```
Field Description Version Editable
```
```
FilterSubjects
```
```
A set of subjects that overlap with
the subjects bound to the stream
to lter delivery to subscribers.
Note this cannot be used with the
FilterSubject (singular) eld.
```
```
2.10.0 Yes
```
```
AckExplicit - The default policy. It means that each individual message must be
acknowledged. It is recommended to use this mode, as it provides the most reliability
and functionality.
AckNone - You do not have to ack any messages, the server will assume ack on
delivery.
AckAll - If you receive a series of messages, you only have to ack the last one you
received. All the previous messages received are automatically acknowledged at the
same time.
```
```
DeliverAll - The default policy. The consumer will start receiving from the earliest
available message.
DeliverLast - When rst consuming messages, the consumer will start receiving
messages with the last message added to the stream, or the last message in the
```
**AckPolicy**

**DeliverPolicy**


The MaxAckPending capability provides one-to-many ow control and applies to both
push and pull consumers. For push consumers, MaxAckPending is the only form of ow
control. However, for pull consumers because the delivery of the messages to the client
application is client-driven (hence the 'pull') rather than server-initiated (hence the 'push')
there is an implicit one-to-one ow control with the subscribers (the maximum batch size
of the Fetch calls). Therefore, if you require high throughput you should remember to set
it to an appropriately high value (e.g. the default value of 1000), as it can otherwise place
a limit on the horizontal scalability of the processing of the stream in high throughput
situations. Conversely, if you have bursts of messages published to the stream and your
consuming application can be high latency to process the messages because it’s relying
on some external higher latency service (like a database for example), then either just
pull/fetch just a few at a time or set MaxAckPending to a value much lower than the
default and select 'AckWait' judiciously to avoid some messages getting re-delivered
because the processing is not fast enough to absorb the bursts (without causing re-
deliveries).

A lter subject provides a way to apply server-side ltering of messages by a consumer
prior to delivering them to clients.

```
stream that matches the consumer's lter subject if dened.
DeliverLastPerSubject - When rst consuming messages, start with the latest one
for each ltered subject currently in the stream.
DeliverNew - When rst consuming messages, the consumer will only start receiving
messages that were created after the consumer was created.
DeliverByStartSequence - When rst consuming messages, start at the rst
message having the sequence number or the next one available. The consumer is
required to specify OptStartSeq which denes the sequence number.
DeliverByStartTime - When rst consuming messages, start with messages on or
after this time. The consumer is required to specify OptStartTime which denes this
start time.
```
**MaxAckPending**

**FilterSubjects**


For example, given a stream factory-events with a bound subject of
factory-events.*.* and modeling a hierarchy of
factory-events.<factory-id>.<event-type>, a consumer factory-A could be
created with a lter of factory-events.A.* and only events for factory A would be
delivered to clients.

A consumer can be congured with a singular form FilterSubject or the plural form
FilterSubjects (as of 2.10). If the plural form, multiple disjoint lters can be applied,
such as [factory-events.A.*, factory-events.B.*] or even across all factories, but
choosing specic event types
[factory-events.*.item_produced, factory-events.*.item_packaged].

```
For use cases that require granular consumer permissions, a single lter will internally use
an extended consumer JetStream API,
$JS.API.CONSUMER.CREATE.{stream}.{consumer}.{filter}, which enables restricting
users to create consumers for only a specic subset of lter. For example an pub-allow
permission on $JS.API.CONSUMER.CREATE.factory-events.*.factory-events.A.*
which would allow a client to only create consumers specic to factory A for this stream.
Currently, when multiple lters are used, the more general form is used
$JS.API.CONSUMER.DURABLE.CREATE.{stream}.{consumer} which does not include the
{filter} token. If granular consumer permissions are required, a different strategy would
need to be used, such as pre-creating consumers for clients.
```
These options apply only to pull consumers. For an example on how congure a pull
consumer using your preferred client, see NATS by Example.

```
Field Description Version Editable
MaxWaiting The maximum numberof waiting pull requests.^ 2.2.0 No
```
```
MaxRequestExpir
es
```
```
The maximum duration a
single pull request will wait for
messages to be available to pull.
```
```
2.7.0 Yes
```
##### Pull-specific


These options apply only to push consumers. For an example on how to congure a push
consumer using your preferred client, see NATS by Example.

```
Field Description Version Editable
```
```
MaxRequestBatc
h
```
```
The maximum batch size a single
pull request can make. When set
with MaxRequestMaxBytes, the
batch size will be constrained
by whichever limit is hit rst.
```
```
2.7.0 Yes
```
```
MaxRequestMaxB
ytes
```
```
The maximum total bytes
that can be requested in a
given batch. When set with
MaxRequestBatch, the batch
size will be constrained by
whicheverlimitishitrst
```
```
2.8.3 Yes
```
```
Field Description Version Editable
```
```
DeliverSubject
```
```
The subject to deliver messages
to. Note, setting this eld implicitly
decides whether the consumer is
push or pull-based. With a deliver
subject, the server will push
messages to client subscribed to
this subject.
```
```
2.2.0 No
```
```
DeliverGroup
```
```
The queue group name which,
if specied, is then used to
distribute the messages
between the subscribers to the
consumer. This is analogous
to a queue group in core NATS.
```
```
2.2.0 Yes
```
```
FlowControl Enables per-subscription ow
control using a sliding-window
protocol. This protocol relies on
the server and client exchanging
messages to regulate when and
how many messages are pushed
```
```
2.2.0 Yes
```
##### Push-specific


**Field Description Version Editable**
to the client. This one-to-one ow
control mechanism works in
tandem with the one-to-many ow
control imposed by
MaxAckPending across all
subscriptions bound to a
consumer.

IdleHeartbeat

```
If the idle heartbeat period is set,
the server will regularly send a
status message to the client (i.e.
when the period has elapsed)
while there are no new messages
to send. This lets the client know
that the JetStream service is still
up and running, even when there is
no activity on the stream. The
message status header will have a
code of 100. Unlike FlowControl
, it will have no reply to address. It
may have a description such "Idle
Heartbeat". Note that this
heartbeat mechanism is all
handled transparently by
supported clients and does not
need to be handled by the
application.
```
```
2.2.0 Yes
```
RateLimit

```
Used to throttle the delivery
of messages to the
consumer, in bits per second.
```
```
2.2.0 Yes
```
HeadersOnly

```
Delivers only the headers of
messages in the stream and
not the bodies. Additionally
adds Nats-Msg-Size
header to indicate the size
of the removed payload.
```
```
2.6.2 Yes
```

### Example

Consider this architecture

While it is an incomplete architecture it does show a number of key points:

A new order arrives on ORDERS.received, gets sent to the NEW Consumer who, on
success, will create a new message on ORDERS.processed. The ORDERS.processed
message again enters the Stream where a DISPATCH Consumer receives it and once
processed it will create an ORDERS.completed message which will again enter the
Stream. These operations are all pull based meaning they are work queues and can
scale horizontally. All require acknowledged delivery ensuring no order is missed.

All messages are delivered to a MONITOR Consumer without any acknowledgement and
using Pub/Sub semantics - they are pushed to the monitor.

```
Orders
```
```
Many related subjects are stored in a Stream
Consumers can have different modes of operation and receive just subsets of the
messages
Multiple Acknowledgement modes are supported
```

As messages are acknowledged to the NEW and DISPATCH Consumers, a percentage of
them are Sampled and messages indicating redelivery counts, ack delays and more, are
delivered to the monitoring system.

Additional documentation introduces the nats utility and how you can use it to create,
monitor, and manage streams and consumers, but for completeness and reference this is
how you'd create the ORDERS scenario. We'll congure a 1 year retention for order related
messages:

```
nats stream add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-by
nats consumer add ORDERS NEW --filter ORDERS.received --ack explicit --pu
nats consumer add ORDERS DISPATCH --filter ORDERS.processed --ack explici
nats consumer add ORDERS MONITOR --filter '' --ack none --target monitor.
```
#### Example Configuration


### JetStream Walkthrough

The following is a small walkthrough on creating a stream and a consumer and
interacting with the stream using the nats cli.

If you are running a local nats-server stop it and restart it with JetStream enabled
using nats-server -js (if that's not already done)

You can then check that JetStream is enabled by using

```
nats account info
```
#### Prerequisite: enabling JetStream


If you see the below then JetStream is not enabled

```
Account Information
User:
Account: $G
Expires: never
Client ID: 5
Client IP: 127.0.0.1
RTT: 128μs
Headers Supported: true
Maximum Payload: 1.0 MiB
Connected URL: nats://127.0.0.1:4222
Connected Address: 127.0.0.1:4222
Connected Server ID: NAMR7YBNZA3U2MXG2JH3FNGKBDVBG2QTMWVO6OT7X
Connected Server Version: 2.11.0-dev
TLS Connection: no
JetStream Account Information:
Account Usage:
Storage: 0 B
Memory: 0 B
Streams: 0
Consumers: 0
Account Limits:
Max Message Payload: 1.0 MiB
Tier: Default:
Configuration Requirements:
Stream Requires Max Bytes Set: false
Consumer Maximum Ack Pending: Unlimited
Stream Resource Usage Limits:
Memory: 0 B of Unlimited
Memory Per Stream: Unlimited
Storage: 0 B of Unlimited
Storage Per Stream: Unlimited
Streams: 0 of Unlimited
Consumers: 0 of Unlimited
```

Let's start by creating a stream to capture and store the messages published on the
subject "foo".

Enter nats stream add <Stream name> (in the examples below we will name the
stream "my_stream"), then enter "foo" as the subject name and hit return to use the
defaults for all the other stream attributes:

```
JetStream Account Information:
JetStream is not supported in this account
```
```
nats stream add my_stream
```
#### 1. Creating a stream


? Subjects foo
? Storage file
? Replication 1
? Retention Policy Limits
? Discard Policy Old
? Stream Messages Limit -1
? Per Subject Messages Limit -1
? Total Stream Size -1
? Message TTL -1
? Max Message Size -1
? Duplicate tracking time window 2m0s
? Allow message Roll-ups No
? Allow message deletion Yes
? Allow purging subjects or the entire stream Yes
Stream my_stream was created

Information for Stream my_stream created 2024-06-07 12:29:36

Subjects: foo
Replicas: 1
Storage: File

Options:

Retention: Limits
Acknowledgments: true
Discard Policy: Old
Duplicate Window: 2m0s
Direct Get: true
Allows Msg Delete: true
Allows Purge: true
Allows Rollups: false

Limits:

Maximum Messages: unlimited
Maximum Per Subject: unlimited
Maximum Bytes: unlimited
Maximum Age: unlimited
Maximum Message Size: unlimited
Maximum Consumers: unlimited

State:

Messages: 0
Bytes: 0 B
First Sequence: 0
Last Sequence: 0


You can then check the information about the stream you just created:

```
Last Sequence: 0
Active Consumers: 0
```
```
nats stream info my_stream
```
```
Information for Stream my_stream created 2024-06-07 12:29:36
Subjects: foo
Replicas: 1
Storage: File
Options:
Retention: Limits
Acknowledgments: true
Discard Policy: Old
Duplicate Window: 2m0s
Direct Get: true
Allows Msg Delete: true
Allows Purge: true
Allows Rollups: false
Limits:
Maximum Messages: unlimited
Maximum Per Subject: unlimited
Maximum Bytes: unlimited
Maximum Age: unlimited
Maximum Message Size: unlimited
Maximum Consumers: unlimited
State:
Messages: 0
Bytes: 0 B
First Sequence: 0
Last Sequence: 0
Active Consumers: 0
```
#### 2. Publish some messages into the stream


Let's now start a publisher

As messages are being published on the subject "foo" they are also captured and stored
in the stream, you can check that by using nats stream info my_stream and even look
at the messages themselves using nats stream view my_stream

Now at this point if you create a 'Core NATS' (i.e. non-streaming) subscriber to listen for
messages on the subject 'foo', you will only receive the messages being published after
the subscriber was started, this is normal and expected for the basic 'Core NATS'
messaging. In order to receive a 'replay' of all the messages contained in the stream
(including those that were published in the past) we will now create a 'consumer'

We can administratively create a consumer using the 'nats consumer add ' command, in
this example we will name the consumer "pull_consumer", and we will leave the delivery
subject to 'nothing' (i.e. just hit return at the prompt) because we are creating a 'pull
consumer' and select all for the start policy, you can then just use the defaults and hit
return for all the other prompts. The stream the consumer is created on should be the
stream 'my_stream' we just created above.

```
nats pub foo --count=1000 --sleep 1 s "publication #{{Count}} @ {{TimeStam
```
```
nats consumer add
```
#### 3. Creating a consumer


You can check on the status of any consumer at any time using nats consumer info or
view the messages in the stream using nats stream view my_stream or even remove
individual messages from the stream using nats stream rmm

Now that the consumer has been created and since there are messages in the stream we
can now start subscribing to the consumer:

```
? Consumer name pull_consumer
? Delivery target (empty for Pull Consumers)
? Start policy (all, new, last, subject, 1h, msg sequence) all
? Acknowledgment policy explicit
? Replay policy instant
? Filter Stream by subjects (blank for all)
? Maximum Allowed Deliveries -1
? Maximum Acknowledgments Pending 0
? Deliver headers only without bodies No
? Add a Retry Backoff Policy No
? Select a Stream my_stream
Information for Consumer my_stream > pull_consumer created 2024-06-07T12:
Configuration:
Name: pull_consumer
Pull Mode: true
Deliver Policy: All
Ack Policy: Explicit
Ack Wait: 30.00s
Replay Policy: Instant
Max Ack Pending: 1,000
Max Waiting Pulls: 512
State:
Last Delivered Message: Consumer sequence: 0 Stream sequence: 0
Acknowledgment Floor: Consumer sequence: 0 Stream sequence: 0
Outstanding Acks: 0 out of maximum 1,000
Redelivered Messages: 0
Unprocessed Messages: 74
Waiting Pulls: 0 of maximum 512
```
#### 3. Subscribing from the consumer


This will print out all the messages in the stream starting with the rst message (which
was published in the past) and continuing with new messages as they are published until
the count is reached.

Note that in this example we are creating a pull consumer with a 'durable' name, this
means that the consumer can be shared between as many consuming processes as you
want. For example instead of running a single nats consumer next with a count of
1000 messages you could have started two instances of nats consumer each with a
message count of 500 and you would see the consumption of the messages from the
consumer distributed between those instances of nats

Once you have iterated over all the messages in the stream with the consumer, you can
get them again by simply creating a new consumer or by deleting that consumer (
nats consumer rm) and re-creating it (nats consumer add).

You can clean up a stream (and release the resources associated with it (e.g. the
messages stored in the stream)) using nats stream purge

You can also delete a stream (which will also automatically delete all of the consumers
that may be dened on that stream) using nats stream rm

```
nats consumer next my_stream pull_consumer --count 1000
```
**Replaying the messages again**

#### 4. Cleaning up


### Key/Value Store

JetStream, the persistence layer of NATS, not only allows for the higher qualities of
service and features associated with 'streaming', but it also enables some functionalities
not found in messaging systems.

One such feature is the Key/Value store functionality, which allows client applications to
create buckets and use them as immediately (as opposed to eventually) consistent,
persistent associative arrays (or maps).

1. Create a bucket, which corresponds to a stream in the underlying storage. Dene
    KV/Stream limits as appropriate
2. Use the operation below.

You can use KV buckets to perform the typical operations you would expect from an
immediately consistent key/value store:

```
Walkthrough
Details
```
```
put: associate a value with a key
get: retrieve the value associated with a key
delete: clear any value associated with a key
purge: clear all the values associated with all keys
keys: get a copy of all of the keys (with a value or operation associated with it)
```
#### Managing a Key Value store

#### Map style operations


You can set limits for your buckets, such as:

Finally, you can even do things that typically can not be done with a Key/Value Store:

```
create: associate the value with a key only if there is currently no value associated with
that key (i.e. compare to null and set)
update: compare and set (aka compare and swap) the value for a key
```
```
the maximum size of the bucket
the maximum size for any single value
a TTL: how long the store will keep values for
```
```
watch: watch for changes happening for a key, which is similar to subscribing (in the
publish/subscribe sense) to the key: the watcher receives updates due to put or delete
operations on the key pushed to it in real-time as they happen
watch all: watch for all the changes happening on all the keys in the bucket
history: retrieve a history of the values (and delete operations) associated with each
key over time (by default the history of buckets is set to 1, meaning that only the latest
value/operation is stored)
```
#### Atomic operations used for locking and

#### concurrency control

#### Limiting size, TTL etc.

#### Treating the Key Value store as a message

#### stream

#### Notes


The key conforms to the same naming restriction as a NATS subject, i.e. it can be a dot-
separated list of tokens (which means that you can then use wildcards to match
hierarchies of keys when watching a bucket), and can only contain valid characters. The
value can be any byte array


### Key/Value Store Walkthrough

The Key/Value Store is a JetStream feature, so we need to verify it is enabled by

which may return

In this case, you should enable JetStream.

If you are running a local nats-server stop it and restart it with JetStream enabled
using nats-server -js (if that's not already done)

You can then check that JetStream is enabled by using

```
nats account info
```
```
JetStream Account Information:
JetStream is not supported in this account
```
```
nats account info
```
#### Prerequisite: enabling JetStream


A 'KV bucket' is like a stream; you need to create it before using it, as in
nats kv add <KV Bucket Name>:

```
Connection Information:
Client ID: 6
Client IP: 127.0.0.1
RTT: 64.996μs
Headers Supported: true
Maximum Payload: 1.0 MiB
Connected URL: nats://127.0.0.1:4222
Connected Address: 127.0.0.1:4222
Connected Server ID: ND2XVDA4Q363JOIFKJTPZW3ZKZCANH7NJI4EJMFSSPTRXDB
JetStream Account Information:
Memory: 0 B of Unlimited
Storage: 0 B of Unlimited
Streams: 0 of Unlimited
Consumers: 0 of Unlimited
```
```
nats kv add my-kv
```
```
my_kv Key-Value Store Status
Bucket Name: my-kv
History Kept: 1
Values Stored: 0
Compressed: false
Backing Store Kind: JetStream
Bucket Size: 0 B
Maximum Bucket Size: unlimited
Maximum Value Size: unlimited
Maximum Age: unlimited
JetStream Stream: KV_my-kv
Storage: File
```
#### Creating a KV bucket


Now that we have a bucket, we can assign, or 'put', a value to a specic key:

which should return the key's value Value1

We can fetch, or 'get', the value for a key "Key1":

You can always delete a key and its value by using

It is harmless to delete a non-existent key (check this!!).

K/V Stores can also be used in concurrent design patterns, such as semaphores, by using
atomic 'create' and 'update' operations.

```
nats kv put my-kv Key1 Value1
```
```
nats kv get my-kv Key1
```
```
my-kv > Key1 created @ 12 Oct 21 20:08 UTC
Value1
```
```
nats kv del my-kv Key1
```
#### Storing a value

#### Getting a value

#### Deleting a value

#### Atomic operations


E.g. a client wanting exclusive use of a le can lock it by creating a key, whose value is the
le name, with create and deleting this key after completing use of that le. A client
can increase the reslience against failure by using a timeout for the bucket containing
this key. The client can use update with a revision number to keep the bucket alive.

Updates can also be used for more ne-grained concurrency control, sometimes known
as optimistic locking, where multiple clients can try a task, but only one can
successfully complete it.

Create a lock/semaphore with the create operation.

Only one create can succeed. First come, rst serve. All concurrent attempts will result
in an error until the key is deleted

We can also atomically update, sometimes known as a CAS (compare and swap)
operation, a key with an additional parameter revision

A second attempt with the same revision 13, will fail

```
nats kv create my-sem Semaphore1 Value1
```
```
nats kv create my-sem Semaphore1 Value1
nats: error: nats: wrong last sequence: 1 : key exists
```
```
nats kv update my-sem Semaphore1 Value2 13
```
```
nats kv update my-sem Semaphore1 Value2 13
nats: error: nats: wrong last sequence: 14
```
##### Create (aka exclusive locking)

##### Update with CAS (aka optimistic locking)


An unusual functionality of a K/V Store is being able to 'watch' a bucket, or a specic key
in that bucket, and receive real-time updates to changes in the store.

For the example above, run nats kv watch my-kv. This will start a watcher on the
bucket we have just created earlier. By default, the KV bucket has a history size of one,
and so it only remembers the last change. In our case, the watcher should see a delete of
the value associated with the key "Key1":

If we now concurrently change the value of 'my-kv' by

The watcher will see that change:

When you are nished using a bucket, you can delete the bucket, and its resources, by
using the rm operator:

```
nats kv watch my-kv
```
```
[2021-10-12 13:15:03] DEL my-kv > Key1
```
```
nats kv put my-kv Key1 Value2
```
```
[ 2021 -10-12 13 :25:14] PUT my-kv > Key1: Value2
```
```
nats kv rm my-kv
```
#### Watching a K/V Store

#### Cleaning up


### Object Store

JetStream, the persistence layer of NATS, not only allows for the higher qualities of
service and features associated with 'streaming', but it also enables some functionalities
not found in messaging systems.

One such feature is the Object store functionality, which allows client applications to
create buckets (corresponding to streams) that can store a set of les. Files are stored
and transmitted in chunks, allowing les of arbitrary size to be transferred safely over the
NATS infrastructure.

**Note:** Object store is not a distributed storage system. All les in a bucket will need to t
on the target le system.

The Object Store implements a chunking mechanism, allowing you to for example store
and retrieve les (i.e. the object) of any size by associating them with a path or le name
as the key.

```
Walkthrough
Details
```
```
add a bucket to hold the les.
put Add a le to the bucket
get Retrieve the le and store it to a designated location
del Delete a le
```
```
watch Subscribe to changes in the bucket. Will receive notications on successful
put and del operations.
```
#### Basic Capabilities

#### Advanced Capabilities


### Object Store Walkthrough

If you are running a local nats-server stop it and restart it with JetStream enabled
using nats-server -js (if that's not already done)

You can then check that JetStream is enabled by using

Which should output something like:

If you see the below instead then JetStream is not enabled

```
nats account info
```
```
Connection Information:
Client ID: 6
Client IP: 127.0.0.1
RTT: 64.996μs
Headers Supported: true
Maximum Payload: 1.0 MiB
Connected URL: nats://127.0.0.1:4222
Connected Address: 127.0.0.1:4222
Connected Server ID: ND2XVDA4Q363JOIFKJTPZW3ZKZCANH7NJI4EJMFSSPTRXDB
JetStream Account Information:
Memory: 0 B of Unlimited
Storage: 0 B of Unlimited
Streams: 0 of Unlimited
Consumers: 0 of Unlimited
```
```
JetStream Account Information:
JetStream is not supported in this account
```
#### Creating an Object Store bucket


Just like you need to create streams before you can use them you need to rst create an
Object Store bucket

which outputs

By default the full le path is used as a key. Provide the key explicitly (e.g. a relative path )
with --name

```
nats object add myobjbucket
```
```
myobjbucket Object Store Status
Bucket Name: myobjbucket
Replicas: 1
TTL: unlimitd
Sealed: false
Size: 0 B
Backing Store Kind: JetStream
JetStream Stream: OBJ_myobjbucket
```
```
nats object put myobjbucket ~/Movies/NATS-logo.mov
```
```
1.5 GiB / 1.5 GiB [======================================================
Object information for myobjbucket > /Users/jnmoyne/Movies/NATS-logo.mov
Size: 1.5 GiB
Modification Time: 14 Apr 22 00:34 +0000
Chunks: 12,656
Digest: sha-256 8ee0679dd1462de393d81a3032d71f43d2bc89c0c8a5
```
```
nats object put --name /Movies/NATS-logo.mov myobjbucket ~/Movies/NATS-lo
```
#### Putting a file in the bucket

#### Putting a file in the bucket by providing a name


```
1.5 GiB / 1.5 GiB [======================================================
Object information for myobjbucket > /Movies/NATS-logo.mov
Size: 1.5 GiB
Modification Time: 14 Apr 22 00:34 +0000
Chunks: 12,656
Digest: sha-256 8ee0679dd1462de393d81a3032d71f43d2bc89c0c8a5
```
```
nats object ls myobjbucket
```
###### ╭─────────────────────────────────────────────

###### │ Bucket Contents

###### ├─────────────────────────────────────┬───────

###### │ Name │ Size │ Time

###### ├─────────────────────────────────────┼───────

###### │ /Users/jnmoyne/Movies/NATS-logo.mov │ 1.5 GiB │ 2022-04-13T17:34:55-07

###### │ /Movies/NATS-logo.mov │ 1.5 GiB │ 2022-04-13T17:35:41-07

###### ╰─────────────────────────────────────┴───────

```
nats object get myobjbucket ~/Movies/NATS-logo.mov
```
```
1.5 GiB / 1.5 GiB [======================================================
Wrote: 1.5 GiB to /Users/jnmoyne/NATS-logo.mov in 5.68s average 279 MiB/s
```
#### Listing the objects in a bucket

#### Getting an object from the bucket

#### Getting an object from the bucket with a

#### specific output path


By default, the le will be stored relative to the local path under its name (not the full
path). To specify an output path use --output

```
nats object get myobjbucket --output /temp/Movies/NATS-logo.mov /Movies/NA
```
```
1.5 GiB / 1.5 GiB [======================================================
Wrote: 1.5 GiB to /temp/Movies/NATS-logo.mov in 5.68s average 279 MiB/s
```
```
nats object rm myobjbucket ~/Movies/NATS-logo.mov
```
```
? Delete 1.5 GiB byte file myobjbucket > /Users/jnmoyne/Movies/NATS-logo.
Removed myobjbucket > /Users/jnmoyne/Movies/NATS-logo.mov
myobjbucket Object Store Status
Bucket Name: myobjbucket
Replicas: 1
TTL: unlimitd
Sealed: false
Size: 16 MiB
Backing Store Kind: JetStream
JetStream Stream: OBJ_myobjbucket
```
```
nats object info myobjbucket
```
#### Removing an object from the bucket

#### Getting information about the bucket


You can seal a bucket, meaning that no further changes are allowed on that bucket

```
myobjbucket Object Store Status
Bucket Name: myobjbucket
Replicas: 1
TTL: unlimitd
Sealed: false
Size: 1.6 GiB
Backing Store Kind: JetStream
JetStream Stream: OBJ_myobjbucket
```
```
nats object watch myobjbucket
```
```
[2022-04-13 17:51:28] PUT myobjbucket > /Users/jnmoyne/Movies/NATS-logo.m
[2022-04-13 17:53:27] DEL myobjbucket > /Users/jnmoyne/Movies/NATS-logo.m
```
```
nats object seal myobjbucket
```
```
? Really seal Bucket myobjbucket, sealed buckets can not be unsealed or m
myobjbucket has been sealed
myobjbucket Object Store Status
Bucket Name: myobjbucket
Replicas: 1
TTL: unlimitd
Sealed: true
Size: 1.6 GiB
Backing Store Kind: JetStream
JetStream Stream: OBJ_myobjbucket
```
#### Watching for changes to a bucket

##### Sealing a bucket


Using nats object rm myobjbucket will delete the bucket and all the les stored in it.

#### Deleting a bucket


### Headers

Message headers are used in a variety of JetStream contexts, such de-duplication, auto-
purging of messages, metadata from republished messages, and more.

Headers that can be set by a client when a message being published.

```
Name Description Example Version
```
```
Nats-Msg-Id
```
```
Client-dened unique
identier for a message
that will be used by the
server apply de-duplication
within the congured
Duplicate Window.
```
```
9f01ccf0-
8c34-4789-
8688-
231a2538a98b
```
```
2.2.0
```
```
Nats-Expected-
Stream
```
```
Used to assert the published
message is received by
some expected stream.
```
```
my-stream 2.2.0
```
```
Nats-Expected-
Last-Msg-Id
```
```
Used to apply optimistic
concurrency control at the
stream-level. The value is the
last expected Nats-Msg-Id
and the server will reject a
publish if the current ID does
not match.
```
```
9f01ccf0-
8c34-4789-
8688-
231a2538a98b
```
```
2.2.0
```
```
Nats-Expected-
Last-Sequence
```
```
Used to apply optimistic
concurrency control at the
stream-level. The value is the
last expected sequence and
the server will reject a publish
if the current sequence does
not match.
```
```
328 2.2.0
```
#### Publish


Headers set messages that are republished.

```
Name Description Example Version
```
```
Nats-Expected-
Last-Subject-
Sequence
```
```
Used to apply optimistic
concurrency control at the
subject-level. The value is the
last expected sequence and
the server will reject a publish
if the current sequence does
not match for the message's
subject.
```
```
38 2.3.1
```
```
Nats-Rollup
```
```
Used to apply a purge of
all prior messages in the
t tth bj t l l
```
```
all for stream,
sub for subject 2.6.2
```
```
Name Description Example Version
```
```
Nats-Stream
```
```
Name of the stream
the message was
republished from.
```
```
Nats-Stream:
my-stream 2.8.3
```
```
Nats-Subject The original subjectof the message.^ events.mouse_clicked 2.8.3
```
```
Nats-Sequence The original sequenceof the message.^193 2.8.3
```
```
Nats-Last-
Sequence
```
```
The last sequence of
the message having the
same subject, otherwise
zero if this is the rst
message for the subject.
```
```
190 2.8.3
```
```
Nats-Time-
Stamp
```
```
The original timestamp
of the message.
```
```
2023-08-
23T19:53:05.7
62416Z
```
```
2.10.0
```
#### RePublish


Headers that are implicitly added to messages sourced from other streams.

Headers added to messages when the consumer is congured to be "headers only"
omitting the body.

Headers used for internal ow-control messages for a mirror.

```
Name Description Example Version
```
```
Nats-Stream-
Source
```
```
Species the origin stream name,
the subject and the sequence
number plus the subject lter
and destination transform of
the message being sourced.
```
```
my-stream 2.2.0
```
```
Name Description Example Version
Nats-Msg-Size Indicates the messagesize in bytes.^1024 2.6.2
```
```
Name Description Example Version
Nats-Last-Consumer 2.2.1
Nats-Last-Stream 2.2.1
Nats-Consumer-Stalled 2.4.0
Nats-Response-Type 2.6.4
```
#### Sources

#### Headers-only

#### Mirror


### Subject Mapping and Partitioning

Subject mapping and transforms is a powerful feature of the NATS server.
Transformations (we will use mapping and transform interchangeably) apply to various
situations when messages are generated and ingested, acting as translations and in
some scenarios as lters.

```
Mapping and transforms is an advanced topic. Before proceeding, please ensure you
understand NATS concepts like clusters, accounts and streams.
```
**Transforms can be applied (for details see below):**

**Transforms may be used for:**

```
On the server level (default $G account). Applying to all matching messages entering
the cluster or leaf node. Non-matching subjects will be unchanged.
On the individual account level following the same rules as above.
On subjects, which are imported into an account.
In JetStream context:
On messages imported by streams
On messages republished by JetStream
On messages copied to a stream via a source or mirror. For this purpose, the
transform acts as a lter.
```
```
Translating between namespaces. E.g. between accounts, but also when clusters and
leaf nodes implement different semantics for the same subject.
Suppressing subjects. E.g. Temporarily for testing.
For backward compatibility after changing the subject naming hierarchy.
Merging subjects together.
Disambiguation and isolation on super-clusters or leaf nodes, by using different
transforms in different clusters and leaf nodes.
Testing. E.g. merging a test subject temporarily into a production subject or rerouting a
production subject away from a production consumer.
```

**Priority and sequence of operations**

On a central cluster

On a leaf cluster

```
Partitioning subjects and JetStream streams
Filtering messages copied (sourced/mirrored) into a JetStream stream
Chaos testing and sampling. Mappings can be weighted. Allowing for a certain
percentage of messages to be rerouted, simulating loss, failure etc.
...
```
```
Transforms are applied as soon as a message enters the scope in which the transform
was dened (cluster, account, leaf node, stream) and independent of how they arrived
(publish by client, passing through gateway, stream import, stream source/mirror).
And before any routing or subscription interest is applied. The message will appear as
if published from the transformed subject under all circumstances.
Transforms are not applied recursively in the same scope. This is necessary to
prevent trivial loops. In the example below only the rst matching rule is applied.
```
```
mappings: {
transform.order target.order
target.order transform.order
}
```
```
Transforms are applied in sequence as they pass through different scopes. For
example:
```
1. A subject is transformed while being published
2. Routed to a leaf node and transformed when received on the leaf node
3. Imported into a stream and stored under a transformed name
4. Republished from the stream to Core NATS under a nal target subject

```
server_name: "hub"
cluster: { name: "hub" }
mappings: {
orders.* orders.central.{{wildcard(1)}}
}
```

A stream cong on the leaf cluster

**Security**

When using **cong le-based account management** (not using JWT security), you can
dene the core NATS account level subject transforms in server conguration les, and
simply need to reload the conguration whenever you change a transform for the change
to take effect.

When using **operator JWT security** (distributed security) with the built-in resolver you
dene the transforms and the import/exports in the account JWT, so after modifying
them, they will take effect as soon as you push the updated account JWT to the servers.

**Testing and debugging**

```
You can easily test individual subject transform rules using the nats CLI tool command
nats server mapping. See examples below.
```
```
From NATS server 2.11 (and NATS versions published thereafter) the handling of subjects,
including mappings can be observed with nats trace
```
```
server_name: "store1"
cluster: { name: "store1" }
mappings: {
orders.central.* orders.local.{{wildcard(1)}}
}
```
```
{
"name": "orders",
"subjects": [ "orders.local.*"],
"subject_transform":{"src":"orders.local.*","dest":"orders.{{wildcard(1
"retention": "limits",
...
"republish": [
{
"src": "orders.*",
"dest": "orders.trace.{{wildcard(1)}}""
}
],
```

```
In the example below a message is rst disambiguated from orders.device1.order1 ->
orders.hub.device1.order1. Then imported into a stream and stored under its original
name.
```
The example of foo:bar is straightforward. All messages the server receives on subject
foo are remapped and can be received by clients subscribed to bar.

When no subject is provided the command will operate in interactive mode:

Example server cong:

```
nats trace orders.device1.order1
Tracing message route to subject orders.device1.order1
Client "NATS CLI Version development" cid:16 cluster:"hub" server:
Mapping subject:"orders.hub.device1.order1"
--J JetStream action:"stored" stream:"orders" subject:"orders.devi
--X No active interest
Legend: Client: --C Router: --> Gateway: ==> Leafnode: ~~> JetStre
Egress Count:
JetStream: 1
```
```
nats server mapping foo bar foo
> bar
```
```
nats server mapping foo bar
> Enter subjects to test, empty subject terminates.
>
>? Subject foo
> bar
>? Subject test
> Error: no matching transforms available
```
#### Simple mapping


With accounts

Wildcard tokens may be referenced by position number in the destination mapping using
(only for versions 2.8.0 and above of nats-server) {{wildcard(position)}}. E.g.
{{wildcard(1)}} references the rst wildcard token, {{wildcard(2)}} references the
second wildcard token, etc..

Example: with this transform "bar.*.*" : "baz.{{wildcard(2)}}.{{wildcard(1)}}",
messages that were originally published to bar.a.b are remapped in the server to
baz.b.a. Messages arriving at the server on bar.one.two would be mapped to
baz.two.one, and so forth. Try it for yourself using nats server mapping.

```
server_name: "hub"
cluster: { name: "hub" }
mappings: {
orders.flush orders.central.flush
orders.* orders.central.{{wildcard(1)}}
}
```
```
server_name: "hub"
cluster: { name: "hub" }
accounts {
accountA: {
mappings: {
orders.flush orders.central.flush
orders.* orders.central.{{wildcard(1)}}
}
}
}
```
```
nats server mapping "bar.*.*" "baz.{{wildcard(2)}}.{{wildcard(1)}}" bar.
> baz.b.a
```
#### Subject token reordering


```
An older style deprecated mapping syntax using $1.$2 en lieu of
{{wildcard(1)}}.{{wildcard(2)}} may be seen in other examples.
```
You can drop tokens from the subject by not using all the wildcard tokens in the
destination transform, with the exception of mappings dened as part of import/export
between accounts in which case all the wildcard tokens must be used in the transform
destination.

```
Import/export mapping must be mapped bidirectionally unambiguous.
```
There are two ways you can split tokens:

You can split a token on each occurrence of a separator string using the
split(separator) transform function.

Examples:

```
nats server mapping "orders.*.*" "foo.{{wildcard(2)}}" orders.local.order
> orders.order1
```
```
Split on '-': nats server mapping "*" "{{split(1,-)}}" foo-bar returns
foo.bar.
Split on '--': nats server mapping "*" "{{split(1,--)}}" foo--bar returns
foo.bar.
```
#### Dropping subject tokens

#### Splitting tokens

##### Splitting on a separator


You can split a token in two at a specic location from the start or the end of the token
using the SplitFromLeft(wildcard index, offset) and
SplitFromRight(wildcard index, offset) transform functions (note that the upper
camel case on all subject transform function names is optional you can also use all
lowercase function names if you prefer).

Examples:

You can slice tokens into multiple parts at a specic interval from the start or the end of
the token by using the SliceFromLeft(wildcard index, number of characters) and
SliceFromRight(wildcard index, number of characters) mapping functions.

Examples:

```
Split the token at 4 from the left:
nats server mapping "*" "{{splitfromleft(1,4)}}" 1234567 returns
1234.567.
Split the token at 4 from the right:
nats server mapping "*" "{{splitfromright(1,4)}}" 1234567 returns
123.4567.
```
```
Split every 2 characters from the left:
nats server mapping "*" "{{slicefromleft(1,2)}}" 1234567 returns
12.34.56.7.
Split every 2 characters from the right:
nats server mapping "*" "{{slicefromright(1,2)}}" 1234567 returns
1.23.45.67.
```
##### Splitting at an offset

#### Slicing tokens

#### Deterministic subject token partitioning


Deterministic token partitioning allows you to use subject-based addressing to
deterministically divide (partition) a ow of messages where one or more of the subject
tokens is mapped into a partition key. Deterministically means, the same tokens are
always mapped into the same key. The mapping will appear random and may not be
fair for a small number of subjects.

For example: new customer orders are published on neworders.<customer id>, you
can partition those messages over 3 partition numbers (buckets), using the
partition(number of partitions, wildcard token positions...) function which
returns a partition number (between 0 and number of partitions-1) by using the following
mapping "neworders.*" : "neworders.{{wildcard(1)}}.{{partition(3,1)}}".

```
Note that multiple token positions can be specied to form a kind of composite partition
key. For example, a subject with the form foo.*.* can have a partition transform of
foo.{{wildcard(1)}}.{{wildcard(2)}}.{{partition(5,1,2)}} which will result in ve
partitions in the form foo.*.*.<n>, but using the hash of the two wildcard tokens when
computing the partition number.
```
This particular transform means that any message published on
neworders.<customer id> will be mapped to
neworders.<customer id>.<a partition number 0, 1, or 2>. i.e.:

```
nats server mapping "neworders.*" "neworders.{{wildcard(1)}}.{{partition(
> neworders.customerid1.0
```
```
nats server mapping "foo.*.*" "foo.{{wildcard(1)}}.{{wildcard(2)}}
> foo.us.customerid.0
```
```
Published on Mapped to
neworders.customerid1 neworders.customerid1.0
neworders.customerid2 neworders.customerid2.2
neworders.customerid3 neworders.customerid3.1
neworders.customerid4 neworders.customerid4.2
```

The transform is deterministic because (as long as the number of partitions is 3)
'customerid1' will always map to the same partition number. The mapping is hash-based,
its distribution is random but tends towards 'perfectly balanced' distribution (i.e. the more
keys you map the more the number of keys for each partition will tend to converge to the
same number).

You can partition on more than one subject wildcard token at a time, e.g.:
{{partition(10,1,2)}} distributes the union of token wildcards 1 and 2 over 10
partitions.

What this deterministic partition transform enables is the distribution of the messages
that are subscribed to using a single subscriber (on neworders.*) into three separate
subscribers (respectively on neworders.*.0, neworders.*.1 and neworders.*.2)
that can operate in parallel.

The core NATS queue-groups and JetStream durable consumer mechanisms to distribute
messages amongst a number of subscribers are partition-less and non-deterministic,
meaning that there is no guarantee that two sequential messages published on the same

```
Published on Mapped to
neworders.customerid5 neworders.customerid5.1
```
```
neworders.customerid6 neworders.customerid6.0
```
```
Published on Mapped to
foo.1.a foo.1.a.1
foo.1.b foo.1.b.0
foo.2.b foo.2.b.9
foo.2.a foo.2.a.2
```
```
nats server mapping "foo.*.*" "foo.{{wildcard(1)}}.{{wildcard(2)}}.{{part
```
##### When is deterministic partitioning uselful


subject are going to be distributed to the same subscriber. While in most use cases a
completely dynamic, demand-driven distribution is what you need, it does come at the
cost of guaranteed ordering because if two subsequent messages can be sent to two
different subscribers which would then both process those messages at the same time
at different speeds (or the message has to be re-transmitted, or the network is slow, etc.)
and that could result in potential 'out of order' message delivery.

This means that if the application requires strictly ordered message processing, you need
to limit distribution of messages to 'one at a time' (per consumer/queue-group, i.e. using
the 'max acks pending' setting), which in turn hurts scalability because it means no
matter how many workers you have subscribed, only one is doing any processing work at
a time.

Being able to evenly split (i.e. partition) subjects in a deterministic manner (meaning that
all the messages on a particular subject are always mapped to the same partition) allows
you to distribute and scale the processing of messages in a subject stream while still
maintaining strict ordering per subject. For example, inserting a partition number as a
token in the message subject as part of the stream denition and then using subject
lters t o create a consumer per partition (or set of partitions).

Another scenario for deterministic partitioning is in the extreme message publication rate
scenarios where you are reaching the limits of the throughput of incoming messages into
a stream capturing messages using a wildcard subject. This limit can be ultimately
reached at very high message rate due to the fact that a single nats-server process is
acting as the RAFT leader (coordinator) for any given stream and can therefore become a
limiting factor. In that case, distributing (i.e. partitioning) that stream into a number of
smaller streams (each one with its own RAFT leader and therefore all these RAFT leaders
are spread over all of the JetStream-enabled nats-servers in the cluster rather than a
single one) in order to scale.

Yet another use case where deterministic partitioning can help is if you want to leverage
local data caching of data (context or potentially heavy historical data for example) that
the subscribing process need to access as part of the processing of the messages.

#### Weighted mappings


Trac can be split by percentage from one subject transform to multiple subject
transforms.

Here's an example for canary deployments, starting with version 1 of your service.

Applications would make requests of a service at myservice.requests. The
responders doing the work of the server would subscribe to myservice.requests.v1.
Your conguration would look like this:

All requests to myservice.requests will go to version 1 of your service.

When version 2 comes along, you'll want to test it with a canary deployment. Version 2
would subscribe to myservice.requests.v2. Launch instances of your service.

Update the conguration le to redirect some portion of the requests made to
myservice.requests to version 2 of your service.

For example the conguration below means 98% of the requests will be sent to version 1
and 2% to version 2.

Once you've determined Version 2 is stable you can switch 100% of the trac o ver to it
and you can then shut down the version 1 instance of your service.

```
myservice.requests: [
{ destination: myservice.requests.v1, weight: 100% }
]
```
```
myservice.requests: [
{ destination: myservice.requests.v1, weight: 98% },
{ destination: myservice.requests.v2, weight: 2% }
]
```
##### For A/B testing or canary releases

##### For traffic shaping in testing


Trac shaping is also useful in testing. You might have a service that runs in QA that
simulates failure scenarios which could receive 20% of the trac t o test the service
requestor.

```
myservice.requests.*: [{ destination: myservice.requests.{{wildcard(1)}},
weight: 80% }, { destination: myservice.requests.fail.{{wildcard(1)}},
weight: 20% }
```
Alternatively, introduce loss into your system for chaos testing by mapping a percentage
of trac t o the same subject. In this drastic example, 50% of the trac published to
foo.loss.a would be articially dropped by the server.

```
foo.loss.>: [ { destination: foo.loss.>, weight: 50% } ]
```
You can both split and introduce loss for testing. Here, 90% of requests would go to your
service, 8% would go to a service simulating failure conditions, and the unaccounted for
2% would simulate message loss.

myservice.requests: [{ destination: myservice.requests.v3, weight: 90% },
{ destination: myservice.requests.v3.fail, weight: 8% }]
the remaining 2% is "lost"

If you are running a super-cluster you can dene transforms that apply only to messages
being published from a specic cluster.

For example if you have 3 clusters named east central and west and you want to
map messages published on foo in the east cluster to foo.east, those published in
the central cluster to foo.central and so on for west you can do so by using the
cluster keyword in the mapping source and destination.

##### For artificial loss

#### Cluster scoped mappings


This means that the application can be portable in terms of deployment and does not
need to be congured with the name of the cluster it happens to be connected in order to
compose the subject: it just publishes to foo and the server will map it to the
appropriate subject based on the cluster it's running in.

You can dene subject mapping transforms as part of the stream conguration.

Transforms can be applied in multiple places in the stream conguration:

Note that when used in Mirror, Sources or Republish, the subject transforms are lters
with optional transformation, while when used in the Stream conguration it only
transforms the subjects of the matching messages and does not act as a lter.

```
mappings = {
"foo":[
{destination:"foo.west", weight: 100%, cluster: "west"},
{destination:"foo.central", weight: 100%, cluster: "centra
{destination:"foo.east", weight: 100%, cluster: "east"}
]
}
```
```
You can apply a subject mapping transformation as part of a stream mirror
You can apply a subject mapping transformation as part of a stream source
You can apply an overall stream ingress subject mapping transformation that applies
to all matching messages regardless of how they are ingested into the stream
You can also apply a subject mapping transformation as part of the re-publishing of
messages
```
#### Subject mapping and transforms in streams


```
For sources and republish transforms the src expression will act as a lter. Non-
matching subjects will be ignored.
For the stream level subject_transform non-matching subjects will stay untouched.
```
{
"name": "orders",
"subjects": [ "orders.local.*"],
"subject_transform":{"src":"orders.local.*","dest":"orders.{{wildcard(1
"retention": "limits",
...
"sources": [
{
"name": "other_orders",
"subject_transforms": [
{
"src": "orders.online.*",
"dest": "orders.{{wildcard(1)}}"
}
]
}
],
"republish": [
{
"src": "orders.*",
"dest": "orders.trace.{{wildcard(1)}}""
}
]

}



### NATS Service Infrastructure

NATS is a client/server system in the fact that you have 'NATS client applications'
(applications using one of the NATS client libraries) that connect to 'NATS servers' that
provide the NATS service. The NATS servers work together to provide a NATS service
infrastructure to their client applications.

NATS is extremely exible and scalable and allows the service infrastructure to be as
small as a single process running locally on your local machine and as large as an
'Internet of NATS' of Leaf Nodes, and Leaf Node clusters all interconnected in a secure
way over a global shared NATS super-cluster.

Regardless of the size and complexity of the NATS service infrastructure being used, the
only conguration needed by the client applications being the location (NATS URLs) of
one or more NATS servers and depending on the required security, their credentials.

Note that if your application is written in Golang then you even have the option of
embedding the NATS server functionality into the application itself (however you need to
then congure your application instances with nats-server conguration information).

You do not actually need to run your NATS service infrastructure, instead you can instead
make use of a public NATS infrastructure offered by a NATS Service Provider such as
Synadia Cloud, think of Synadia Cloud as being an 'Internet of NATS' (literally an
"InterNATS") and of Synadia as being an "InterNATS Service Provider".

You will typically start by running a single instance of nats-server on your local
development machine, and have your applications connect to it while you do your
application development and local testing.

Next you will probably want to start testing and running those applications and servers in
a VPC, or a region or in some on-prem location, so you will deploy either single NATS
server or clusters of NATS servers in your VPCs/regions/on-prem/etc... locations and in

#### The Evolution of your NATS service

#### infrastructure


each location have the applications connect their local nats-server or nats-server cluster.
You can then connect those local nats-servers or local nats-server clusters together by
making them leaf nodes connecting to a 'backbone' cluster or super-cluster, or by
connecting them directly together via gateway connections.

If you have very many client applications (i.e. applications deployed on end-user devices
all over the Internet, or for example many IoT devices) or many servers in many locations
you will then scale your NATS service infrastructure by deploying clusters of NATS
servers in multiple locations and multiple cloud providers and VPCs, and connecting
those clusters together into a global super-cluster and then devise a scheme to
intelligently direct your client applications to the right 'closest' NATS server cluster.

You can deploy and run your own NATS service infrastructure of nats-server instances,
composed of servers, clusters of servers, super-cluster and leaf node NATS servers.

If using Kubernetes we recommend you use the Helm charts.

#### Running your own NATS service infrastructure

##### Virtualization and containerization considerations


### NATS Adaptive

### Deployment Architectures

From a single process to a global super-cluster with leaf node servers, you can always
adapt your NATS service deployment to your needs. From servers and VPCs in many
clouds, to partially connected small edge devices and everything in between, you can
always easily extend and scale your NATS service as your needs grow.

The simplest version of a NATS service infrastructure is a single nats-server process.
The nats-server binary is highly optimized, very lightweight and extremely ecient in
its resources' usage.

Client applications establish a connection to the URL of that nats-server process (e.g.
"nats://localhost").

If you need a fault-tolerant NATS service or if you need to scale your service capacity, you
can cluster a set of nats-server processes together in a cluster.

Client applications establish and maintain a connection to (one of) the nats server URL(s)
composing the cluster (e.g. "nats://server1","nats://server2",...).

###### NATS Client nats-server NATS Client

#### A single server

#### A cluster of servers


You can go further than a single cluster and have disaster recovery and get global
deployments (e.g. on multiple locations or regions, multiple VPCs or multiple Cloud
providers) by deploying multiple clusters and connecting them together via gateway
connections (which are interest pruned).

Client applications establish a connection to (one of) the nats server URL(s) of one of the
clusters (e.g.
"nats://us-west-1.company.com","nats://us-west-2.company.com",...).

You can easily 'extend' the NATS service provided by a cluster or super-cluster by
deploying 'locally' one or more **leaf node** nats servers that proxy and route trac between
their client applications and the NATS service infrastructure. The context of 'locality' in
this case is not just physical: it could mean a location, an edge device or a single
development machine, but it could also service a VPC, a group of server processes for a
specic application or different accounts, or even a business unit. Leaf node NATS

```
NATS Client nats-server
```
```
nats-server nats-server NATS Client
```
```
NATS Client nats-server
```
```
NATS Client
nats-server
```
```
nats-server
```
```
nats-server
```
```
nats-server
```
```
nats-server
```
#### A super-cluster

#### With Leaf Nodes


servers can be congured to connect to their cluster over a WebSocket connection
(rather than TLS or plain TCP).

Leaf nodes appear to the cluster as a single account connection. Leaf nodes can provide
continuous NATS service for their clients, even while being temporarily disconnected
from the cluster(s). You can even enable JetStream on the leaf nodes in order to create
local streams that are mirrored (mirroring is store and forward and therefore can recover
from connectivity outages) to global streams in the upstream cluster(s).

Client applications are congured with the URLs of their 'local' leaf node server(s) and
establish a connection to (one of) the leaf node server(s) (e.g.
"nats://leaf-node-1","nats://leaf-node-2",...).

NATS Service Geo-anity in Queues

```
NATS Client
```
```
nats-server
```
```
nats-server NATS Client
```
```
nats-server
```
```
nats-server
```
```
nats-server
(leaf node)
```
```
(leaf node)
```
```
NATS Client
NATS Client
NATS Client
```
```
NATS Client
```
```
NATS Client NATS Client
```
```
NATS Client
```
#### See Also


```
Geo-anity in Queues
```
Publish-Subscribe PPublish-Subscribe Pattern using the new Nattern using the new NATS CLIATS CLI


### Security

NATS has a lot of security features:

You can use accounts for multi-tenancy: each account has its own independent 'subject
namespace' and you control the import/export of both streams of messages and
services between accounts, and any number of users that client applications can be
authenticated as. The subjects or subject wildcards that a user is allowed to publish
and/or subscribe to can be controlled either through server conguration or as part of
signed JWTs.

JWT authentication/authorization administration is decentralized because each account
private key holder can manage their users and their authorizations on their own, without
the need for any conguration change on the NATS servers by minting their own JWTs
and distributing them to the users. There is no need for the NATS server to ever store any
user private keys as they only need to validate the signature chain of trust contained in
the user JWT presented by the client application to validate that they have the proper
public key for that user.

The JetStream persistence layer of NATS also provides encryption at rest.

```
Connections can be encrypted with TLS
Client connections can be authenticated in many ways:
Token Authentication
Username/Password credentials
TLS Certicate
NKEY with Challenge
Decentralized JWT Authentication/Authorization
You can also integrate NATS with your existing authentication/authorization system
or create your own custom authentication using the Auth callout
Authenticated clients are identied as users and have a set of authorizations
```

### Connectivity

NATS supports several kinds of connectivity directly to the NATS servers.

There is also a number of adapters available to bridge trac t o and from other
messaging systems

```
Plain NATS connections
TLS encrypted NATS connections
WebSocket NATS connections
MQTT client connections
```
```
Kafka Bridge
JMS which can also be used to bridge MQ and RabbitMQ, since they both offer a JMS
interface
```

## Using NATS


### NATS Tools

The most common form of connecting to the NATS messaging system will be through
application build with any of the 40+ client libraries available for NATS.

The client application will connect to an instance of the NATS server, be it a single server,
a cluster of servers or even a global super-cluster such as Synadia Cloud, sending and
receiving messages via a range of subscribers contracts. If the application is written in
GoLang the NATS server can even be embedded into a Go application.

Client APIs will also allow access to almost all server conguration tasks when using an
account with sucient permissions.

Besides using the client API to manage NATS servers, the NATS ecosystem also has
many tools to interact with other applications and services over NATS and streams,
support server conguration, enhance monitoring or tune performance such as:

```
General interaction and management
nats - The nats Command Line Tool is the easiest way to interact with, test and
manage NATS and JetStream from a terminal or from scripts. It's list of features are
ever growing, so please download the latest version.
Security
nk - Generate NKeys for use with JSon Web Tokens (JWT) used with nsc
nsc - Congure Operators, Accounts, Users and permission oine to later push
them to a production server. This is the preferred tools to create security
conguration unless you are using Synadia Control Plane
nats account server - ( legacy, replaced by the built-in NATS resolver ) a custom
security server. NAS can still be used as a reference implementation for you tailor-
made security integration.
```
#### Using NATS from client application

#### Command Line Tooling


Monitoring
nats top - Monitor NATS Servers
prometheus-nats-exporter - Export NATS server metrics to Prometheus and a
Grafana dashboard.
Benchmarking
see nats bench subcommand of the nats tool


### nats

A command line utility to interact with and manage NATS.

This utility replaces various past tools that were named in the form nats-sub and
nats-pub, adds several new capabilities and supports full JetStream management.

Check out the repo for all the details: github.com/nats-io/natscli.

For macOS:

For Arch Linux:

Download the correct .deb le for your computer from here.

If you have an Intel CPU, then it'll probably be this one (for version X.Y.Z):
nats-X.Y.Z-amd64.deb Then run this command to install the le.

Or with the yay package manager

For Windows:

```
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
```
```
sudo dpkg -i nats-X.Y.Z-amd64.deb
```
```
yay natscli
```
```
Invoke-WebRequest -Uri "https://get-nats.io/install.ps1" -OutFile "instal
```
#### Installing nats


Depending on your PowerShell settings, you may be prevented from running the install
script because it is currently not digitally signed. To work around this, you can change the
settings for the current shell session with this command:

You can read about execution policies here.

Binaries are also available as GitHub Releases.

```
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```
```
NATS Command Line Interface README
nats help
nats help [<command>...] or nats [<command>...] --help
Remember to look at the cheat sheets!
nats cheat
nats cheat --sections
nats cheat <section>>
```
```
nats context
nats account
nats pub
nats sub
nats request
nats reply
nats bench
```
#### Using nats

##### Getting help

##### Interacting with NATS


The CLI has a number of conguration settings that can be passed either as command
line arguments or set in environment variables.

```
nats events
nats rtt
nats server
nats latency
nats governor
```
```
nats stream
nats consumer
nats backup
nats restore
```
```
nats kv
```
```
nats errors
nats schema
```
```
nats --help
```
##### Monitoring NATS

##### Managing and interacting with streams

##### Managing and interacting with the K/V Store

##### Get reference information

#### Configuration Contexts


Output extract

The server URL can be set using the --server CLI ag, or the NATS_URL environment
variable, or using NATS Contexts.

The password can be set using the --password CLI ag, or the NATS_PASSWORD
environment variable, or using NATS Contexts. For example: if you want to create a script
that prompts the user for the system user password (so that for example it doesn't
appear in ps or history or maybe you don't want it stored in the prole) and then
execute one or more nats commands you do something like:

A context is a named conguration that stores all of these settings. You can designate a
default context and switch between contexts.

A context can be created with nats context create my_context_name and then
modied withnats context edit my_context_name:

```
...
-s, --server=URL NATS server urls ($NATS_URL)
--user=USER Username or Token ($NATS_USER)
--password=PASSWORD Password ($NATS_PASSWORD)
--creds=FILE User credentials ($NATS_CREDS)
--nkey=FILE User NKEY ($NATS_NKEY)
--tlscert=FILE TLS public certificate ($NATS_CERT)
--tlskey=FILE TLS private key ($NATS_KEY)
--tlsca=FILE TLS certificate authority chain ($NATS_CA
--timeout=DURATION Time to wait on responses from NATS
($NATS_TIMEOUT)
--context=NAME Configuration context ($NATS_CONTEXT)
...
```
```
#!/bin/bash
echo "-n" "system user password: "
read -s NATS_PASSWORD
export NATS_PASSWORD
nats server report jetstream --user system
```
##### NATS Contexts


This context is stored in the le ~/.config/nats/context/my_context_name.json.

A context can also be created by specifying settings with nats context save

List your contexts

We passed --select to the local one meaning it will be the default when nothing is
set.

Select a context

```
{
"description": "",
"url": "nats://127.0.0.1:4222",
"token": "",
"user": "",
"password": "",
"creds": "",
"nkey": "",
"cert": "",
"key": "",
"ca": "",
"nsc": "",
"jetstream_domain": "",
"jetstream_api_prefix": "",
"jetstream_event_prefix": "",
"inbox_prefix": "",
"user_jwt": ""
}
```
```
nats context save example --server nats://nats.example.net:4222 --descrip
nats context save local --server nats://localhost:4222 --description 'Loc
```
```
nats context ls
```
```
Known contexts:
example Example.Net Server
local* Local Host
```

Check the round trip time to the server (using the currently selected context)

You can also specify a context directly

All nats commands are context aware and the nats context command has various
commands to view, edit and remove contexts.

Server URLs and Credential paths can be resolved via the nsc command by specifying
an URL, for example to nd user new within the orders account of the acme operator
you can use this:

The server list and credentials path will now be resolved via nsc, if these are specically
set in the context, the specic context conguration will take precedence.

```
nats context select
```
```
nats rtt
```
```
nats://localhost:4222:
nats://127.0.0.1:4222: 245.115μs
nats://[::1]:4222: 390.239μs
```
```
nats rtt --context example
```
```
nats://nats.example.net:4222:
nats://192.0.2.10:4222: 41.560815ms
nats://192.0.2.11:4222: 41.486609ms
nats://192.0.2.12:4222: 41.178009ms
```
```
nats context save example --description 'Example.Net Server' --nsc nsc://
```
#### Generating bcrypted passwords


The server supports hashing of passwords and authentication tokens using bcrypt. To
take advantage of this, simply replace the plaintext password in the conguration with its
bcrypt hash, and the server will automatically utilize bcrypt as needed. See also:
Bcrypted Passwords.

The nats utility has a command for creating bcrypt hashes. This can be used for a
password or a token in the conguration.

To use the password on the server, add the hash into the server conguration le's
authorization section.

Note the client will still have to provide the plain text version of the password, the server
however will only store the hash to verify that the password is correct when supplied.

Publish-subscribe pattern using the NATS CLI

```
nats server passwd
```
```
? Enter password [? for help] **********************
? Reenter password [? for help] **********************
$2a$11$3kIDaCxw.Glsl1.u5nKa6eUnNDLV5HV9tIuUp7EHhMt6Nm9myW1aS
```
```
authorization {
user: derek
password: $2a$11$3kIDaCxw.Glsl1.u5nKa6eUnNDLV5HV9tIuUp7EHhMt6Nm9myW1a
}
```
#### See Also


```
Publish-subscribe Pattern using NATS CLI
```
Publish-Subscribe PPublish-Subscribe Pattern using the new Nattern using the new NATS CLIATS CLI


### nats bench

NATS is fast and lightweight and places a priority on performance. the nats CLI tool
can, amongst many other things, be used for running benchmarks and measuring
performance of your target NATS service infrastructure. In this tutorial you learn how to
benchmark and tune NATS on your systems and environment.

Verify that the NATS server starts successfully, as well as the HTTP monitor:

```
Install the NATS CLI Tool
Install the NATS server
```
```
nats-server -m 8222 -js
```
```
[89075] 2021/10/05 23:26:35.342816 [INF] Starting nats-server
[89075] 2021/10/05 23:26:35.342971 [INF] Version: 2.6.1
[89075] 2021/10/05 23:26:35.342974 [INF] Git: [not set]
[89075] 2021/10/05 23:26:35.342976 [INF] Name: NDUYLGUUNSD53IUR77SQ
[89075] 2021/10/05 23:26:35.342979 [INF] Node: ESalpH2B
[89075] 2021/10/05 23:26:35.342981 [INF] ID: NDUYLGUUNSD53IUR77SQ
[89075] 2021/10/05 23:26:35.343583 [INF] Starting JetStream
[89075] 2021/10/05 23:26:35.343946 [INF] _ ___ _____ ___ _____ ___ __
[89075] 2021/10/05 23:26:35.343955 [INF] _ | | __|_ _/ __|_ _| _ \ _
[89075] 2021/10/05 23:26:35.343957 [INF] | || | _| | | __ \ | | | / _|
[89075] 2021/10/05 23:26:35.343959 [INF] __/|___| |_| |___/ |_| |_|____/
[89075] 2021/10/05 23:26:35.343960 [INF]
[89075] 2021/10/05 23:26:35.343962 [INF] https://docs.nats.io/je
[89075] 2021/10/05 23:26:35.343964 [INF]
[89075] 2021/10/05 23:26:35.343967 [INF] ---------------- JETSTREAM -----
[89075] 2021/10/05 23:26:35.343970 [INF] Max Memory: 48.00 GB
[89075] 2021/10/05 23:26:35.343973 [INF] Max Storage: 581.03 GB
[89075] 2021/10/05 23:26:35.343974 [INF] Store Directory: "/var/folders
[89075] 2021/10/05 23:26:35.343979 [INF] --------------------------------
```
#### Prerequisites

#### Start the NATS server with monitoring enabled


Let's run a test to see how fast a single publisher can publish one million 16 byte
messages to the NATS server.

The output tells you the number of messages and the number of payload bytes that the
client was able to publish per second:

Now increase the number of messages published:

When using both publishers and subscribers, nats bench reports aggregate, as well as
individual publish and subscribe throughput performance.

Let's look at throughput for a single publisher with a single subscriber:

```
nats bench foo --pub 1 --size 16
```
```
23:33:51 Starting pub/sub benchmark [msgs=100,000, msgsize=16 B, pubs=1,
23:33:51 Starting publisher, publishing 100,000 messages
Finished 0s [=======================================================
Pub stats: 5,173,828 msgs/sec ~ 78.95 MB/sec
```
```
nats bench foo --pub 1 --size 16 --msgs 10000000
```
```
23:34:29 Starting pub/sub benchmark [msgs=10,000,000, msgsize=16 B, pubs=
23:34:29 Starting publisher, publishing 10,000,000 messages
Finished 2s [=======================================================
Pub stats: 4,919,947 msgs/sec ~ 75.07 MB/sec
```
```
nats bench foo --pub 1 --sub 1 --size 16
```
#### Run a publisher throughput test

#### Run a publish/subscribe throughput test


Note that the output shows the aggregate throughput as well as the individual publisher
and subscriber performance:

When specifying multiple publishers, or multiple subscribers, nats bench will also
report statistics for each publisher and subscriber individually, along with min/max/avg
and standard deviation.

Let's increase both the number of messages, and the number of subscribers.:

```
23:36:00 Starting pub/sub benchmark [msgs=100,000, msgsize=16 B, pubs=1,
23:36:00 Starting subscriber, expecting 100,000 messages
23:36:00 Starting publisher, publishing 100,000 messages
Finished 0s [=======================================================
Finished 0s [=======================================================
NATS Pub/Sub stats: 5,894,441 msgs/sec ~ 89.94 MB/sec
Pub stats: 3,517,660 msgs/sec ~ 53.68 MB/sec
Sub stats: 2,957,796 msgs/sec ~ 45.13 MB/sec
```
```
nats bench foo --pub 1 --sub 5 --size 16 --msgs 1000000
```
#### Run a 1:N throughput test


When more than 1 publisher is specied, nats bench evenly distributes the total
number of messages (-msgs) across the number of publishers (-pub).

Now let's increase the number of publishers and examine the output:

```
23:38:08 Starting pub/sub benchmark [msgs=1,000,000, msgsize=16 B, pubs=1
23:38:08 Starting subscriber, expecting 1,000,000 messages
23:38:08 Starting subscriber, expecting 1,000,000 messages
23:38:08 Starting subscriber, expecting 1,000,000 messages
23:38:08 Starting subscriber, expecting 1,000,000 messages
23:38:08 Starting subscriber, expecting 1,000,000 messages
23:38:08 Starting publisher, publishing 1,000,000 messages
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
NATS Pub/Sub stats: 7,123,965 msgs/sec ~ 108.70 MB/sec
Pub stats: 1,188,419 msgs/sec ~ 18.13 MB/sec
Sub stats: 5,937,525 msgs/sec ~ 90.60 MB/sec
[1] 1,187,633 msgs/sec ~ 18.12 MB/sec (1000000 msgs)
[2] 1,187,597 msgs/sec ~ 18.12 MB/sec (1000000 msgs)
[3] 1,187,526 msgs/sec ~ 18.12 MB/sec (1000000 msgs)
[4] 1,187,528 msgs/sec ~ 18.12 MB/sec (1000000 msgs)
[5] 1,187,505 msgs/sec ~ 18.12 MB/sec (1000000 msgs)
min 1,187,505 | avg 1,187,557 | max 1,187,633 | stddev 48 msgs
```
```
nats bench foo --pub 5 --sub 5 --size 16 --msgs 1000000
```
#### Run a N:M throughput test


In one shell start a nats bench in 'reply mode' and let it run

```
23:39:28 Starting pub/sub benchmark [msgs=1,000,000, msgsize=16 B, pubs=5
23:39:28 Starting subscriber, expecting 1,000,000 messages
23:39:28 Starting subscriber, expecting 1,000,000 messages
23:39:28 Starting subscriber, expecting 1,000,000 messages
23:39:28 Starting subscriber, expecting 1,000,000 messages
23:39:28 Starting subscriber, expecting 1,000,000 messages
23:39:28 Starting publisher, publishing 200,000 messages
23:39:28 Starting publisher, publishing 200,000 messages
23:39:28 Starting publisher, publishing 200,000 messages
23:39:28 Starting publisher, publishing 200,000 messages
23:39:28 Starting publisher, publishing 200,000 messages
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
Finished 0s [=======================================================
NATS Pub/Sub stats: 7,019,849 msgs/sec ~ 107.11 MB/sec
Pub stats: 1,172,667 msgs/sec ~ 17.89 MB/sec
[1] 236,240 msgs/sec ~ 3.60 MB/sec (200000 msgs)
[2] 236,168 msgs/sec ~ 3.60 MB/sec (200000 msgs)
[3] 235,541 msgs/sec ~ 3.59 MB/sec (200000 msgs)
[4] 234,911 msgs/sec ~ 3.58 MB/sec (200000 msgs)
[5] 235,545 msgs/sec ~ 3.59 MB/sec (200000 msgs)
min 234,911 | avg 235,681 | max 236,240 | stddev 485 msgs
Sub stats: 5,851,064 msgs/sec ~ 89.28 MB/sec
[1] 1,171,181 msgs/sec ~ 17.87 MB/sec (1000000 msgs)
[2] 1,171,169 msgs/sec ~ 17.87 MB/sec (1000000 msgs)
[3] 1,170,867 msgs/sec ~ 17.87 MB/sec (1000000 msgs)
[4] 1,170,641 msgs/sec ~ 17.86 MB/sec (1000000 msgs)
[5] 1,170,250 msgs/sec ~ 17.86 MB/sec (1000000 msgs)
min 1,170,250 | avg 1,170,821 | max 1,171,181 | stddev 349 msgs
```
```
nats bench foo --sub 1 --reply
```
#### Run a request-reply latency test


And in another shell send some requests

In this case the average latency of request-reply between the two nats bench
processes over NATS was 1/8,601th of a second (116.2655505 microseconds).

You can now hit control-c to kill that nats bench --reply process

Note: by default nats bench subscribers in 'reply mode' join a queue group, so you can
use nats bench for example to simulate a bunch of load balanced server processes.

First let's publish some messages into a stream, nats bench will automatically create a
stream called benchstream using default attributes.

```
nats bench foo --pub 1 --request --msgs 10000
```
```
23:47:35 Benchmark in request-reply mode
23:47:35 Starting request-reply benchmark [msgs=10,000, msgsize=128 B, pu
23:47:35 Starting publisher, publishing 10,000 messages
Finished 1s [=======================================================
Pub stats: 8,601 msgs/sec ~ 1.05 MB/sec
```
```
nats bench bar --js --pub 1 --size 16 --msgs 1000000
```
```
00:00:10 Starting JetStream benchmark [msgs=1,000,000, msgsize=16 B, pubs
00:00:10 Starting publisher, publishing 1,000,000 messages
Finished 3s [=======================================================
Pub stats: 272,497 msgs/sec ~ 4.16 MB/sec
```
#### Run JetStream benchmarks

##### Measure JetStream publication performance


We can now measure the speed of replay of messages stored in the stream to a
consumer

By default nats bench --js subscribers use 'ordered push' consumers, which are
ordered, reliable and ow controlled but not 'acknowledged' meaning that the subscribers
do not send an acknowledgement back to the server upon receiving each message from
the stream. Ordered push consumers are the preferred way for a single application
instance to get it's own copy of all (or some) of the data stored in a stream. However, you
can also benchmark 'pull consumers', which are instead the preferred way to horizontally
scale the processing (or consumption) of the messages in the stream where the
subscribers do acknowledge the processing of every single message, but can leverage
batching to increase the processing throughput.

Don't be afraid to test different JetStream storage and replication options (assuming you
have access to a JetStream enabled cluster of servers if you want to go beyond
--replicas 1), and of course the number of publishing/subscribing threads and the
publish or pull subscribe batch sizes.

Note: If you change the attributes of a stream between runs you will have to delete the
stream (e.g. run nats stream rm benchstream)

```
nats bench bar --js --sub 1 --msgs 1000000
```
```
00:05:04 JetStream ordered push consumer mode: subscribers will not acknow
00:05:04 Starting JetStream benchmark [msgs=1,000,000, msgsize=128 B, pub
00:05:04 Starting subscriber, expecting 1,000,000 messages
Finished 1s [=======================================================
Sub stats: 777,480 msgs/sec ~ 94.91 MB/sec
```
##### Measure JetStream consumption (replay) performance

**Push and pull consumers**

##### Play around with the knobs


Once you have nished benchmarking streams, remember that if you have stored many
messages in the stream (which is very easy and fast to do) your stream may end up using
a certain amount of resources on the nats-server(s) infrastructure (i.e. memory and les)
that you may want to reclaim.

You can instruct use the --purge bench command ag to tell nats to purge the
stream of messages before starting its benchmark, or purge the stream manually using
nats stream purge benchstream or just delete it altogether using
nats stream rm benchstream.

##### Leave no trace: clean up the resources when you are

##### finished


### nk

nk is a command line tool that generates nkeys. NKeys are a highly secure public-key
signature system based on Ed25519.

With NKeys the server can verify identity without ever storing secrets on the server. The
authentication system works by requiring a connecting client to provide its public key and
digitally sign a challenge with its private key. The server generates a random challenge
with every connection request, making it immune to playback attacks. The generated
signature is validated a public key, thus proving the identity of the client. If the public key
validation succeeds, authentication succeeds.

```
NKey is an awesome replacement for token authentication, because a connecting
client will have to prove it controls the private key for the authorized public key.
```
To get started with NKeys, you’ll need the nk tool from https://github.com/nats-
io/nkeys/tree/master/nk repository. If you have go installed, enter the following at a
command prompt:

To generate a User NKEY:

```
go install github.com/nats-io/nkeys/nk@latest
```
```
nk -gen user -pubout
```
```
SUACSSL3UAHUDXKFSNVUZRF5UHPMWZ6BFDTJ7M6USDXIEDNPPQYYYCU3VY
UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4
```
#### Installing nk

#### Generating NKeys and Configuring the Server


The rst output line starts with the letter S for Seed. The second letter U stands for
User. Seeds are private keys; you should treat them as secrets and guard them with care.

The second line starts with the letter U for User, and is a public key which can be safely
shared.

To use nkey authentication, add a user, and set the nkey property to the public key of
the user you want to authenticate. You are only required to use the public key and no
other properties are required. Here is a snippet of conguration for the nats-server:

To complete the end-to-end conguration and use an nkey, the client is congured to
use the seed, which is the private key.

```
authorization: {
users: [
{ nkey: UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4 }
]
}
```

### nsc

NATS account congurations are built using the nsc tool. The NSC tool allows you to:

Installing nsc is easy:

```
Additional ways of installing nsc are described at nsc's github repository
```
The script will download the latest version of nsc and install it into your system.

In case NSC is not initialized already do nsc init

Output of tree -L 2 nsc/

```
Create and edit Operators, Accounts, Users
Manage publish and subscribe permissions for Users
Dene Service and Stream exports from an account
Reference Service and Streams from another account
Generate Activation tokens that grants access to a private service or stream
Generate User credential les
Describe Operators, Accounts, Users, and Activations
Push and pull account JWTs to an account JWTs server
```
```
curl -L https://raw.githubusercontent.com/nats-io/nsc/master/install.py |
```
#### Installation


**IMPORTANT** : nsc version 2.2.0 has been released. This version of nsc only supports
nats-server v2.2.0 and nats-account-server v1.0.0. For more information please
refer to the nsc 2.2.0 release notes.

You can nd various task-oriented tutorials to working with the tool here:

For more specic browsing of the tool syntax, check out the nsc tool documentation. It
can be found within the tool itself:

Or an online version here.

```
nsc/
```
###### ├── accounts

###### │ ├── nats

###### │ └── nsc.json

###### └── nkeys

###### ├── creds

###### └── keys

```
5 directories, 1 file
```
```
Basic Usage
Conguring Account Streams Import/Export
Conguring Account Services Import/Export
Signing Keys
Revoking Users or Activations
Working with Managed Operators
```
```
nsc help
```
#### Tutorials

#### Tool Documentation


### Basics

NSC allows you to manage identities. Identities take the form of nkeys. Nkeys are a
public-key signature system based on Ed25519 for the NATS ecosystem.

The nkey identities are associated with NATS conguration in the form of a JSON Web
Token (JWT). The JWT is digitally signed by the private key of an issuer forming a chain
of trust. The nsc tool creates and manages these identities and allows you to deploy
them to a JWT account server, which in turn makes the congurations available to nats-
servers.

There’s a logical hierarchy to the entities:

NSC allows you to create, edit, and delete these entities, and will be central to all account-
based conguration.

In this guide, you’ll run end-to-end on some of the conguration scenarios:

Let’s run through the process of creating some identities and JWTs and work through the
process.

```
Operators are responsible for running nats-servers, and issuing account JWTs.
Operators set the limits on what an account can do, such as the number of
connections, data limits, etc.
Accounts are responsible for issuing user JWTs. An account denes streams and
services that can be exported to other accounts. Likewise, they import streams and
services from other accounts.
Users are issued by an account, and encode limits regarding usage and
authorization over the account's subject space.
```
```
Generate NKey identities and their associated JWTs
Make JWTs accessible to a nats-server
Congure a nats-server to use JWTs
```
#### Creating an Operator, Account and User


Let’s create an operator called MyOperator.

There is an additional switch --sys that sets up the system account which is required
for interacting with the NATS server. You can create and set the system account later.

With the above command, the tool generated an NKEY for the operator, stored the private
key safely in its keystore.

Lets add a service URL to the operator. Service URLs specify where the nats-server is
listening. Tooling such as nsc can make use of that conguration:

Creating an account is just as easy:

As expected, the tool generated an NKEY representing the account and stored the private
key safely in the keystore.

Finally, let's create a user:

```
nsc add operator MyOperator
```
```
[ OK ] generated and stored operator key "ODSWWTKZLRDFBPXNMNAY7XB2BIJ45SV
[ OK ] added operator "MyOperator"
[ OK ] When running your own nats-server, make sure they run at least ver
```
```
nsc edit operator --service-url nats://localhost:4222
```
```
[ OK ] added service url "nats://localhost:4222"
[ OK ] edited operator "MyOperator"
```
```
nsc add account MyAccount
```
```
[ OK ] generated and stored account key "AD2M34WBNGQFYK37IDX53DPRG74RLLT7
[ OK ] added account "MyAccount"
```
```
nsc add user MyUser
```

As expected, the tool generated an NKEY representing the user, and stored the private key
safely in the keystore. In addition, the tool generated a credentials le. A credentials le
contains the JWT for the user and the private key for the user. Credential les are used by
NATS clients to identify themselves to the system. The client will extract and present the
JWT to the nats-server and use the private key to verify its identity.

NSC manages three different directories:

The stores directory contains a number of directories. Each named by an operator in
question, which in turn contains all accounts and users:

```
[ OK ] generated and stored user key "UAWBXLSZVZHNDIURY52F6WETFCFZLXYUEFJA
[ OK ] generated user creds file `~/.nkeys/creds/MyOperator/MyAccount/MyU
[ OK ] added user "MyUser" to account "MyAccount"
```
```
The nsc home directory which stores nsc related data. By default nsc home lives in
~/.nsc and can be changed via the $NSC_HOME environment variable.
An nkeys directory, which stores all the private keys. This directory by default lives in
~/.nkeys and can be changed via the $NKEYS_PATH environment variable. The
contents of the nkeys directory should be treated as secrets.
A stores directory, which contains JWTs representing the various entities. This
directory lives in $NSC_HOME/nats, and can be changed using the command
nsc env -s <dir>. The stores directory can stored under revision control. The JWTs
themselves do not contain any secrets.
```
```
tree ~/.nsc/nats
```
##### NSC Assets

**The NSC Stores Directory**


These JWTs are the same artifacts that the NATS servers will use to check the validity of
an account, its limits, and the JWTs that are presented by clients when they connect to
the nats-server.

The nkeys directory contains all the private keys and credential les. As mentioned
before, care must be taken to keep these les secure.

The structure keys directory is machine friendly. All keys are sharded by their kind O for
operators, A for accounts, U for users. These prex es are also part of the public key.
The second and third letters in the public key are used to create directories where other
like-named keys are stored.

```
/Users/myusername/.nsc/nats
```
###### └── MyOperator

###### ├── MyOperator.jwt

###### └── accounts

###### └── MyAccount

###### ├── MyAccount.jwt

###### └── users

###### └── MyUser.jwt

```
tree ~/.nkeys
```
```
/Users/myusername/.nkeys
```
###### ├── creds

###### │ └── MyOperator

###### │ └── MyAccount

###### │ └── MyUser.creds

###### └── keys

###### ├── A

###### │ └── DE

###### │ └── ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYCBW7RRNFTV5NG

###### ├── O

###### │ └── AF

###### │ └── OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4HB7ACPE4RTJP

###### └── U

###### └── DB

###### └── UDBD5FNQPSLIO6CDMIS5D4EBNFKYWVDNULQTFTUZJXWFNYLGFF52VZN7

**The NKEYS Directory**


The nk les themselves are named after the complete public key, and stored in a single
string - the private key in question:

The private keys are encoded into a string, and always begin with an S for seed. The
second letter starts with the type of key in question. O for operators, A for accounts,
U for users.

In addition to containing keys, the nkeys directory contains a creds directory. This
directory is organized in a way friendly to humans. It stores user credential les or
creds les for short. A credentials le contains a copy of the user JWT and the private
key for the user. These les are used by NATS clients to connect to a NATS server:

You can list the current entities you are working with by doing:

```
cat ~/.nkeys/keys/U/DB/UDBD5FNQPSLIO6CDMIS5D4EBNFKYWVDNULQTFTUZJXWFNYLGFF
```
```
SUAG35IAY2EF5DOZRV6MUSOFDGJ6O2BQCZHSRPLIK6J3GVCX366BFAYSNA
```
```
cat ~/.nkeys/creds/MyOperator/MyAccount/MyUser.creds
```
```
-----BEGIN NATS USER JWT-----
eyJ0eXAiOiJKV1QiLCJhbGciOiJlZDI1NTE5LW5rZXkifQ.eyJqdGkiOiI0NUc3MkhIQUVCRF
------END NATS USER JWT------
************************* IMPORTANT *************************
NKEY Seed printed below can be used to sign and prove identity.
NKEYs are sensitive and should be treated as secrets.
-----BEGIN USER NKEY SEED-----
SUAP2AY6UAWHOXJBWDNRNKJ2DHNC5VA2DFJZTF6C6PMLKUCOS2H2E2BA2E
------END USER NKEY SEED------
*************************************************************
```
```
nsc list keys
```
##### Listing Keys


The different entity names are listed along with their public key, and whether the key is
stored. Stored keys are those that are found in the nkeys directory.

In some cases you may want to view the private keys:

If you don't have the seed (perhaps you don't control the operator), nsc will decorate the
row with a!. If you have more than one account, you can show them all by specifying
the --all ag.

You can view a human readable version of the JWT by using nsc:

```
+------------------------------------------------------------------------
| Keys
+------------+----------------------------------------------------------+
| Entity | Key |
+------------+----------------------------------------------------------+
| MyOperator | ODSWWTKZLRDFBPXNMNAY7XB2BIJ45SV756BHUT7GX6JQH6W7AHVAFX6C |
| MyAccount | AD2M34WBNGQFYK37IDX53DPRG74RLLT7FFWBOBMBUXMAVBCVAU5VKWIY |
| MyUser | UAWBXLSZVZHNDIURY52F6WETFCFZLXYUEFJAHRXDW7D2K4445IY4BVXP |
+------------+----------------------------------------------------------+
```
```
nsc list keys --show-seeds
```
```
+------------------------------------------------------------------------
| Seeds Keys
+------------+-----------------------------------------------------------
| Entity | Private Key
+------------+-----------------------------------------------------------
| MyOperator | SOAJ3JDZBE6JKJO277CQP5RIAA7I7HBI44RDCMTIV3TQRYQX35OTXSMHAE
| MyAccount | SAAACXWSQIKJ4L2SEAUZJR3BCNSRCN32V5UJSABCSEP35Q7LQRPV6F4JPI
| MyUser | SUAP2AY6UAWHOXJBWDNRNKJ2DHNC5VA2DFJZTF6C6PMLKUCOS2H2E2BA2E
+------------+-----------------------------------------------------------
[! ] seed is not stored
[ERR] error reading seed
```
```
nsc describe operator
```
#### The Operator JWT


Since the operator JWT is just a JWT you can use other tools, such as jwt.io to decode a
JWT and inspect its contents. All JWTs have a header, payload, and signature:

All NATS JWTs will use the algorithm ed25519 for signature. The payload will list
different things. On our basically empty operator, we will only have standard JWT claim
elds:

jti - a jwt id iat - the timestamp when the JWT was issued in UNIX time iss - the
issuer of the JWT, in this case the operator's public key sub - the subject or identity
represented by the JWT, in this case the same operator type - since this is an operator
JWT, operator is the type

```
+------------------------------------------------------------------------
| Operator Details
+-----------------------+------------------------------------------------
| Name | MyOperator
| Operator ID | ODSWWTKZLRDFBPXNMNAY7XB2BIJ45SV756BHUT7GX6JQH6W
| Issuer ID | ODSWWTKZLRDFBPXNMNAY7XB2BIJ45SV756BHUT7GX6JQH6W
| Issued | 2021-10-27 22:58:28 UTC
| Expires |
| Operator Service URLs | nats://localhost:4222
| Require Signing Keys | false
+-----------------------+------------------------------------------------
```
```
{
"typ": "jwt",
"alg": "ed25519"
}
{
"jti": "ZP2X3T2R57SLXD2U5J3OLLYIVW2LFBMTXRPMMGISQ5OF7LANUQPQ",
"iat": 1575468772,
"iss": "OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4HB7ACPE4RTJPG",
"name": "O",
"sub": "OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4HB7ACPE4RTJPG",
"type": "operator",
"nats": {
"operator_service_urls": [
"nats://localhost:4222"
]
}
}
```

NATS specic is the nats object, which is where we add NATS specic JWT
conguration to the JWT claim.

Because the issuer and subject are one and the same, this JWT is self-signed.

Again we can inspect the account:

Finally the user JWT:

```
nsc describe account
```
```
+------------------------------------------------------------------------
| Account Details
+---------------------------+--------------------------------------------
| Name | MyAccount
| Account ID | AD2M34WBNGQFYK37IDX53DPRG74RLLT7FFWBOBMBUXMA
| Issuer ID | ODSWWTKZLRDFBPXNMNAY7XB2BIJ45SV756BHUT7GX6J
| Issued | 2021-10-27 22:59:01 UTC
| Expires |
+---------------------------+--------------------------------------------
| Max Connections | Unlimited
| Max Leaf Node Connections | Unlimited
| Max Data | Unlimited
| Max Exports | Unlimited
| Max Imports | Unlimited
| Max Msg Payload | Unlimited
| Max Subscriptions | Unlimited
| Exports Allows Wildcards | True
| Response Permissions | Not Set
+---------------------------+--------------------------------------------
| Jetstream | Disabled
+---------------------------+--------------------------------------------
| Imports | None
| Exports | None
+---------------------------+--------------------------------------------
```
##### The Account JWT

##### The User JWT


The user id is the public key for the user, the issuer is the account. This user can publish
and subscribe to anything, as no limits are set.

When a user connects to a nats-server, it presents it's user JWT and signs a nonce using
its private key. The server veries if the user is who they say they are by validating that the
nonce was signed using the private key associated with the public key, representing the
identify of the user. Next, the server fetches the issuer account and validates that the
account was issued by a trusted operator completing the chain of trust verication.

Let’s put all of this together, and create a simple server conguration that accepts
sessions from U.

To congure a server to use accounts, you need to congure it to select the type of
account resolver it will use. The preferred option being to congure the server to use the
built-in NATS Based Resolver.

```
nsc describe user
```
```
+------------------------------------------------------------------------
| User
+----------------------+-------------------------------------------------
| Name | MyUser
| User ID | UAWBXLSZVZHNDIURY52F6WETFCFZLXYUEFJAHRXDW7D2K444
| Issuer ID | AD2M34WBNGQFYK37IDX53DPRG74RLLT7FFWBOBMBUXMAVBCVA
| Issued | 2021-10-27 22:59:21 UTC
| Expires |
| Bearer Token | No
| Response Permissions | Not Set
+----------------------+-------------------------------------------------
| Max Msg Payload | Unlimited
| Max Data | Unlimited
| Max Subs | Unlimited
| Network Src | Any
| Time | Any
+----------------------+-------------------------------------------------
```
#### Account Server Configuration


If you don’t have a nats-server installed, let’s do that now:

Let’s create a conguration that references our operator JWT and the nats-account-server
as a resolver. You can use nsc itself to generate the security part of the server
conguration that you can just add to your nats-server cong le.

For example to use the NATS resolver (which is the recommended resolver conguration)
use nsc generate config --nats-resolver.

Edit this generated conguration as needed (e.g. adjust the location where the server will
store the JWTs in resolver.dir) and paste it into your nats-server conguration (or
save it to a le and import that le from within you server cong le).

At minimum, the server requires the operator JWT, which we have pointed at directly,
and a resolver.

e.g.

And example server cong myconfig.cfg

Now start this local test server using nats-server -c myconfig.cfg

```
go get github.com/nats-io/nats-server
```
```
nsc generate config --nats-resolver > resolver.conf
```
```
server_name: servertest
listen: 127.0.0.1:4222
http: 8222
jetstream: enabled
include resolver.conf
```
#### NATS Server Configuration


The nats-server requires a designated account for operations and monitoring of the
server, cluster, or supercluster. If you see this error message:

```
nats-server: using nats based account resolver - the system account needs
to be specified in configuration or the operator jwt
```
Then there is no system account to interact with the server and you need to add one to
the conguration or operator JWT. Let’s add one to the operator JWT using nsc:

(and re-generate resolver.conf)

Now start the local test server using: nats-server -c myconfig.cfg

In order for the nats servers to know about the account(s) you have created or changes to
the attributes for those accounts, you need to push any new accounts or any changes to
account attributes you may have done locally using nsc into the built-in account
resolver of the nats-server. You can do this using nsc push:

For example to push the account named 'MyAccount' that you have just created into the
nats server running locally on your machine use:

You can also use nsc pull -u nats://localhost to pull the view of the accounts that
the local NATS server has into your local nsc copy (i.e. in ~/.nsc)

As soon as you 'push' an the account JWT to the server (that server's built-in NATS
account resolver will take care of distributing that new (or new version of) the account
JWT to the other nats servers in the cluster) then the changes will take effect and for

```
nsc add account -n SYS`
nsc edit operator --system-account SYS
```
```
nsc push -a MyAccount -u nats://localhost
```
#### Pushing the local nsc changes to the nats

#### server


example any users you may have created with that account will then be able to connect
to any of the nats server in the cluster using the user's JWT.

Install the nats CLI Tool if you haven't already.

Create a subscriber:

Publish a message:

Subscriber shows:

If you are going to use those credentials with nats you should create a context so you
don't have to pass the connection and authentication arguments each time:

To make it easier to work, you can use the NATS clients built right into NSC. These tools
know how to nd the credential les in the keyring. For convenience, the tools are aliased
to sub, pub, req, reply:

```
nats sub --creds ~/.nkeys/creds/MyOperator/MyAccount/MyUser.creds ">"
```
```
nats pub --creds ~/.nkeys/creds/MyOperator/MyAccount/MyUser.creds hello NA
```
```
Received on [hello]: ’NATS’
```
```
nats context add myuser --creds ~/.nkeys/creds/MyOperator/MyAccount/MyUse
```
#### Client Testing

##### Create a nats context

##### NSC Embeds NATS tooling


See nsc tool -h for more detailed information.

User authorization, as expected, also works with JWT authentication. With nsc you can
specify authorization for specic subjects to which the user can or cannot publish or
subscribe. By default a user doesn't have any limits on the subjects that it can publish or
subscribe to. Any message stream or message published in the account is subscribable
by the user. The user can also publish to any subject or imported service. Note that
authorization, if congured, must be specied on a per user basis.

When specifying limits it is important to remember that clients by default use generated
"inboxes" to allow publish requests. When specifying subscribe and publish permissions,
you need to enable clients to subscribe and publish to _INBOX.>. You can further
restrict it, but you'll be responsible for segmenting the subject space so as to not break
request-reply communications between clients.

Let's say you have a service that your account clients can make requests to under q. To
enable the service to receive and respond to requests it requires permissions to
subscribe to q and publish permissions under _INBOX.>:

```
nsc sub --user MyUser ">"
...
nsc pub --user MyUser hello NATS
...
```
```
nsc add user s --allow-pub "_INBOX.>" --allow-sub q
```
```
[ OK ] added pub pub "_INBOX.>"
[ OK ] added sub "q"
[ OK ] generated and stored user key "UDYQFIF75SQU2NU3TG4JXJ7C5LFCWAPXX5S
[ OK ] generated user creds file `~/.nkeys/creds/MyOperator/MyAccount/s.c
[ OK ] added user "s" to account "MyAccount"
```
```
nsc describe user s
```
#### User Authorization


As you can see, this client is now limited to publishing responses to _INBOX.>
addresses and subscribing to the service's request subject.

Similarly, we can limit a client:

Lets look at that new user

```
+------------------------------------------------------------------------
| User
+----------------------+-------------------------------------------------
| Name | s
| User ID | UDYQFIF75SQU2NU3TG4JXJ7C5LFCWAPXX5SSRB276YQOOFXH
| Issuer ID | AD2M34WBNGQFYK37IDX53DPRG74RLLT7FFWBOBMBUXMAVBCVA
| Issued | 2021-10-27 23:23:16 UTC
| Expires |
| Bearer Token | No
+----------------------+-------------------------------------------------
| Pub Allow | _INBOX.>
| Sub Allow | q
| Response Permissions | Not Set
+----------------------+-------------------------------------------------
| Max Msg Payload | Unlimited
| Max Data | Unlimited
| Max Subs | Unlimited
| Network Src | Any
| Time | Any
+----------------------+-------------------------------------------------
```
```
nsc add user c --allow-pub q --allow-sub "_INBOX.>"
```
```
[ OK ] added pub pub "q"
[ OK ] added sub "_INBOX.>"
[ OK ] generated and stored user key "UDIRTIVVHCW2FLLDHTS27ENXLVNP4EO4Z5M
[ OK ] generated user creds file `~/.nkeys/creds/MyOperator/MyAccount/c.c
[ OK ] added user "c" to account "MyAccount"
```
```
nsc describe user c
```

The client has the opposite permissions of the service. It can publish on the request
subject q , and receive replies on an inbox.

As your projects become more involved, you may work with one or more accounts. NSC
tracks your current operator and account. If you are not in a directory containing an
operator, account or user, it will use the last operator/account context.

To view your current environment:

```
+------------------------------------------------------------------------
| User
+----------------------+-------------------------------------------------
| Name | c
| User ID | UDIRTIVVHCW2FLLDHTS27ENXLVNP4EO4Z5MR7FZUNXFXWREP
| Issuer ID | AD2M34WBNGQFYK37IDX53DPRG74RLLT7FFWBOBMBUXMAVBCVA
| Issued | 2021-10-27 23:26:09 UTC
| Expires |
| Bearer Token | No
+----------------------+-------------------------------------------------
| Pub Allow | q
| Sub Allow | _INBOX.>
| Response Permissions | Not Set
+----------------------+-------------------------------------------------
| Max Msg Payload | Unlimited
| Max Data | Unlimited
| Max Subs | Unlimited
| Network Src | Any
| Time | Any
+----------------------+-------------------------------------------------
```
```
nsc env
```
#### The NSC Environment


If you have multiple accounts, you can use nsc env --account <account name> to set
the account as the current default. If you have dened NKEYS_PATH or NSC_HOME in the
environment, you'll also see their current effective values. Finally, if you want to set the
stores directory to anything other than the default, you can do
nsc env --store <dir containing an operator>. If you have multiple accounts, you
can try having multiple terminals, each in a directory for a different account.

```
+------------------------------------------------------------------------
| NSC Environment
+--------------------+-----+---------------------------------------------
| Setting | Set | Effective Value
+--------------------+-----+---------------------------------------------
| $NSC_CWD_ONLY | No | If set, default operator/account from cwd on
| $NSC_NO_GIT_IGNORE | No | If set, no .gitignore files written
| $NKEYS_PATH | No | ~/.nkeys
| $NSC_HOME | No | ~/.nsc
| Config | | ~/.nsc/nsc.json
| $NATS_CA | No | If set, root CAs in the referenced file will
| | | If not set, will default to the system trust
+--------------------+-----+---------------------------------------------
| From CWD | | No
| Stores Dir | | ~/.nsc/nats
| Default Operator | | MyOperator
| Default Account | | MyAccount
| Root CAs to trust | | Default: System Trust Store
+--------------------+-----+---------------------------------------------
```

### Streams

To share messages you publish with other accounts, you have to Export a Stream.
Exports are associated with the account performing the export and advertised in
exporting account’s JWT.

To add a stream to your account:

```
Note that we have exported stream with a subject that contains a wildcard. Any
subject that matches the pattern will be exported.
```
To review the stream export:

```
nsc add export --name abc --subject "a.b.c.>"
```
```
[ OK ] added public stream export "abc"
```
```
nsc describe account
```
##### Adding a Public Stream Export


Messages this account publishes on a.b.c.> will be forwarded to all accounts that
import this stream.

Importing a stream enables you to receive messages that are published by a different
Account. To import a Stream, you have to create an Import. To create an Import you need
to know:

###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ A

###### │ Account ID │ ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-05 13:35:42 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Imports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Exports │

###### ├──────┬────────┬─────────┬────────┬──────────

###### │ Name │ Type │ Subject │ Public │ Revocations │ Tracking │

###### ├──────┼────────┼─────────┼────────┼──────────

###### │ abc │ Stream │ a.b.c.> │ Yes │ 0 │ N/A │

###### ╰──────┴────────┴─────────┴────────┴──────────

```
The exporting account’s public key
The subject where the stream is published
You can map the stream’s subject to a different subject
```
##### Importing a Stream


With the required information, we can add an import to the public stream.

```
Notice that messages published by the remote account will be received on the same
subject as they are originally published. Sometimes you would like to prex messages
received from a stream. To add a prex specify --local-subject. Subscribers in
our account can listen to abc.>. For example if --local-subject abc, The
message will be received as abc.a.b.c.>.
```
And verifying it:

```
Self-imports are not valid; you can only import streams from other accounts.
```
```
nsc add account B
```
```
[ OK ] generated and stored account key "AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI
[ OK ] added account "B"
```
```
nsc add import --src-account ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYCB
```
```
[ OK ] added stream import "a.b.c.>"
```
```
nsc describe account
```

Let's also add a user to make requests from the service:

If your NATS servers are congured to use the built-in NATS resolver, remember that you
need to 'push' any account changes you may have done locally using nsc add to the
servers for those changes to take effect.

###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ B

###### │ Account ID │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64EC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-05 13:39:55 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Exports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Imports

###### ├─────────┬────────┬─────────┬──────────────┬─

###### │ Name │ Type │ Remote │ Local/Prefix │ Expires │ From Account │

###### ├─────────┼────────┼─────────┼──────────────┼─

###### │ a.b.c.> │ Stream │ a.b.c.> │ │ │ A │

###### ╰─────────┴────────┴─────────┴──────────────┴─

```
nsc add user b
```
```
[ OK ] generated and stored user key "UDKNTNEL5YD66U2FZZ2B3WX2PLJFKEFHAPJ
[ OK ] generated user creds file "~/.nkeys/creds/O/B/b.creds"
[ OK ] added user "b" to account "B"
```
##### Pushing the changes to the NATS servers


e.g. nsc push -i or nsc push -a B -u nats://localhost

then

If you want to create a stream that is only accessible to accounts you designate, you can
create a private stream. The export will be visible in your account, but subscribing
accounts will require an authorization token that must be created by you and generated
specically for the subscribing account.

The authorization token is simply a JWT signed by your account where you authorize the
client account to import your export.

This is sSimilar to when we dened an export, but this time we added the --private
ag. The other thing to note is that the subject for the request has a wildcard. This
enables the account to map specic subjects to specically authorized accounts.

```
nsc sub --account B --user b "a.b.c.>"
```
```
nsc pub --account A --user U a.b.c.hello world
```
```
nsc add export --subject "private.abc.*" --private --account A
```
```
nsc describe account A
```
##### Testing the Stream

#### Securing Streams

##### Creating a Private Stream Export


For a foreign account to import a private stream, you have to generate an activation
token. In addition to granting permissions to the account, the activation token also allows
you to subset the exported stream's subject.

To generate a token, you'll need to know the public key of the account importing the
service. We can easily nd the public key for account B by running:

###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ A

###### │ Account ID │ ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-05 14:24:02 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Imports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Exports

###### ├───────────────┬────────┬───────────────┬────

###### │ Name │ Type │ Subject │ Public │ Revocations │ Track

###### ├───────────────┼────────┼───────────────┼────

###### │ abc │ Stream │ a.b.c.> │ Yes │ 0 │ N/A

###### │ private.abc.* │ Stream │ private.abc.* │ No │ 0 │ N/A

###### ╰───────────────┴────────┴───────────────┴────

```
nsc list keys --account B
```
##### Generating an Activation Token


The command took the account that has the export ('A'), the public key of account B, the
subject where the stream will publish to account B.

For completeness, the contents of the JWT le look like this:

When decoded it looks like this:

###### ╭─────────────────────────────────────────────

###### │ Keys

###### ├────────┬────────────────────────────────────

###### │ Entity │ Key │ Si

###### ├────────┼────────────────────────────────────

###### │ O │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4HB7ACPE4RTJPG │

###### │ B │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64ECPXSFQZROJMP2H │

###### │ b │ UDKNTNEL5YD66U2FZZ2B3WX2PLJFKEFHAPJ3NWJBFF44PT76Y2RAVFVE │

###### ╰────────┴────────────────────────────────────

```
nsc generate activation --account A --target-account AAM46E3YF5WOZSE5WNYW
```
```
[ OK ] generated "private.abc.*" activation for account "AAM46E3YF5WOZSE5W
[ OK ] wrote account description to "/tmp/activation.jwt"
```
```
cat /tmp/activation.jwt
```
```
-----BEGIN NATS ACTIVATION JWT-----
eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJIS1FPQU9aQkVKS1JYNFJRUV
------END NATS ACTIVATION JWT------
```
```
nsc describe jwt -f /tmp/activation.jwt
```

The token can be shared directly with the client account.

```
If you manage many tokens for many accounts, you may want to host activation
tokens on a web server and share the URL with the account. The benet to the hosted
approach is that any updates to the token would be available to the importing account
whenever their account is updated, provided the URL you host them in is stable.
```
Importing a private stream is more natural than a public one as the activation token given
to you already has all of the necessary details. Note that the token can be an actual le
path or a remote URL.

Describe account B

###### ╭─────────────────────────────────────────────

###### │ Activation

###### ├─────────────────┬───────────────────────────

###### │ Name │ private.abc.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64

###### │ Account ID │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64ECPXSFQZROJM

###### │ Issuer ID │ ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYCBW7RRNFTV5

###### │ Issued │ 2019-12-05 14:26:13 UTC

###### │ Expires │

###### ├─────────────────┼───────────────────────────

###### │ Hash ID │ GWIS5YCSET4EXEOBXVMQKXAR4CLY4IIXFV4MEMRUXPSQ7L4YTZ4Q=

###### ├─────────────────┼───────────────────────────

###### │ Import Type │ Stream

###### │ Import Subject │ private.abc.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64

###### ├─────────────────┼───────────────────────────

###### │ Max Messages │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Network Src │ Any

###### │ Time │ Any

###### ╰─────────────────┴───────────────────────────

```
nsc add import --account B --token /tmp/activation.jwt
```
```
[ OK ] added stream import "private.abc.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6X
```
#### Importing a Private Stream


Testing a private stream is no different than testing a public one:

then

```
nsc describe account B
```
###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ B

###### │ Account ID │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64EC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-05 14:29:16 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Exports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │

###### ├─────────────────────────────────────────────

###### │ Name │

###### ├─────────────────────────────────────────────

###### │ a.b.c.> │

###### │ private.abc.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64ECPXSFQZROJMP2H │

###### ╰─────────────────────────────────────────────

```
nsc tools sub --account B --user b private.abc.AAM46E3YF5WOZSE5WNYWHN3YYI
```
```
nsc tools pub --account A --user U private.abc.AAM46E3YF5WOZSE5WNYWHN3YYI
```
##### Testing the Private Stream


### Services

To share services that other accounts can reach via request reply, you have to Export a
Service. Services are associated with the account performing the replies and are
advertised in the exporting accounts' JWT.

To add a service to your account:

To review the service export:

```
nsc add export --name help --subject help --service
```
```
[ OK ] added public service export "help"
```
```
nsc describe account
```
#### Adding a Public Service Export


Importing a service enables you to send requests to the remote Account. To import a
Service, you have to create an Import. To create an import you need to know:

To learn how to inspect a JWT from an account server, check this article.

###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ A

###### │ Account ID │ ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-04 18:20:42 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Imports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Exports │

###### ├──────┬─────────┬─────────┬────────┬─────────

###### │ Name │ Type │ Subject │ Public │ Revocations │ Tracking │

###### ├──────┼─────────┼─────────┼────────┼─────────

###### │ help │ Service │ help │ Yes │ 0 │ - │

###### ╰──────┴─────────┴─────────┴────────┴─────────

```
The exporting account’s public key
The subject the service is listening on
You can map the service’s subject to a different subject
Self-imports are not valid; you can only import services from other accounts.
```
#### Importing a Service


First let's create a second account to import the service into:

Add the import of the subject 'help'

Verifying our work:

```
nsc add account B
```
```
[ OK ] generated and stored account key "AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI
[ OK ] added account "B"
```
```
nsc add import --src-account ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYCB
```
```
[ OK ] added service import "help"
```
```
nsc describe account
```

Let's also add a user to make requests from the service:

If your nats servers are congured to use the built-in NATS resolver, remember that you
need to 'push' any account changes you may have done (locally) using nsc add to the
servers for those changes to take effect.

###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ B

###### │ Account ID │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64EC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-04 20:12:42 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Exports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Imports

###### ├──────┬─────────┬────────┬──────────────┬────

###### │ Name │ Type │ Remote │ Local/Prefix │ Expires │ From Account │ Pu

###### ├──────┼─────────┼────────┼──────────────┼────

###### │ help │ Service │ help │ help │ │ A │ Yes

###### ╰──────┴─────────┴────────┴──────────────┴────

```
nsc add user b
```
```
[ OK ] generated and stored user key "UDKNTNEL5YD66U2FZZ2B3WX2PLJFKEFHAPJ
[ OK ] generated user creds file "~/.nkeys/creds/O/B/b.creds"
[ OK ] added user "b" to account "B"
```
##### Pushing the changes to the nats servers


e.g. nsc push -i or nsc push -a B -u nats://localhost

To test the service, we can install the 'nats' CLI tool:

Set up a process to handle the request. This process will run from account 'A' using user
'U':

Remember you can also do:

Send the request:

The service receives the request:

And the response is received by the requestor:

Or more simply:

```
nats reply --creds ~/.nkeys/creds/O/A/U.creds help "I will help"
```
```
nsc reply --account A --user U help "I will help"
```
```
nats request --creds ~/.nkeys/creds/O/B/b.creds help me
```
```
Received on [help]: 'me'
```
```
Received [_INBOX.v6KAX0v1bu87k49hbg3dgn.StIGJF0D] : 'I will help'
```
```
nsc reply --account A --user U help "I will help"
nsc req --account B --user b help me
```
```
published request: [help] : 'me'
received reply: [_INBOX.GCJltVq1wRSb5FoJrJ6SE9.w8utbBXR] : 'I will help'
```
##### Testing the Service


If you want to create a service that is only accessible to accounts you designate you can
create a private service. The export will be visible in your account, but subscribing
accounts will require an authorization token that must be created by you and generated
specically for the requesting account. The authorization token is simply a JWT signed by
your account where you authorize the client account to import your service.

As before, we declared an export, but this time we added the --private ag. The other
thing to note is that the subject for the request has a wildcard. This enables the account
to map specic subjects to specically authorized accounts.

```
nsc add export --subject "private.help.*" --private --service --account A
```
```
[ OK ] added private service export "private.help.*"
```
```
nsc describe account A
```
#### Securing Services

##### Creating a Private Service Export


For the foreign account to import a private service and be able to send requests, you have
to generate an activation token. The activation token in addition to granting permission to
the account allows you to subset the service’s subject:

To generate a token, you’ll need to know the public key of the account importing the
service. We can easily nd the public key for account B by running:

###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ A

###### │ Account ID │ ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-04 20:19:19 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Imports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Exports

###### ├────────────────┬─────────┬────────────────┬─

###### │ Name │ Type │ Subject │ Public │ Revocations │ Tr

###### ├────────────────┼─────────┼────────────────┼─

###### │ help │ Service │ help │ Yes │ 0 │ -

###### │ private.help.* │ Service │ private.help.* │ No │ 0 │ -

###### ╰────────────────┴─────────┴────────────────┴─

```
nsc list keys --account B
```
##### Generating an Activation Token


The command took the account that has the export ('A'), the public key of account B, the
subject where requests from account B will be handled, and an output le where the
token can be stored. The subject for the export allows the service to handle all requests
coming in on private.help.*, but account B can only request from a specic subject.

For completeness, the contents of the JWT le looks like this:

When decoded it looks like this:

###### ╭─────────────────────────────────────────────

###### │ Keys

###### ├────────┬────────────────────────────────────

###### │ Entity │ Key │ Si

###### ├────────┼────────────────────────────────────

###### │ O │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4HB7ACPE4RTJPG │

###### │ B │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64ECPXSFQZROJMP2H │

###### │ b │ UDKNTNEL5YD66U2FZZ2B3WX2PLJFKEFHAPJ3NWJBFF44PT76Y2RAVFVE │

###### ╰────────┴────────────────────────────────────

```
nsc generate activation --account A --target-account AAM46E3YF5WOZSE5WNYW
```
```
[ OK ] generated "private.help.*" activation for account "AAM46E3YF5WOZSE
[ OK ] wrote account description to "/tmp/activation.jwt"
```
```
cat /tmp/activation.jwt
```
```
-----BEGIN NATS ACTIVATION JWT-----
eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJUS01LNEFHT1pOVERDTERGU
------END NATS ACTIVATION JWT------
```
```
nsc describe jwt -f /tmp/activation.jwt
```

The token can be shared directly with the client account.

```
If you manage many tokens for many accounts, you may want to host activation
tokens on a web server and share the URL with the account. The benet to the hosted
approach is that any updates to the token would be available to the importing account
whenever their account is updated, provided the URL you host them in is stable. When
using a JWT account server, the tokens can be stored right on the server and shared
by an URL that is printed when the token is generated.
```
Importing a private service is more natural than a public one because the activation token
stores all the necessary details. Again, the token can be an actual le path or a remote
URL.

###### ╭─────────────────────────────────────────────

###### │ Activation

###### ├─────────────────┬───────────────────────────

###### │ Name │ private.help.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q6

###### │ Account ID │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64ECPXSFQZROJM

###### │ Issuer ID │ ADETPT36WBIBUKM3IBCVM4A5YUSDXFEJPW4M6GGVBYCBW7RRNFTV5

###### │ Issued │ 2019-12-04 20:33:30 UTC

###### │ Expires │

###### ├─────────────────┼───────────────────────────

###### │ Hash ID │ DD6BZKI2LTQKAJYD5GTSI4OFUG72KD2BF74NFVLUNO47PR4OX64Q=

###### ├─────────────────┼───────────────────────────

###### │ Import Type │ Service

###### │ Import Subject │ private.help.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q6

###### ├─────────────────┼───────────────────────────

###### │ Max Messages │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Network Src │ Any

###### │ Time │ Any

###### ╰─────────────────┴───────────────────────────

```
nsc add import --account B -u /tmp/activation.jwt --local-subject private
```
#### Importing a Private Service


Describe account B

When importing a service, you can specify the local subject you want to use to make
requests. The local subject in this case is private.help. However when the request is
forwarded by NATS, the request is sent on the remote subject.

```
[ OK ] added service import "private.help.AAM46E3YF5WOZSE5WNYWHN3YYISVZOS
```
```
nsc describe account B
```
###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ B

###### │ Account ID │ AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI6XHTF2Q64EC

###### │ Issuer ID │ OAFEEYZSYYVI4FXLRXJTMM32PQEI3RGOWZJT7Y3YFM4

###### │ Issued │ 2019-12-04 20:38:06 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Exports │ None

###### ╰───────────────────────────┴─────────────────

###### ╭─────────────────────────────────────────────

###### │ Imp

###### ├──────────────┬─────────┬────────────────────

###### │ Name │ Type │ Remote

###### ├──────────────┼─────────┼────────────────────

###### │ help │ Service │ help

###### │ private.help │ Service │ private.help.AAM46E3YF5WOZSE5WNYWHN3YYISVZOSI

###### ╰──────────────┴─────────┴────────────────────

##### Testing the Private Service


Testing a private service is no different than a public one:

```
nsc reply --account A --user U "private.help.*" "help is here"
nsc req --account B --user b private.help help_me
```
```
published request: [private.help] : 'help_me'
received reply: [_INBOX.3MhS0iCHfqO8wUl1x59bHB.jpE2jvEj] : 'help is here'
```

### Signing Keys

As previously discussed, NKEYs are identities, and if someone gets a hold of an account
or operator nkey they can do everything you can do as you.

NATS has a strategies to let you deal with scenarios where your private keys escape out
in the wild.

The rst and most important line of defense is Signing Keys. Signing Keys allow you have
multiple NKEY identities of the same kind (Operator or Account) that have the same
degree of trust as the standard Issuer nkey.

The concept behind the signing key is that you can issue a JWT for an operator or an
account that lists multiple nkeys. Typically the issuer will match the Subject of the entity
issuing the JWT. With SigningKeys, a JWT is considered valid if it is signed by the Subject
of the Issuer or one of its signing keys. This enables guarding the private key of the
Operator or Account more closely while allowing Accounts, Users or Activation Tokens be
signed using alternate private keys.

If an issue should arise where somehow a signing key escapes into the wild, you would
remove the compromised signing key from the entity, add a new one, and reissue the
entity. When a JWT is validated, if the signing key is missing, the operation is rejected.
You are also on the hook to re-issue all JWTs (accounts, users, activation tokens) that
were signed with the compromised signing key.

This is effectively a large hammer. You can mitigate the process a bit by having a larger
number of signing keys and then rotating the signing keys to get a distribution you can
easily handle in case of a compromise. In a future release, we’ll have a revocation
process were you can invalidate a single JWT by its unique JWT ID (JTI). For now a
sledge hammer you have.

With greater security process, there’s greater complexity. With that said, nsc doesn’t
track public or private signing keys. As these are only identities that when in use presume
a manual use. That means that you the user will have to track and manage your private
keys more closely.

Let’s get a feel for the workow. We are going to:


All signing key operations revolve around the global nsc ag -K or --private-key.
Whenever you want to modify an entity, you have to supply the parent key so that the JWT
is signed. Normally this happens automatically but in the case of signing keys, you’ll have
to supply the ag by hand.

Creating the operator:

To add a signing key we have to rst generate one with nsc:

```
On a production environment private keys should be saved to a le and always
referenced from the secured le.
```
Now we are going to edit the operator by adding a signing key with the --sk ag
providing the generated operator public key (the one starting with O ):

```
Create an operator with a signing key
Create an account with a signing key
The account will be signed using the operator’s signing key
Create an user with the account’s signing key
```
```
nsc add operator O2
```
```
[ OK ] generated and stored operator key "OABX3STBZZRBHMWMIMVHNQVNUG2O3D54
[ OK ] added operator "O2"
```
```
nsc generate nkey --operator --store
```
```
SOAEW6Z4HCCGSLZJYZQMGFQY2SY6ZKOPIAKUQ5VZY6CW23WWYRNHTQWVOA
OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57BXVU6WUTHFEAO3CU5GLQYF5
operator key stored ~/.nkeys/keys/O/AZ/OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57BX
```
```
nsc edit operator --sk OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57BXVU6WUTHFEAO3CU5
```

Check our handy work:

Now let’s create an account called A and sign it with the generated operator private
signing key. To sign it with the key specify the -K ag and the private key or a path to the
private key:

Let’s generate an account signing key, again we use nk:

Let’s add the signing key to the account, and remember to sign the account with the
operator signing key:

```
[ OK ] added signing key "OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57BXVU6WUTHFEAO3
[ OK ] edited operator "O2"
```
```
nsc describe operator
```
###### ╭─────────────────────────────────────────────

###### │ Operator Details

###### ├──────────────┬──────────────────────────────

###### │ Name │ O2

###### │ Operator ID │ OABX3STBZZRBHMWMIMVHNQVNUG2O3D54BMZXX5LMBYKSAPDSHIWPMMFY

###### │ Issuer ID │ OABX3STBZZRBHMWMIMVHNQVNUG2O3D54BMZXX5LMBYKSAPDSHIWPMMFY

###### │ Issued │ 2019-12-05 14:36:16 UTC

###### │ Expires │

###### ├──────────────┼──────────────────────────────

###### │ Signing Keys │ OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57BXVU6WUTHFEAO3CU5GLQYF5

###### ╰──────────────┴──────────────────────────────

```
nsc add account A -K ~/.nkeys/keys/O/AZ/OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57
```
```
[ OK ] generated and stored account key "ACDXQQ6KD5MVSFMK7GNF5ARK3OJC6PEI
[ OK ] added account "A"
```
```
nsc generate nkey --account --store
```
```
SAAA4BVFTJMBOW3GAYB3STG3VWFSR4TP4QJKG2OCECGA26SKONPFGC4HHE
ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMWFREPKNO6MPA7ZETFEEF7
account key stored ~/.nkeys/keys/A/DU/ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMW
```

Let's take a look at the account

We can see that the signing key
ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMWFREPKNO6MPA7ZETFEEF7 was added to the
account. Also the issuer is the operator signing key (specied by the -K).

Now let’s create a user and sign it with the account signing key starting with
ADUQTJD4TF4O.

```
nsc edit account --sk ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMWFREPKNO6MPA7ZET
```
```
[ OK ] added signing key "ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMWFREPKNO6MPA
[ OK ] edited account "A"
```
```
nsc describe account
```
###### ╭─────────────────────────────────────────────

###### │ Account Details

###### ├───────────────────────────┬─────────────────

###### │ Name │ A

###### │ Account ID │ ACDXQQ6KD5MVSFMK7GNF5ARK3OJC6PEICWCH5PQ7HO2

###### │ Issuer ID │ OAZBRNE7DQGDYT5CSAGWDMI5ENGKOEJ57BXVU6WUTHF

###### │ Issued │ 2019-12-05 14:48:22 UTC

###### │ Expires │

###### ├───────────────────────────┼─────────────────

###### │ Signing Keys │ ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMWFREPKNO

###### ├───────────────────────────┼─────────────────

###### │ Max Connections │ Unlimited

###### │ Max Leaf Node Connections │ Unlimited

###### │ Max Data │ Unlimited

###### │ Max Exports │ Unlimited

###### │ Max Imports │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Max Subscriptions │ Unlimited

###### │ Exports Allows Wildcards │ True

###### ├───────────────────────────┼─────────────────

###### │ Imports │ None

###### │ Exports │ None

###### ╰───────────────────────────┴─────────────────


Check the account

As expected, the issuer is now the signing key we generated earlier. To map the user to
the actual account, an Issuer Account eld was added to the JWT that identies the
public key of account A.

Scoped Signing Keys simplify user permission management. Previously if you wanted to
limit the permissions of users, you had to specify permissions on a per-user basis. With
scoped signing keys, you associate a signing key with a set of permissions. This
conguration lives on the account JWT and is managed with the

```
nsc add user U -K ~/.nkeys/keys/A/DU/ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMW
```
```
[ OK ] generated and stored user key "UD47TOTKVDY4IQRGI6D7XMLZPHZVNV5FCD4
[ OK ] generated user creds file "~/.nkeys/creds/O2/A/U.creds"
[ OK ] added user "U" to account "A"
```
```
nsc describe user
```
###### ╭─────────────────────────────────────────────

###### │ User

###### ├──────────────────────┬──────────────────────

###### │ Name │ U

###### │ User ID │ UD47TOTKVDY4IQRGI6D7XMLZPHZVNV5FCD4CNQICLV3FXLQB

###### │ Issuer ID │ ADUQTJD4TF4O6LTTHCKDKSHKGBN2NECCHHMWFREPKNO6MPA7

###### │ Issuer Account │ ACDXQQ6KD5MVSFMK7GNF5ARK3OJC6PEICWCH5PQ7HO27VKGC

###### │ Issued │ 2019-12-05 14:50:07 UTC

###### │ Expires │

###### ├──────────────────────┼──────────────────────

###### │ Response Permissions │ Not Set

###### ├──────────────────────┼──────────────────────

###### │ Max Messages │ Unlimited

###### │ Max Msg Payload │ Unlimited

###### │ Network Src │ Any

###### │ Time │ Any

###### ╰──────────────────────┴──────────────────────

#### Scoped Signing Keys


nsc edit signing-key command. You can add as many scoped signing keys as
necessary.

To issue a user with a set of permissions, simply sign the user with the signing key having
the permission set you want. The user conguration must not have any permissions
assigned to it.

On connect, the nats-server will assign the permissions associated with that signing key
to the user. If you update the permissions associated with a signing key, the server will
immediately update permissions for users signed with that key.

Generate the signing key

Add a service to the account

```
nsc add account A
```
```
[ OK ] generated and stored account key "ADLGEVANYDKDQ6WYXPNBEGVUURXZY4LL
[ OK ] push jwt to account server:
[ OK ] pushed account jwt to the account server
> NGS created a new free billing account for your JWT, A [ADLGEVANY
> Use the 'ngs' command to manage your billing plan.
> If your account JWT is *not* in ~/.nsc, use the -d flag on ngs c
[ OK ] pull jwt from account server
[ OK ] added account "A"
```
```
nsc edit account -n A --sk generate
```
```
[ OK ] added signing key "AAZQXKDPOTGUCOCOGDW7HWWVR5WEGF3KYL7EKOEHW2XWRS2
[ OK ] push jwt to account server
[ OK ] pull jwt from account server
[ OK ] account server modifications:
> allow wildcard exports changed from true to false
[ OK ] edited account "A"
```
```
nsc edit signing-key --account A --role service --sk AAZQXKDPOTGUCOCOGDW7
```

Since the signing key has a unique role name within an account, it can be subsequently
used for easier referencing.

To see the permissions for the user enter nsc describe user - you will see in the report
that the user is scoped, and has the permissions listed. You can inspect and modify the
scoped permissions with nsc edit signing keys - pushing updates to the account will
reassign user permissions

Available as of NATS 2.9.0

Although scoped signing keys are very useful and improve security, by limiting the scope
of a particular signing key, the permissions that are set may be too rigid in multi-user
setups. For example, given two users pam and joe, we may want to allow them to
subscribe to their own namespaced subject in order to service requests, e.g. pam.> and
joe.>. The permission structure is the same between these users, but they differ in the
concrete subjects which are further scoped to some property about that user.

Template functions can be used to declare the structure within a scope signing key, but
utilize basic templating so that each user that is created with the signing key has user-
specic subjects.

The following template functions will expand when a user is created.

```
[ OK ] set max responses to 1
[ OK ] added deny pub ">"
[ OK ] added sub "q.>"
[ OK ] push jwt to account server
[ OK ] pull jwt from account server
[ OK ] edited signing key "AAZQXKDPOTGUCOCOGDW7HWWVR5WEGF3KYL7EKOEHW2XWRS
```
```
nsc add user U -K service
```
```
[ OK ] generated and stored user key "UBFRJ6RNBYJWSVFBS7O4ZW5MM6J3EPE75II
[ OK ] generated user creds file `~/test/issue-2621/keys/creds/synadia/A/
[ OK ] added user "U" to account "A"
```
##### Template functions


For example, given a scoped signing key with a templated --allow-sub subject:

We can create two users in different teams.

The resulting --allow-sub permission per user would be expanded to:

and

```
{{name()}} - expands to the name of the user, e.g. pam
{{subject()}} - expands to the user public nkey value of the user, e.g. UAC...
{{account-name()}} - expands to the signing account name, e.g. sales
{{account-subject()}} - expands to the account public nkey value, e.g. AXU...
{{tag(key)}} - expands key:value tags associated with the signing key
```
```
nsc edit signing-key \
--account sales \
--role team-service \
--sk AXUQXKDPOTGUCOCOGDW7HWWVR5WEGF3KYL7EKOEHW2XWRS2PT5AOTRH3 \
--allow-sub "{{account-name()}}.{{tag(team)}}.{{name()}}.>" \
--allow-pub-response
```
```
nsc add user pam -K team-service --tag team:support
nsc add user joe -K team-service --tag team:leads
```
```
sales.support.pam.>
```
```
sales.leads.joe.>
```

### Revocation

NATS supports two types of revocations. Both of these are stored in the Account JWT, so
that the nats-server can see the revocations and apply them.

Users are revoked by public key and time. Access to an export, called an activation, can
be revoked for a specic account at a specic time. The use of time here can be
confusing, but is designed to support the primary uses of revocation.

When a user or activation is revoked at time T, it means that any user JWT or activation
token created before that time is invalid. If a new user JWT or new activation token is
created after T it can be used. This allows an account owner to revoke a user and renew
their access at the same time.

Let's look at an example. Suppose you created a user JWT with access to the subject
"billing". Later you decide you don't want that user to have access to "billing". Revoke the
user, say at noon on May 1st 2019, and create a new user JWT without access to "billing".
The user can no longer log in with the old JWT because it is revoked, but they can log in
with the new JWT because it was created after noon May 1st 2019.

```
nsc provides a number of commands to create, remove or list revocations:
```
```
nsc revocations -h
```

Both add commands take the ag --at which defaults to 0, for now, which can be used
to set the unix timestamp as described above. By default revocations are at the current
time, but you can set them in the past for situations where you know when a problem
occurred and was xed.

Deleting a revocation is permanent and can allow an old activation or user JWT to be
valid again. Therefore delete should only be used if you are sure the tokens in question
have expired.

If your nats servers are congured to use the built-in NATS resolver, remember that you
need to 'push' any account changes you may have done (locally) using
nsc revocations to the servers for those changes to take effect.

i.e. nsc push -i or nsc push -a B -u nats://localhost

```
Manage revocation for users and activations from an account
Usage:
nsc revocations [command]
Available Commands:
add-user Revoke a user
add_activation Revoke an accounts access to an export
delete-user Remove a user revocation
delete_activation Remove an account revocation from an export
list-users List users revoked in an account
list_activations List account revocations for an export
Flags:
-h, --help help for revocations
Global Flags:
-i, --interactive ask questions for various settings
-K, --private-key string private key
Use "nsc revocations [command] --help" for more information about a comma
```
##### Pushing the changes to the nats servers


If there are any clients currently connected with as a user that gets added to the
revocations, their connections will be immediately terminated as soon as you 'push' your
revocations to a nats server.


### Managed Operators

You can use nsc to administer multiple operators. Operators can be thought of as the
owners of nats-servers, and fall into two categories: local and managed. The key
difference, pardon the pun, is that managed operators are ones which you don't have the
nkey for. An example of a managed operator is the Synadia's NGS.

Accounts, as represented by their JWTs, are signed by the operator. Some operators may
use local copies of JWTs (i.e. using the memory resolver), but most should use the NATS
account resolver built-in to 'nats-server' to manage their JWTs. Synadia uses a custom
server for their JWTs that works similarly to the open-sourced account server.

There are a few special commands when dealing with server based operators:

For managed operators this push/pull behavior is built into nsc. Each time you edit your
account JWT nsc will push the change to a managed operator's server and pull the
signed response. If this fails the JWT on disk may not match the value on the server. You
can always push or pull the account again without editing it. Note - push only works if the
operator JWT was congured with an account server URL.

The managed operator will not only sign your account JWT with its key, but may also edit
the JWT to include limits to constrain your access to their NATS servers. Some operators
may also add demonstration or standard imports. Generally you can remove these,
although the operator gets the nal call on all Account edits. As with any deployment, the
managed operator doesn't track user JWTs.

To start using a managed operator you need to tell nsc about it. There are a couple
ways to do this. First you can manually tell nsc to download the operator JWT using the
add operator command:

```
Account JWTs can be pushed to the server using nsc push
Account JWTs can be pulled from a server using nsc pull
```
```
nsc add operator -i
```

The operator JWT (or details) should be provided to you by the operator. The second way
to add a managed operator is with the init command:

You can use the name of an existing operator, or a well known one (currently only
"synadia").

Once you add a managed operator you can add accounts to it normally, with the caveat
that new accounts are pushed and pulled as described above.

To dene a well known operator, you would tell nsc about an operator that you want
people in your environment to use by name with a simple environment variable of the
form nsc_<operator name>_operator the value of this environment variable should be
the URL for getting the operator JWT. For example:

will tell nsc that there is a well known operator named zoom with its JWT at
https://account-server-host/jwt/v1/operator. With this denition you can now
use the -u ag with the name "zoom" to add the operator to an nsc store directory.

The operator JWT should have its account JWT server property set to point to the
appropriate URL. For our example this would be:

You can also set one or more service urls. These allow the nsc tool actions like pub
and sub to work. For example:

```
nsc init -o synadia -n MyFirstAccount
```
```
export nsc_zoom_operator=https://account-server-host/jwt/v1/operator
```
```
nsc edit operator -u https://account-server-host/jwt/v1
```
```
nsc edit operator -n nats://localhost:4222
nsc tool pub hello world
```
#### Defining "Well Known Operators"


### nats-top

nats-top is a top-like tool for monitoring nats-server servers.

The nats-top tool provides a dynamic real-time view of a NATS server. nats-top can
display a variety of system summary information about the NATS server, such as
subscription, pending bytes, number of messages, and more, in real time. For example:

nats-top can be installed using go install. For example:

With newer versions of Go, you will be required to use
go install github.com/nats-io/nats-top@latest.

NOTE: You may have to run the above command as user sudo depending on your setup.
If you receive an error that you cannot install nats-top because your $GOPATH is not set,
when in fact it is set, use command sudo -E go get github.com/nats-io/nats-top
to install nats-top. The -E ag tells sudo to preserve the current user's environment.

```
nats-top
```
```
nats-server version 0.6.4 (uptime: 31m42s)
Server:
Load: CPU: 0.8% Memory: 5.9M Slow Consumers: 0
In: Msgs: 34.2K Bytes: 3.0M Msgs/Sec: 37.9 Bytes/Sec: 3389.7
Out: Msgs: 68.3K Bytes: 6.0M Msgs/Sec: 75.8 Bytes/Sec: 6779.4
Connections: 4
HOST CID SUBS PENDING MSGS_TO MSGS_FROM
127.0.0.1:56134 2 5 0 11.6K 11.6K
127.0.1.1:56138 3 1 0 34.2K 0
127.0.0.1:56144 4 5 0 11.2K 11.1K
127.0.0.1:56151 5 8 0 11.4K 11.5K
```
```
go install github.com/nats-io/nats-top
```
#### Installation


Once installed, nats-top can be run with the command nats-top and optional
arguments.

Optional arguments inclde the following:

While in nats-top view, you can use the following commands.

Use the o<option> command to set the primary sort key to the <option> value. The
option value can be one of the following: cid, subs, pending, msgs_to, msgs_from
, bytes_to, bytes_from, lang, version.

```
nats-top [-s server] [-m monitor] [-n num_connections] [-d delay_in_secs]
```
```
Option Description
-m monitor Monitoring http port from nats-server.
-n num_connections Limit the connections requested to the server (default 1024).
-d delay_in_secs Screen refresh interval (default 1 second).
-sort by Field to use for sorting the connections (see below).
```
#### Usage

#### Options

#### Commands

##### option


You can also set the sort option on the command line using the -sort ag. F or
example: nats-top -sort bytes_to.

Use the n<limit> command to set the sample size of connections to request from the
server.

You can also set this on the command line using the -n num_connections ag. F or
example: nats-top -n 1.

Note that if n<limit> is used in conjunction with -sort, the server will respect both
options allowing queries such as the following: Query for the connection with largest
number of subscriptions: nats-top -n 1 -sort subs.

Use the s command to toggle displaying connection subscriptions.

Use the? command to show help message with options.

Use the q command to quit nats-top.

For a walkthrough with nats-top check out the tutorial.

##### limit

##### s,? and q Commands

##### Tutorial


### Tutorial

You can use nats-top to monitor in realtime NATS server connections and message
statistics.

You may need to run the following instead:

Result:

```
Set up your Go environment
Install the NATS server
```
```
go install github.com/nats-io/nats-top@latest
```
```
sudo -E go install github.com/nats-io/nats-top
```
```
nats-server -m 8222
```
```
nats-top
```
#### Prerequisites

#### 1. Install nats-top

#### 2. Start the NATS server with monitoring

#### enabled

#### 3. Start nats-top


Run some NATS client programs and exchange messages.

For the best experience, you will want to run multiple subscribers, at least 2 or 3. Refer to
the example pub-sub clients.

In nats-top, enter the command o followed by the option, such as bytes_to. You see
that nats-top sorts the BYTES_TO column in ascending order.

```
nats-server version 0.6.6 (uptime: 2m2s)
Server:
Load: CPU: 0.0% Memory: 6.3M Slow Consumers: 0
In: Msgs: 0 Bytes: 0 Msgs/Sec: 0.0 Bytes/Sec: 0
Out: Msgs: 0 Bytes: 0 Msgs/Sec: 0.0 Bytes/Sec: 0
Connections: 0
HOST CID SUBS PENDING MSGS_TO MSGS_FROM
```
```
nats-server version 0.6.6 (uptime: 30m51s)
Server:
Load: CPU: 0.0% Memory: 10.3M Slow Consumers: 0
In: Msgs: 56 Bytes: 302 Msgs/Sec: 0.0 Bytes/Sec: 0
Out: Msgs: 98 Bytes: 512 Msgs/Sec: 0.0 Bytes/Sec: 0
Connections: 3
HOST CID SUBS PENDING MSGS_TO MSGS_FROM
::1:58651 6 1 0 52 0
::1:58922 38 1 0 21 0
::1:58953 39 1 0 21 0
```
#### 4. Run NATS client programs

#### 5. Check nats-top for statistics

#### 6. Sort nats-top statistics


Use some different sort options to explore nats-top, such as:

```
cid, subs, pending, msgs_to, msgs_from, bytes_to, bytes_from, lang,
version
```
You can also set the sort option on the command line using the -sort ag. F or
example: nats-top -sort bytes_to.

In nats-top, enter the command s to toggle displaying connection subscriptions. When
enabled, you see the subscription subject in nats-top table:

```
nats-server version 0.6.6 (uptime: 45m40s)
Server:
Load: CPU: 0.0% Memory: 10.4M Slow Consumers: 0
In: Msgs: 81 Bytes: 427 Msgs/Sec: 0.0 Bytes/Sec: 0
Out: Msgs: 154 Bytes: 792 Msgs/Sec: 0.0 Bytes/Sec: 0
sort by [bytes_to]:
Connections: 3
HOST CID SUBS PENDING MSGS_TO MSGS_FROM
::1:59259 83 1 0 4 0
::1:59349 91 1 0 2 0
::1:59342 90 1 0 0 0
```
```
nats-server version 0.6.6 (uptime: 1h2m23s)
Server:
Load: CPU: 0.0% Memory: 10.4M Slow Consumers: 0
In: Msgs: 108 Bytes: 643 Msgs/Sec: 0.0 Bytes/Sec: 0
Out: Msgs: 185 Bytes: 1.0K Msgs/Sec: 0.0 Bytes/Sec: 0
Connections: 3
HOST CID SUBS PENDING MSGS_TO MSGS_FROM
::1:59708 115 1 0 6 0
::1:59758 122 1 0 1 0
::1:59817 124 1 0 0 0
```
#### 7. Use different sort options

#### 8. Display the registered subscriptions.


Use the q command to quit nats-top.

For example, to query for the connection with largest number of subscriptions:

Result: nats-top displays only the client connection with the largest number of
subscriptions:

```
nats-top -n 1 -sort subs
```
```
nats-server version 0.6.6 (uptime: 1h7m0s)
Server:
Load: CPU: 0.0% Memory: 10.4M Slow Consumers: 0
In: Msgs: 109 Bytes: 651 Msgs/Sec: 0.0 Bytes/Sec: 0
Out: Msgs: 187 Bytes: 1.0K Msgs/Sec: 0.0 Bytes/Sec: 0
Connections: 3
HOST CID SUBS PENDING MSGS_TO MSGS_FROM
::1:59708 115 1 0 6 0
```
#### 9. Quit nats-top

#### 10. Restart nats-top with a specified query


