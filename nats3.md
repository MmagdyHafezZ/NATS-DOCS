# Configuring NATS Server

While the NATS server has many ags that allow for simple testing of features from
command line. The standard way of conguring the NATS server product is through a
conguration le. We use a simple conguration format that combines the best of
traditional formats and newer styles such as JSON and YAML.

The NATS conguration supports the following syntax:

The NATS conguration le is parsed with UTF-8 encoding.

```
We strongly recommend using only ASCII for names and vno ASCII text to comments. alues, limiting the use of Unicode,
```
The NATS conguration in the le can also be rendered as a JSON object (with
comments!), but to combine it with variables the variables still have to be unquoted.

```
nats-server -config my-server.conf
```
```
Lines can be commented with # and //
Values can be assigned to properties with delimiters:
Equals sign: foo = 2
Colon: foo: 2
Whitespace: foo 2
Arrays are enclosed in brackets: ["a", "b", "c"]
Maps are enclosed in braces: {foo: 2}
Maps can be assigned with no delimiter accounts { SYS {...}, cloud-user {...} }
Semicolons can be optionally used as terminators host: 127.0.0.1; port: 4222;
```
**Note**


```
JSON cong les should be limited machine generated conguration les
```
The conguration parser is very forgiving, as you have seen:

String values that start with a digit can create issues. To force such values as strings,
quote them.
BAD Cong:

Fixed Cong:

Server congurations can specify variables. Variables allow you to reference a value from
one or more sections in the conguration.

```
values can be a primitive, or a list, or a map
strings and numbers typically do the right thing
numbers support units such as, 1K for 1000, 1KB for 1024
```
```
listen: 127.0.0.1:4222authorization: {
# Bad - Number parsing error token: 3secret
}
```
```
listen: 127.0.0.1:4222authorization: {
# Good token: "3secret"
}
```
## Strings and Numbers

## Variables


**Variables syntax:**

**Variable resolution sequence:**

```
nats-server: variable reference for 'PORT' on line 5 can not be found
If the envirdepending on the seronment variable vver version you are running.alue begins with a number you may have trouble resolving it
```
```
Are block-scoped
Are referenced with a a usage like foo = "$VAR1"$ prex. V ariables in quotes blocks ar will result in foo being the litere ignored. For example,al string "$VAR1".
Variables MUSTunknown field be used t from variable bo be recognized as such. y nding a reference to the variable.The cong parser will distinguish
Variable reference which are not dened will be resolved from environment variables.
```
```
Look for variable in same scope
Look for variable in parent scopes
Look for variable in enviroment variables
If not found stop server startup with the error below
```
```
# Define a variable in the configTOKEN: "secret"
# Reference the variableauthorization {
token: $TOKEN}
```
```
# Define a variable in the config# But TOKEN is never used resulting in a config parsing error
TOKEN: "secret"
# Reference the variableauthorization {
token: "another secret"}
```

A similar conguration, but this time, the variable is resolved from the environment:

The include directive allows you to split a server conguration into several les. This is
useful for separating conguration into chunks that you can easily reuse between
different servers.
Includes must use relative paths, and are relative to the main conguration (the one
specied via the -c option):
server.conf:

```
Note that include is not followed by = or : , as it is a directive.
```
auth.conf:

```
unknown field "TOKEN"
```
```
exportnats-server TOKEN =-c"hello" /config/file
```
```
# TOKEN is defined in the environmentauthorization {
token: $TOKEN}
```
```
listen: 127.0.0.1:4222include ./auth.conf
```
```
authorization: { token: "f0oBar"
}
```
## Include Directive


The cong le is being read by the server on startup and is not re-scanned for changes
and not locked.
A server can reload most conguration changes without requiring a server restart or
clients to disconnect by sending the nats-server a signal:

As of NATS v2.10.0, a reload signal can be sent on a NATS service using a system
account user, where <server-id> is the unique ID of the server be targeted.

Cong les have the following structure (in no specic order). All blocks and properties
are optional (except host and port).
Please see sections below for links to detailed explanations of each conguration block

```
> nats-server -c server.conf
```
```
nats-server --signal reload
```
```
nats --user sys --password sys request '$SYS.REQ.SERVER.<server-id>.RELOA
```
## Configuration Reloading

## Configuration Properties


#General settingshost: 0.0.0.
port: 4222
# Various server level options# ...

# The following sections are maps with a set of (nested) properties
jetstream { # Jetstream storage location, limits and encryption

} store_dir: nats
tls { # Configuration map for tls parameters used for client connections,
# routes and https monitoring connections.}

gateway { # Configuration map for gateway. Gateways are used to connected clust
}
leafnodes { # Configuration map for leafnodes. LeafNodes are lightweight clusters
}
mqtt { # Configuration map for mqtt. Allow clients to connect via mqtt proto
}
websocket { # Configuration map for websocket. Allow clients to connect via webso
}
accounts { # List of accounts and user within accounts
# User may have an authorization and authentication section}

authorization { # User may have an authorization and authentication section
# This section is only useful when no accounts are defined}

mappings { # Subject mappings for default account
# When accounts are defined this section must be in the account map


```
# When accounts are defined this section must be in the account map}
resolver { # Pointer to external Authentication/Authorization resolver
# There are multiple possible resolver type explained in their own ch # memory, nats-base, url ... more may be added in the future
# This parameter can be a value `MEMORY` for simple configuration # or a map of properties for connecting to the resolver
}
resolver_tls { # TLS configuration for an URL based resolver
}
resolver_preload { # List of JWT tokens to be loaded at server start.
}
```
```
Property Description Default / Example
host Host for client connections. 0.0.0.
port Port for client connections. 4222
listen Listen specication for client connections. Either use thisor the options host<host>:<port> and/or port.^ Inherits fr0.0.0.0:4222host and omport^
```
```
client_advertise Alternativadvertise to clients and other ser<host>:<port>e client listen specication or just <host>ver. to^
Useful in cluster setups with NAT.
```
```
0.0.0.0:
```
```
tls Congurused for client connections, rand https monitation map for oring connections.tls parametersoutes active by default.Plain text tls {} No tlsTCP/IP.^
```
```
gateway CongurGateways are used tclusters intation map for o superclusters.o connectedgateway.^ None by default.gateway {}^
```
### Connectivity


Note that each accounts forms its own subject namespace. Therefore the mappings
section can appear on the server level (applying to the default account) or on the account
level.

```
Property Description Default / Example
leafnodes CongurLeafNodes aration map for e lightweight clusters.leafnodes.^ None by default.leafnodes {}^
mqtt Congurclients to connect via mqtt pration map for mqtt. Allowotocol.^ active by default.mqtt {} Not^
websocket Conguration map for websocket. active by default.websocket {} Not^
```
```
Property Description Default
cluster CongurNats Serload balancing and ration map for vers can form a cluster foredundancycluster.^.^ active by default.cluster {} Not^
```
### Clustering

### Subject Mappings


```
host: 0.0.0.0port:
mappings: {foo: bar
}
accounts: { accountA: {
mappings: { orders.acme.*: orders.$
users: [}
{user: admin, password: admin}, {user: user, password: user}
] },
}
Property Description Default
mappings
```
```
Congursubjects aliasing and patterns based tration map for mapping subjectanslation.. Allows for
Can be used tleafnode conguro great effect in superation and when sourcluster andcing streams. (none set)mappings {}^
```
```
Property Description Default
```
```
ping_interval
```
Duration at which pings arnodes and routes. In the pre sent to clients, leafesence of client trac, (^)
such as messages or client side pings, the serwill not send pings. Therefore it is recommendedver
to keep this value bigger than what clients use.
"2m"
ping_max After how many unanswerwill allow before closing the connection.ed pings the server^2
write_deadline Maximum number of seconds the serblock when writing. Once this threxceeded the connection will be closed. See eshold isver will (^) slow
consumer on how to deal with this on the client.
"10s"

### Connection Timeouts


You can enable JetStream in the server's conguration by simply adding a
jetstream {} map. By default, the JetStream subsystem will store data in the /tmp
directory, but you can specify the directory to use via the store_dir, as well as the
limits for JetStream storage (a value of 0 means no limit).
Normally JetStream will be run in clustered mode and will replicate data, so the best
place to store JetStream data would be locally on a fast SSD. One should specically
avoid NAS or NFS storage for JetStream.

```
Property Description Default
max_connections Maximum number of active client connections. 64K
```
```
max_control_line Maximum length of a prcombined length of subject and queue grIncreasing this value maotocol line (includingy require client oup).^
changes to be used. Applies to all trac.
```
```
4KB
```
```
max_payload
```
```
Maximum number of bReducing this size may force you to implement ytes in a message payload.
chunkingpayloads. It is not r in your clients. Applies tecommended to client and leafnodeo use values over
8MB but max payload must be equal tmax_payload can be set up to or smaller than the o 64MB. The
max_pending value.
```
```
1MB
```
```
max_pending
```
Maximum number of bconnection. Applies to client connections. Note thatytes buffered for a (^)
applications can also set 'Pof messages and total size) for their subscriptions.endingLimits' (number 64MB
max_subscriptions Maximum numbers of subscriptions perclient and leafnode accounts connection. (^0) , unlimited

### Limits

### JetStream


Note that each JetStrJetstream replicates data between cluster nodes (up team enabled server MUST use its own individual sto 5 replicas), achieorage directory.ving redundancy (^)
and availability through this.
Jetstream does not implement standbstandby server shares a storage dir with an activy and fault te server, you must makolerance through a share sure only one ised le system. If a
active at any time. Access conicts are not detected. We do not recommend such a setup.
Here's an example minimal le that will store data in a local "nats" directory with some
limits.
$ nats-server -c js.conf
jetstream { store_dir: nats
# 1GB max_memory_store: 1073741824
# 10GB max_file_store: 10737418240
}
**Property Description Default Version**
store_dir Directory to use forJetStream storage.^ /tmp/nats/jetstream 2.2.
max_memory_store Maximum sizthe 'memory' storagee of^ 75% ofavailable^
memory 2.2.
max_file_store Maximum siz'le ' storage. For unitse of the^
use m mb g gb t tb 1TB 2.2.
cipher Set to enable stencryption at reither chachapolyestorage-level. Choose or aes^. (not set) 2.3.
key The encrencryption is enabled. A kyption key to use wheney (not set) 2.3.


A default nats server will have no authentication or authorization enabled. This is useful
for development and simple embedded use cases only. The default account is $G.
Once at least one user is congured in the authorization or accounts sections the default
$G account an no-authentication user are disabled. You can restore no authentication
access by setting the no_auth_user.

```
Property Description length of at least 32 bytes is Default Version
recommended. Note, this kHMAC-256 hashed on startupey is
which reduces the b64. yte length to
```
max_outstanding_catchup Max in-ight bfor stream catch-upytes (^) 32MB 2.9.
sync_interval
Examples: Change the default fsync/sync10s 1m always -
interval for page cache in thelest ore. By default JetStream (^)
relies on strcluster to guarantee data iseam replication in the
available after an OS crrun JetStream without rash. If youeplication (^)
or with a rmay want to shorten theeplication of just 2 y ou
fsync/sync interforce an fsync after eachval. - You can
messsage with slow down the thralwaysoughput t, this willo a
few hundred msg/s.
2m 2.10.

### Authentication and Authorization

**Centralized Authentication and Authorization**


The Conguration options here refer to JWT based authentication and authorization.

```
Property Description Default
authorization
```
Congurauthentication/authorizationation map for client (^). List of user
and their auth setting. when only the default account ($G) is activThis section is used (^) e. (not set)authorization {}^
accounts
Conguraccountsation map for multi tenancy via. A list of accounts each with its (^)
own users and their auth settings. Eachacount forms its own subject and stream (^)
namespace, with not data sharexplicit import and exported unless is congur (^) ed.
(not set)accounts {}^
no_auth_user
Usernameor an account present in the. A client connecting withoutauthorization block (^)
any form of authentication will be associatedwith this user, its permissions and account.
(not set) - will denyunauthorized access (^)
by default if any otherusers are congured.
**Property Description**
operator The Json Web Token of the auth operator.
resolver
The built-in Nto use an external account serATS resolver, ver. (When the operMEMORY for static or ator JWTURL(<url>)
contains an account URL, it will be used as default. In thiscase resolver is only needed to overwrite the default.)
resolver_tls (This is for an outgoing connection and thernot use tls congurtimeoutation map, verify for tls connections t and map_and_verifyefore doeso the resolver.)^
resolver_preload Mapconsist of to preload account public k<account public nkey>eys and their corr, value is the esponding JW<corresponding jwt>T. Keys^.
**Decentralized Authentication and Authorization**

### Runtime Configuration


```
Property Description Default
disable_sublist_cache If accounts. true disable subscription caches for allThis is saves resources in situations^
where different subjects are used all the time.
cache enabledfalse,^
```
```
lame_duck_duration
```
```
In lame duck mode the serand slowly closes client connections. After thisver rejects new clients
duration is ocannot be set lower than 30 seconds. Starver the server shuts down. This valuet lame
duck mode with: nats-server --signal ldm.
```
```
"2m"
```
```
lame_duck_grace_period This is the durafter entering lame duck mode, beforation the server waits,^ e
starting to close client connections "10s"
```
```
Property Description Default
server_name The servers name, shows up in logging. Defaultsto the sera domain, all server's id. When JetStrver names need team is used, withino be unique.^ GeneratedServer ID^
```
```
server_tags
```
A set of tags describing prserver. This will be exposed throperties of theough /varz (^)
and can be used for system rrequests, such as placement of stresourceeams. It is (^)
recommended to use key:value style notation.
[]
trace If messages. Extrue enable prcludes the system account.otocol trace log^ disabledfalse,^
trace_verbose If messages. Includes the system account.true enable protocol trace log^ disabledfalse,^
debug If true enable debug log messages disabledfalse,^
logtime If set to false, log without timestamps timestamptrue, include^

### Cluster Configuration, Monitoring and Tracing


**Property Description Default**
log_file Log le name, relative to... No log le
log_size_limit Size in bytes after the logle r olls over to a new one^0 , unlimited

max_traced_msg_len Set a limit tthe payload of a message.o the trace of (^0) , unlimited
syslog Log to syslog. disabledfalse,^
remote_syslog Syslog server address. (not set)
http_port http port for server monitoring. (inactive)
http Listen specication for server monitoring.<host>:<port> (inactive)
https_port https porThis is inuenced bt for server monity the tls proring.operty.^ (inactive)
http_base_path base path for monitoring endpoints. /
https Listen specication for TLS server monitthe tls section to be present.oring. Requir<host>:<port>es^ (inactive)
system_account Name of the system account. Users of thisaccount can subscribe tSee System Accounts for moro system ee details.vents.^ $SYS
pid_file File containing PIDcan serve as input t, relative to ... Thiso nats-server --signal^ (non set)
port_file_dir Directory to write a le containing theservers open ports to, relative to ...^ (not set)
connect_error_reports Number of attempts at which a rroute, gateway or leaf node connection is repeated failedeported.^
Connect attempts are made once every second.
every hour^3600 , approx^


**Property Description Default**
reconnect_error_reports Number of failed attempts ta route, gateway or leaf node connection.o reconnect^
Default is to report every attempt.
failed attempt^1 , every^


# Configuring JetStream

To enable JetStream in a server we have to congure it at the top level rst:

You can also use the -js, --jetstream and -sd, --store_dir <dir> ags fr om the
command line

JetStream is compatible with NATS 2.0 Multi-Tenancy using Accounts. A JetStream
enabled server supports creating fully isolated JetStream environments for different
accounts.
This will dynamically determine the available resources. It's recommended that you set
specic limits though:

At this point JetStream will be enabled and if you have a server that does not have
accounts enabled, all users in the server would have access to JetStream

```
jetstream: enabled
```
```
jetstream { store_dir: /data/jetstream
max_mem: 1G max_file: 100G
}
```
## Configuring JetStream

## Enabling JetStream for a nats-server

## Multi-tenancy & Resource Mgmt

## Setting account resource limits


Here the HR account would have access to all the resources congured on the server,
we can restrict it:

Now the HR account is limited in various dimensions.
If you try to congure JetStream for an account without enabling it globally you'll get a
warning and the account designated as System cannot have JetStream enabled.

If your setup is in operator mode, JetStream specic account conguration can be stored
in account JWT. The earlier account named HR can be congured as follows:

```
jetstream { store_dir: /data/jetstream
max_mem: 1G max_file: 100G
}
accounts { HR: {
jetstream: enabled }
}
```
```
jetstream { store_dir: /data/jetstream
max_mem: 1G max_file: 100G
}
accounts { HR: {
jetstream { max_mem: 512M
max_file: 1G max_streams: 10
max_consumers: 100 }
}}
```
**Operator mode account resources limits using the nsc CLI tool**


nscnsc addedit account account --name --name HR HR --js-mem-storage 1 G --js-disk-storage 512 M --


# Configuration Management

In many cases managing the conguration in your application code is the best model,
many teams though wish to pre-create Streams and Consumers.
We support a number of tools to assist with this:
CLI with Conguration Files
Terraform
GitHub Actions
Kubernetes JetStream Controller


# NATS Admin CLI

The nats CLI can be used to manage Streams and Consumers easily using it's
--config ag, for example:

This creates a new Stream based on orders.json. The orders.json le can be
extracted from an existing stream using nats stream info ORDERS -j | jq .config

This edits an existing stream ensuring it complies with the conguration in
orders.json

This creates a new Consumer based on orders_new.json. The orders_new.json le
can be extracted from an existing stream using
nats con info ORDERS NEW -j | jq .config

```
nats str add ORDERS --config orders.json
```
```
nats str edit ORDERS --config orders.json
```
```
nats con add ORDERS NEW --config orders_new.json
```
## nats Admin CLI

## Add a new Stream

## Edit an existing Stream

## Add a New Consumer


# Terraform

Terraform is a Cloud conguration tool from Hashicorp. We maintain a Provider for
Terraform called terraform-provider-jetstream that can maintain JetStream using
Terraform.
Find it in the Terraform registry.

In your project you can congure the Provider like this:

Sample code below that creates the ORDERS example. Review the Project README for
full details.

```
provider "jetstream" { servers = "connect.ngs.global"
credentials = "ngs_jetstream_admin.creds"}
```
## Setup


resource "jetstream_stream" "ORDERS" { name = "ORDERS"
subjects = ["ORDERS.*"] storage = "file"
max_age = 60 * 60 * 24 * 365}

resource "jetstream_consumer" "ORDERS_NEW" { stream_id = jetstream_stream.ORDERS.id
durable_name = "NEW" deliver_all = true
filter_subject = "ORDERS.received" sample_freq = 100
}
resource "jetstream_consumer" "ORDERS_DISPATCH" { stream_id = jetstream_stream.ORDERS.id
durable_name = "DISPATCH" deliver_all = true
filter_subject = "ORDERS.processed" sample_freq = 100
}
resource "jetstream_consumer" "ORDERS_MONITOR" { stream_id = jetstream_stream.ORDERS.id
durable_name = "MONITOR" deliver_last = true
ack_policy = "none" delivery_subject = "monitor.ORDERS"
}
output "ORDERS_SUBJECTS" { value = jetstream_stream.ORDERS.subjects
}


# GitHub Actions

We have a pack of GitHub Actions that let you manage an already running JetStream
Server, useful for managing releases or standing up test infrastructure.
Full details and examples are in the jetstream-gh-actions repository, here's an example.


onname: push: orders
jobs:
# First we delete the ORDERS stream and consumer if they already existclean_orders:
runs-onsteps: : ubuntu-latest

- nameuses:: orders_streamnats-io/jetstream-gh-action/delete/stream@master
withmissing_ok: : 1
streamserver:: ORDERSjs.example.net

# Now we create the Stream and Consumers using the same configuration f# nats CLI utility would use as shown above
create_ordersruns-on: ubuntu-latest:
needssteps:: clean_orders

- - usesname:: actions/checkout@masterorders_stream
useswith:: nats-io/jetstream-gh-action/create/stream@master
configserver:: ORDERS.jsonjs.example.net
- nameuses:: orders_new_consumernats-io/jetstream-gh-action/create/consumer@master
withconfig: : ORDERS_NEW.json
streamserver:: ORDERSjs.example.net

# We publish a message to a specific Subject, perhaps some consumer is # waiting there for it to kick off tests
publish_messageruns-on: ubuntu-latest:
needssteps:: create_orders

- - usesname:: actions/checkout@masterorders_new_consumer
useswith:: nats-io/jetstream-gh-action@master
subjectmessage:: ORDERS.deploymentPublished new deployment via "${{ github.event_name }}
server: js.example.net


# Kubernetes Controller

The JetStream controllers allow you to manage NATS JetStream Streams and
Consumers via K8S CRDs. You can nd more info on how to deploy and usage here.
Below you can nd an example of how to create a stream and a couple of consumers:

Once the CRDs are installed you can use kubectl to manage the streams and
consumers as follows:

```
---apiVersion: jetstream.nats.io/v1beta1
kindmetadata: Stream:
```
(^) specname: : mystream
namesubjects: mystream: ["orders.*"]
storagemaxAge:: 1hmemory
---apiVersion: jetstream.nats.io/v1beta1
kindmetadata: Consumer:
(^) specname: : my-push-consumer
streamNamedurableName: (^) :mystream my-push-consumer
deliverSubjectdeliverPolicy:: lastmy-push-consumer.orders
ackPolicyreplayPolicy: none: instant
---apiVersion: jetstream.nats.io/v1beta1
kindmetadata: Consumer:
(^) specname: : my-pull-consumer
streamNamedurableName: (^) :mystream my-pull-consumer
deliverPolicyfilterSubject:: allorders.received
maxDeliverackPolicy:: explicit 20


$NAME kubectl getSTATE streams STREAM NAME SUBJECTS
mystream Created mystream [orders.*]
$NAME kubectl get consumersSTATE STREAM CONSUMER ACK POLICY
my-pull-consumermy-push-consumer CreatedCreated mystreammystream my-pull-consumermy-push-consumer explicitnone

# If you end up in an Errored state, run kubectl describe for more info.# kubectl describe streams mystream
# kubectl describe consumers my-pull-consumer


# Clustering

NATS supports running each server in clustered mode. You can cluster servers together
for high volume messaging systems and resiliency and high availability.
NATS servers achieve this by gossiping about and connecting to, all of the servers they
know, thus dynamically forming a full mesh. Once clients connect or re-connect to a
particular server, they are informed about current cluster members. Because of this
behavior, a cluster can grow, shrink and self heal. The full mesh does not necessarily have
to be explicitly congured either.
Note that NATS clustered servers have a forwarding limit of one hop. This means that
each nats-server instance will **only** forward messages that it has received **from a
client** to the immediately adjacent nats-server instances to which it has routes.
Messages received **from** a route will only be distributed to local clients.
For the cluster to successfully form a full mesh and NATS to function as intended and
described throughout the documentation - temporary errors permitting - it is necessary
that servers can connect to each other and that clients can connect to each server in the
cluster.

In addition to a port to listen for clients, nats-server listens on a "cluster" URL (the
-cluster option). Additional nats-server servers can then add that URL to their
-routes argument to join the cluster. These options can also be specied in a cong
le, but only the command-line version is shown in this overview for simplicity.

## NATS Server Clustering

## Cluster URLs

## Running a Simple Cluster


Here is a simple cluster running on the same machine:
Server A - the 'seed server'

Server B

Check the output of the server for the selected client and route ports.
Server C

Check the output of the server for the selected client and route ports.
Each server has a client and cluster port specied. Servers with the routes option
establish a route to the seed server. Because the clustering protocol gossips members of
the cluster, all servers are able to discover other server in the cluster. When a server is
discovered, the discovering server will automatically attempt to connect to it in order to
form a full mesh. Typically only one instance of the server will run per machine, so you
can reuse the client port (4222) and the cluster port (4248), and simply the route to the
host/port of the seed server.
Similarly, clients connecting to any server in the cluster will discover other servers in the
cluster. If the connection to the server is interrupted, the client will attempt to connect to
all other known servers.
There is no explicit conguration for seed server. They simply serve as the starting point
for server discovery by other members of the cluster as well as clients. As such these are
the servers that clients have in their list of connect urls and cluster members have in their
list of routes. They reduce conguration as not every server needs to be in these lists. But
the ability for other server and clients to successfully connect depends on seed server

```
nats-server -p 4222 -cluster nats://localhost:4248 --cluster_name test-cl
```
```
nats-server -p 5222 -cluster nats://localhost:5248 -routes nats://localho
```
```
nats-server -p 6222 -cluster nats://localhost:6248 -routes nats://localho
```

running. If multiple seed server are used, they make use of the routes option as well, so
they can establish routes to one another.

The following cluster options are supported:

When a NATS server routes to a specied URL, it will advertise its own cluster URL to all
other servers in the route effectively creating a routing mesh to all other servers.
**Note:** when using the -routes option, you must also specify a -cluster option.
Clustering can also be congured using the server cong le.

The following example demonstrates how to run a cluster of 3 servers on the same host.
We will start with the seed server and use the -D command line parameter to produce
debug information.

Alternatively, you could use a conguration le, let's call it seed.conf, with a content
similar to this:

```
--routes [rurl-1, rurl-2] Routes to solicit and connect--cluster nats://host:port Cluster URL for solicited routes
```
```
nats-server -p 4222 -cluster nats://localhost:4248 -D
```
## Command Line Options

## Three Server Cluster Example


And start the server like this:

This will produce an output similar to:

It is also possible to specify the hostname and port independently. At the minimum, the
port is required. If you leave the hostname off it will bind to all the interfaces ('0.0.0.0').

Now let's start two more servers, each one connecting to the seed server.

When running on the same host, we need to pick different ports for the client connections
-p, and for the port used to accept other routes -cluster. Note that -routes points
to the -cluster address of the seed server (localhost:4248).

```
# Cluster Seed Node
listen: 127.0.0.1:4222http: 8222
cluster { listen: 127.0.0.1:4248
}
```
```
nats-server -config ./seed.conf -D
```
```
[83329] 2020/02/12 16:04:52.369039 [INF] Starting nats-server version 2.1[83329] 2020/02/12 16:04:52.369130 [DBG] Go build version go1.13.6
[83329] 2020/02/12 16:04:52.369133 [INF] Git commit [not set][83329] 2020/02/12 16:04:52.369360 [INF] Starting http monitor on 127.0.0
[83329] 2020/02/12 16:04:52.369436 [INF] Listening for client connections [83329] 2020/02/12 16:04:52.369441 [INF] Server id is NDSGCS74MG5ZUMBOVWO
[83329] 2020/02/12 16:04:52.369443 [INF] Server is ready[83329] 2020/02/12 16:04:52.369534 [INF] Listening for route connections
```
```
cluster { host: 127.0.0.1
port: 4248}
```
```
nats-server -p 5222 -cluster nats://localhost:5248 -routes nats://localho
```

Here is the log produced. See how it connects and registers a route to the seed server (
...GzM).

From the seed's server log, we see that the route is indeed accepted:

Finally, let's start the third server:

Again, notice that we use a different client port and cluster address, but still point to the
same seed server at the address nats://localhost:4248:

```
[83330] 2020/02/12 16:05:09.661047 [INF] Starting nats-server version 2.1[83330] 2020/02/12 16:05:09.661123 [DBG] Go build version go1.13.6
[83330] 2020/02/12 16:05:09.661125 [INF] Git commit [not set][83330] 2020/02/12 16:05:09.661341 [INF] Listening for client connections
[83330] 2020/02/12 16:05:09.661347 [INF] Server id is NAABC2CKRVPZBIECMLZ[83330] 2020/02/12 16:05:09.661349 [INF] Server is ready
[83330] 2020/02/12 16:05:09.662429 [INF] Listening for route connections [83330] 2020/02/12 16:05:09.662676 [DBG] Trying to connect to route on lo
[83330] 2020/02/12 16:05:09.663308 [DBG] 127.0.0.1:4248 - rid:1 - Route c[83330] 2020/02/12 16:05:09.663370 [INF] 127.0.0.1:4248 - rid:1 - Route c
[83330] 2020/02/12 16:05:09.663537 [DBG] 127.0.0.1:4248 - rid:1 - Registe[83330] 2020/02/12 16:05:09.663549 [DBG] 127.0.0.1:4248 - rid:1 - Sent lo
```
```
[83329] 2020/02/12 16:05:09.663386 [INF] 127.0.0.1:62941 - rid:1 - Route [83329] 2020/02/12 16:05:09.663665 [DBG] 127.0.0.1:62941 - rid:1 - Regist
[83329] 2020/02/12 16:05:09.663681 [DBG] 127.0.0.1:62941 - rid:1 - Sent l
```
```
nats-server -p 6222 -cluster nats://localhost:6248 -routes nats://localho
```

First a route is created to the seed server (...IOW) and after that, a route from ...3PK

- which is the ID of the second server - is accepted.
The log from the seed server shows that it accepted the route from the third server:

And the log from the second server shows that it connected to the third.

At this point, there is a full mesh cluster of NATS servers.

Now, the following should work: make a subscription to the rst server (port 4222). Then
publish to each server (ports 4222, 5222, 6222). You should be able to receive messages
without problems.

```
[83331] 2020/02/12 16:05:12.838022 [INF] Listening for client connections [83331] 2020/02/12 16:05:12.838029 [INF] Server id is NBE7SLUDLFIMHS2U634
[83331] 2020/02/12 16:05:12.838031 [INF] Server is ready...
[83331] 2020/02/12 16:05:12.839203 [INF] Listening for route connections [83331] 2020/02/12 16:05:12.839453 [DBG] Trying to connect to route on lo
[83331] 2020/02/12 16:05:12.840112 [DBG] 127.0.0.1:4248 - rid:1 - Route c[83331] 2020/02/12 16:05:12.840198 [INF] 127.0.0.1:4248 - rid:1 - Route c
[83331] 2020/02/12 16:05:12.840324 [DBG] 127.0.0.1:4248 - rid:1 - Registe[83331] 2020/02/12 16:05:12.840342 [DBG] 127.0.0.1:4248 - rid:1 - Sent lo
[83331] 2020/02/12 16:05:12.840717 [INF] 127.0.0.1:62946 - rid:2 - Route [83331] 2020/02/12 16:05:12.840906 [DBG] 127.0.0.1:62946 - rid:2 - Regist
[83331] 2020/02/12 16:05:12.840915 [DBG] 127.0.0.1:62946 - rid:2 - Sent l
```
```
[83329] 2020/02/12 16:05:12.840111 [INF] 127.0.0.1:62945 - rid:2 - Route [83329] 2020/02/12 16:05:12.840350 [DBG] 127.0.0.1:62945 - rid:2 - Regist
[83329] 2020/02/12 16:05:12.840363 [DBG] 127.0.0.1:62945 - rid:2 - Sent l
```
```
[83330] 2020/02/12 16:05:12.840529 [DBG] Trying to connect to route on 12[83330] 2020/02/12 16:05:12.840684 [DBG] 127.0.0.1:6248 - rid:2 - Route c
[83330] 2020/02/12 16:05:12.840695 [INF] 127.0.0.1:6248 - rid:2 - Route c[83330] 2020/02/12 16:05:12.840814 [DBG] 127.0.0.1:6248 - rid:2 - Registe
[83330] 2020/02/12 16:05:12.840827 [DBG] 127.0.0.1:6248 - rid:2 - Sent lo
```
### Testing the Cluster


Testing server A

Testing server B

Testing server C

Testing using seed (i.e. A, B and C) server URLs

```
natsnats subpub -s-s "nats://127.0.0.1:4222""nats://127.0.0.1:4222" hellohello &world_4222
```
```
23:34:45 Subscribing on hello23:34:45 Published 10 bytes to "hello"
[#1] Received on "hello"world_4222
```
```
nats pub -s "nats://127.0.0.1:5222" hello world_5222
[#2] Received on "hello"23:36:09 Published 10 bytes to "hello"
world_5222
```
```
nats pub -s "nats://127.0.0.1:6222" hello world_6222
23:38:40 Published 10 bytes to "hello"[#3] Received on "hello"
world_6222
```
```
nats pub -s "nats://127.0.0.1:4222,nats://127.0.0.1:5222,nats://127.0.0.1
[#4] Received on "hello"23:39:16 Published 11 bytes to "hello"
whole_world
```

# Clustering Configuration

The cluster conguration map has the following conguration options:
**Property Description**
host Interface where the gateway will listen for incoming route connections.
port Port where the gateway will listen for incoming route connections.
name Name of the cluster (recommended for NATS +v2.2)
listen Combines host and port as <host>:<port>.
tls A connection. is used for client and sertls congurverifyation map is always enabled and ver. for securing the clusteringSee for certicate pitfalls.cert_file^

```
advertisecluster_advert or
ise
```
```
Hostporby other cluster members. t <host>:<port>This is useful in setups with N to advertise how this server can be contactedAT. When
using TLS this is imporwill use when discovering the rtant to set to control the hostname that clientsoute since by default this will be an IP,
otherwise TLS hostname verication may fail with an IP SANs error.
no_advertise When set to 'true', the server will not gossip its server URLs to clients.
routes A list of other serignored. Should authentication via /password be required, specify them as parvers (URLs) to cluster with. Self-rtoken or t of the URL.usernameoutes are^
```
```
connect_retries After how many failed connect attempts testablishing a connection t, do not retry. When enabled, attempts will be made once ao a discovered route. Default is o give up^0
second. This, does not apply to explicitly congured routes.
authorization Authorizationusername/ map for conguring cluster rpassword is used, it denes the authentication mechanismoutes. When a single
```
this server expects, and how this serestablishing a connection to a discoveredver will authenticate itself when route. This will not be used for (^)
routes explicitly listed in part of the URL. With this authentication mode, either use the sameroutes and therefore have to be provided as
credentials throughout the system or list every route explicitly on every


**Property Description**

server. If the provide the expected tls congurusernameation map species. Here different certicates can be used,verify_and_map only (^)
but they have to map to the same allows for timeout which is honorusernameed but users. The authorization map also and token
congurThe permissionsation are not suppor block is ignorted and will pred. event the server from starting.
pool_size The size of the connection pool used tpinned accounts. Default is 3. Refer to o distribute load acrv2 Routes for details.oss non-
accounts An optional list of accounts that will haroute connection. Refer to v2 Routes for details.ve a pinned^
compression The comprpeer servers. Default is ession mode the seracceptver will use when connecting with. Refer to v2 Routes for details.^
cluster { name: example
# host/port for inbound route connections from other server listen: localhost:4244
# Authorization for route connections # Other server can connect if they supply the credentials listed here
# This server will connect to discovered routes using this user authorization {
user: route_user password: pwd
timeout: 0.5 }
# This server establishes routes with these server. # This server solicits new routes and Routes are actively solicited and
# Other servers can connect to us if they supply the correct credential # in their routes definitions from above.
routes = [ nats://route_user:pwd@127.0.0.1:4245
nats://route_user:pwd@127.0.0.1:4246 ]
}


# v2 Routes

Introduced in NATS v2.10.0

Before the v2.10.0 release, two servers in a cluster had only one connection to transmit
all messages for all accounts, which could lead to slow down and increased memory
usage when data was not transmitted fast enough.
The v2.10.0 release introduces the ability to have multiple route connections between two
servers. By default, without any conguration change, clustering two v2.10.0+ servers
together will create 3 route connections. This can of course be congured to a different
value by explicitly conguring the pool size.

Each of those connections will handle a specic subset of the accounts, and the
assignment of an account to a specic connection index in the pool is the same in any
server in the cluster. It is required that each server in the cluster have the same pool size
value, otherwise, clustering will fail to be established with an error similar to:

In the event that a given connection of the pool breaks, automatic reconnection occurs as
usual. However, while the disconnection is happening, trac for accounts handled by that

```
cluster { pool_size: 3
}
```
```
[ERR] 127.0.0.1:6222 - rid:6 - Mismatch route pool size: 3 vs 4
```
## Connection pooling

## Pool size requirements

## Handling loss of connection


connection is stopped (that is, trac is not routed through other connections), the same
way that it was when there was a single route connection.

Note that although the cluster's pool_size conguration parameter can be changed
through a conguration reload, connections between servers will likely break since there
will be a mismatch between servers. It is possible though to do a rolling reload by setting
the same value on all servers. Client and other connections (including dedicated account
routes - see next section) will not be closed.

When monitoring is enabled, you will see that the /routez page now has more
connections than before, which is expected.

It is possible to disable route connection pooling by setting the pool_size conguration
parameter to the value -1. When that is the case, a server will behave as a server pre-
v2.10.0 and create a single route connection between its peers.
Note that in that mode, no accounts list can be dened (see "Accounts Pinning" below).

In addition to connection pooling, the release v2.10.0 has introduced the ability to
congure a list of accounts that will have a dedicated route connection.

```
cluster { accounts: [acc1, acc2]
}
```
### Configuration reload

### Monitoring

### Disabling connection pooling

## Account pinning


Note that by default, the server will create a dedicated route for the system account (no
specic conguration is needed).
Having a dedicated route improves performance and reduces latency, but another benet
is that since the route is dedicated to an account, the account name does not need to be
added to the message route protocols. Since account names can be quite long, this
reduces the number of bytes that need to be transmitted in all other cases.

In the event that an account route connection breaks, automatic reconnection occurs as
usual. However, while the disconnection is happening, trac for this account is stopped.

The accounts list can be modied and a conguration signal be sent to the server. All
servers need to have the same list, however, it is possible to perform a rolling
conguration reload.
For instance, adding an account to the list of server A and issuing a conguration
reload will not produce an error, even though the other server in the cluster does not have
that account in the list yet. A dedicated connection will not yet be established, but trac
for this account in the pooled connection currently handling it will stop. When the
conguration reload happens on the other server, a dedicated connection will then be
established and this account’s trac will resume.
When removing an account from the list and issuing a conguration reload, the
connection for this account will be closed, and trac for this account will stop. Other
server(s) that still have this account congured with a dedicated connection will fail to
reconnect. When they are also sent the conguration reload (with updated accounts
conguration), the account trac will now be handled by a connection in the pool.
Note that conguration reload of changes in the accounts list do not affect existing
pool connections, and therefore should not affect trac for other accounts.

### Handling loss of connection

### Configuration reload


When monitoring is enabled, the route connection's information now has a new eld
called account that displays the name of the account this route is for.

As indicated in the connection pooling section, if pool_size is set to -1, the
accounts list cannot be congured nor would be in use.

Although it is recommended that all servers part of the same cluster be at the same
version number, a v2.10.0 server will be able to connect to an older server and in that
case create a single route to that server. This allows for an easy deployment of a v2.10.0
release into an existing cluster running an older release.

Release v2.10.0 introduces the ability to congure compression between servers having
route connections. The current compression algorithm used is S2, an extension of
Snappy. By default, routes do not use compression and it needs to be explicitly enabled.

```
cluster { compression: {
mode: accept }
}
```
### Monitoring

### Disabling connection pooling

### Connecting to an older server

## Compression

### Compression Modes


There are several modes of compression and there is no requirement to have the same
mode between routed servers.

When s2_auto compression is used, it relies on a rtt_thresholds option, which is a
list of three latency thresholds that dictate increasing or decreasing the compression
mode.

The default rtt_thresholds value is [10ms, 50ms, 100ms]. The way to read this is
that if the RTT is under 10ms, no compression is applied. Once 10ms is reached,
s2_fast is applied and so on with the remaining two thresholds for s2_better and
s2_best.

The compression conguration can be changed through conguration reload. If the
value is changed from off to anything else, then connections are closed and recreated,

```
off - Explicitly disables compression for any route between the server and a peer.
mode of the peer it connects taccept (default) - Does not initiate compro. ession, but will accept the compression^
s2_fast - Applies compression, but optimizes for speed over compression ratio.
ratio.s2_better - Applies compression, providing a balance of speed and compression^
s2_best - Applies compression and optimizes for compression ratio over speed.
measured between the sers2_auto - Choose the apprver and the peeropriate s2_* mode relative to the round-trip time (R. See rtt_thresholds below. TT)^
```
```
cluster { compression: {
mode: s2_auto rtt_thresholds: [10ms, 50ms, 100ms]
}}
```
### Round-trip time thresholds

### Configuration reload


same as if the compression mode was set to something and disabled by setting it to
off.
For all other compression modes, the mode is changed dynamically without a need to
close the route connections.

When monitoring is enabled, the route connection's information now has a new eld
called compression that displays the current compression mode. It can be off or any
other mode described above. Note that s2_auto is not displayed, instead, what will be
displayed is the current mode, say s2_best or s2_uncompressed.
If connected to an older server, the compression eld will display not supported.

It is possible to have a v2.10.0+ server, with a compression mode congured, connect to
an older server that does not support compression. The connection will simply not use
compression.

### Monitoring

### Connecting to an older server


# JetStream Clustering

Clustering in JetStream is required for a highly available and scalable system. Behind
clustering is RAFT. There's no need to understand RAFT in depth to use clustering, but
knowing a little explains some of the requirements behind setting up JetStream clusters.

JetStream uses a NATS optimized RAFT algorithm for clustering. Typically RAFT
generates a lot of trac, but the NATS server optimizes this by combining the data plane
for replicating messages with the messages RAFT would normally use to ensure
consensus. Each server participating requires an unique server_name (only applies
within the same domain).

The RAFT groups include API handlers, streams, consumers, and an internal algorithm
designates which servers handle which streams and consumers.
The RAFT algorithm has a few requirements:

In order to ensure data consistency across complete restarts, a quorum of servers is
required. A quorum is ½ cluster size + 1. This is the minimum number of nodes to ensure
at least one node has the most recent data and state after a catastrophic failure. So for a
cluster size of 3, you’ll need at least two JetStream enabled NATS servers available to
store new messages. For a cluster size of 5, you’ll need at least 3 NATS servers, and so
forth.

```
A log to persist state
A quorum for consensus
```
## RAFT

## RAFT Groups

## The Quorum


**Meta Group** - all servers join the Meta Group and the JetStream API is managed by this
group. A leader is elected and this owns the API and takes care of server placement.

**Stream Group** - each Stream creates a RAFT group, this group synchronizes state and
data between its members. The elected leader handles ACKs and so forth, if there is no
leader the stream will not accept messages.

```
Meta Group
```
### RAFT Groups


**Consumer Group** - each Consumer creates a RAFT group, this group synchronizes
consumer state between its members. The group will live on the machines where the
Stream Group is and handle consumption ACKs etc. Each Consumer will have their own
group.

```
Stream Groups
```
```
Consumer Groups
```

Generally, we recommend 3 or 5 JetStream enabled servers in a NATS cluster. This
balances scalability with a tolerance for failure. For example, if 5 servers are JetStream
enabled You would want two servers is one “zone”, two servers in another, and the
remaining server in a third. This means you can lose any one “zone” at any time and
continue operating.

This is possible and even recommended in some cases. By mixing server types you can
dedicate certain machines optimized for storage for Jetstream and others optimized
solely for compute for standard NATS servers, reducing operational expense. With the
right conguration, the standard servers would handle non-persistent NATS trac and the
JetStream enabled servers would handle JetStream trac.

To congure JetStream clusters, just congure clusters as you normally would by
specifying a cluster block in the conguration. Any JetStream enabled servers in the list
of clusters will automatically chatter and set themselves up. Unlike core NATS clustering
though, each JetStream node **must specify** a server name and cluster name.
Below are explicitly listed server conguration for a three-node cluster across three
machines, n1-c1, n2-c1, and n3-c1.

A user and password under the system account ($SYS) should be congured. The
following conguration uses a bcrypted password: a very long s3cr3t! password.

### Cluster Size

### Mixing JetStream enabled servers with standard NATS

### servers

## Configuration

### Server password configuration


```
server_name=n1-c1listen=4222
accounts { $SYS {
users = [ { user: "admin",
pass: "$2a$11$DRh4C0KNbNnD8K/hb/buWe1zPxEHrLEiDmuq1Mi0rRJiH/W25Qi }
] }
}
jetstream { store_dir=/nats/storage
}
cluster { name: C1
listen: 0.0.0.0:6222 routes: [
nats://host_b:6222 nats://host_c:6222
]}
```
### Server 1 (host_a)

### Server 2 (host_b)


```
server_name=n2-c1listen=4222
accounts { $SYS {
users = [ { user: "admin",
pass: "$2a$11$DRh4C0KNbNnD8K/hb/buWe1zPxEHrLEiDmuq1Mi0rRJiH/W25Qi }
] }
}
jetstream { store_dir=/nats/storage
}
cluster { name: C1
listen: 0.0.0.0:6222 routes: [
nats://host_a:6222 nats://host_c:6222
]}
```
### Server 3 (host_c)


Add nodes as necessary. Choose a data directory that makes sense for your environment,
ideally a fast SSD, and launch each server. After two servers are running you'll be ready to
use JetStream.

```
server_name=n3-c1listen=4222
accounts { $SYS {
users = [ { user: "admin",
pass: "$2a$11$DRh4C0KNbNnD8K/hb/buWe1zPxEHrLEiDmuq1Mi0rRJiH/W25Qi }
] }
}
jetstream { store_dir=/nats/storage
}
cluster { name: C1
listen: 0.0.0.0:6222 routes: [
nats://host_a:6222 nats://host_b:6222
]}
```

# Administration

Once a JetStream cluster is operating interactions with the CLI and with nats CLI is the
same as before. For these examples, lets assume we have a 5 server cluster, n1-n5 in a
cluster named C1.

Within an account there are operations and reports that show where users data is placed
and which allow them some basic interactions with the RAFT system.

When adding a stream using the nats CLI the number of replicas will be asked, when
you choose a number more than 1, (we suggest 1, 3 or 5), the data will be stored on
multiple nodes in your cluster using the RAFT protocol as above.

Example output extract:

```
nats str add ORDERS --replicas 3
```
```
....Information for Stream ORDERS created 2021-02-05T12:07:34+01:00
....Configuration:
.... Replicas: 3
Cluster Information:
Name: C1 Leader: n1-c1
Replica: n4-c1, current, seen 0.07s ago Replica: n3-c1, current, seen 0.07s ago
```
## Account Level

## Creating clustered streams


Above you can see that the cluster information will be reported in all cases where Stream
info is shown such as after add or using nats stream info.
Here we have a stream in the NATS cluster C1, its current leader is a node n1-c1 and it
has 2 followers - n4-c1 and n3-c1.
The current indicates that followers are up to date and have all the messages, here
both cluster peers were seen very recently.
The replica count cannot be edited once congured.

Users can get overall statistics about their streams and also where these streams are
placed:

Every RAFT group has a leader that's elected by the group when needed. Generally there
is no reason to interfere with this process, but you might want to trigger a leader change
at a convenient time. Leader elections will represent short interruptions to the stream so
if you know you will work on a node later it might be worth moving leadership away from
it ahead of time.

```
nats stream report
Obtaining Stream stats+----------+-----------+----------+--------+---------+------+---------+--
| Stream | Consumers | Messages | Bytes | Storage | Lost | Deleted | C+----------+-----------+----------+--------+---------+------+---------+--
| ORDERS | 4 | 0 | 0 B | File | 0 | 0 | n| ORDERS_3 | 4 | 0 | 0 B | File | 0 | 0 | n
| ORDERS_4 | 4 | 0 | 0 B | File | 0 | 0 | n| ORDERS_5 | 4 | 0 | 0 B | File | 0 | 0 | n
| ORDERS_2 | 4 | 1,385 | 13 MiB | File | 0 | 1 | n| ORDERS_0 | 4 | 1,561 | 14 MiB | File | 0 | 0 | n
+----------+-----------+----------+--------+---------+------+---------+--
```
### Viewing Stream Placement and Stats

**Forcing Stream and Consumer leader election**


Moving leadership away from a node does not remove it from the cluster and does not
prevent it from becoming a leader again, this is merely a triggered leader election.

The same is true for consumers, nats consumer cluster step-down ORDERS NEW.

Systems users can view state of the Meta Group - but not individual Stream or
Consumers.

We have a high level report of cluster state:

```
nats stream cluster step-down ORDERS
14:32:17 Requesting leader step down of "n1-c1" in a 3 peer RAFT group14:32:18 New leader elected "n4-c1"
Information for Stream ORDERS created 2021-02-05T12:07:34+01:00...
Cluster Information:
Name: c1 Leader: n4-c1
Replica: n1-c1, current, seen 0.12s ago Replica: n3-c1, current, seen 0.12s ago
```
```
nats server report jetstream --user admin --password s3cr3t!
```
## System Level

### Viewing the cluster state


This is a full cluster wide report, the report can be limited to a specic account using
--account.
Here we see the distribution of streams, messages, api calls etc by across 2 super
clusters and an overview of the RAFT meta group.
In the Meta Group report the server n2-c1 is not current and has not been seen for 9
seconds, it's also behind by 2 raft operations.
This report is built using raw data that can be obtained from the monitor port on the
/jsz url, or over nats using:

```
+------------------------------------------------------------------------| JetStream Summary
+--------+---------+---------+-----------+----------+--------+--------+--| Server | Cluster | Streams | Consumers | Messages | Bytes | Memory | F
+--------+---------+---------+-----------+----------+--------+--------+--| n3-c2 | c2 | 0 | 0 | 0 | 0 B | 0 B | 0
| n3-c1 | c1 | 6 | 24 | 2,946 | 27 MiB | 0 B | 2| n2-c2 | c2 | 0 | 0 | 0 | 0 B | 0 B | 0
| n1-c2 | c2 | 0 | 0 | 0 | 0 B | 0 B | 0 | n2-c1 | c1 | 6 | 24 | 2,946 | 27 MiB | 0 B | 2
| n1-c1* | c1 | 6 | 24 | 2,946 | 27 MiB | 0 B | 2+--------+---------+---------+-----------+----------+--------+--------+--
| | | 18 | 72 | 8,838 | 80 MiB | 0 B | 8+--------+---------+---------+-----------+----------+--------+--------+--
+---------------------------------------------------+| RAFT Meta Group Information |
+-------+--------+---------+---------+--------+-----+| Name | Leader | Current | Offline | Active | Lag |
+-------+--------+---------+---------+--------+-----+| n1-c1 | yes | true | false | 0.00s | 0 |
| n1-c2 | | true | false | 0.05s | 0 || n2-c1 | | true | false | 0.05s | 0 |
| n2-c2 | | true | false | 0.05s | 0 || n3-c1 | | true | false | 0.05s | 0 |
| n3-c2 | | true | false | 0.05s | 0 |+-------+--------+---------+---------+--------+-----+
```
```
nats server req jetstream --user admin --password s3cr3t! --help
```

This will produce a wealth of raw information about the current state of your cluster - here
requesting it from the leader only.

```
usage: nats server request jetstream [<flags>] [<wait>]
Show JetStream details
Flags: -h, --help Show context-sensitive help (also try --h
--version Show application version. -s, --server=NATS_URL NATS server urls
--user=NATS_USER Username or Token --password=NATS_PASSWORD Password
--creds=NATS_CREDS User credentials --nkey=NATS_NKEY User NKEY
--tlscert=NATS_CERT TLS public certificate --tlskey=NATS_KEY TLS private key
--tlsca=NATS_CA TLS certificate authority chain --timeout=NATS_TIMEOUT Time to wait on responses from NATS
--js-api-prefix=PREFIX Subject prefix for access to JetStream AP --js-event-prefix=PREFIX Subject prefix for access to JetStream Ad
--js-domain=DOMAIN JetStream domain to access --context=CONTEXT Configuration context
--trace Trace API interactions --limit=2048 Limit the responses to a certain amount o
--offset=0 Start at a certain record --name=NAME Limit to servers matching a server name
--host=HOST Limit to servers matching a server host n --cluster=CLUSTER Limit to servers matching a cluster name
--tags=TAGS ... Limit to servers with these configured ta --account=ACCOUNT Show statistics scoped to a specific acco
--accounts Include details about accounts --streams Include details about Streams
--consumer Include details about Consumers --config Include details about configuration
--leader Request a response from the Meta-group le --all Include accounts, streams, consumers and
Args: [<wait>] Wait for a certain number of responses
```
```
nats server req jetstream --user admin --password s3cr3t! --leader
```
**Forcing Meta Group leader election**


Similar to Streams and Consumers above the Meta Group allows leader stand down. The
Meta Group is cluster wide and spans all accounts, therefore to manage the meta group
you have to use a SYSTEM user.

Generally when shutting down NATS, including using Lame Duck Mode, the cluster will
notice this and continue to function. A 5 node cluster can withstand 2 nodes being down.
There might be a case though where you know a machine will never return, and you want
to signal to JetStream that the machine will not return. This will remove it from the
Stream in question and all it's Consumers.
After the node is removed the cluster will notice that the replica count is not honored
anymore and will immediately pick a new node and start replicating data to it. The new
node will be selected using the same placement rules as the existing stream.

At this point the stream and all consumers will have removed n4-c1 from the group,
they will all start new peer selection and data replication.

```
nats server raft step-down --user admin --password s3cr3t!
17:44:24 Current leader: n2-c217:44:24 New leader: n1-c2
```
```
nats stream cluster peer-remove ORDERS
? Select a Peer n4-c114:38:50 Removing peer "n4-c1"
14:38:50 Requested removal of peer "n4-c1"
```
```
$ nats stream info ORDERS
```
### Evicting a peer


We can see a new replica was picked, the stream is back to replication level of 3 and
n4-c1 is not active any more in this Stream or any of its Consumers.

```
....Cluster Information:
Name: c1 Leader: n3-c1
Replica: n1-c1, current, seen 0.02s ago Replica: n2-c1, outdated, seen 0.42s ago
```

# Troubleshooting

Diagnosing problems in NATS JetStream clusters requires:

The following tips and commands (while not an exhaustive list) can be useful when
diagnosing problems in NATS JetStream clusters:

1. Look at debug and trnats-serace logs can be turned on frver logs. By default, only warning and errom the command line using or logs are produced, but-D and -DV (^) ,
respectively. Alternatively, enabling debug or trace in the server cong.

2. Make sure that in the congured in this section: NATS JetStr{ $SYS { users } }eam conguration. , at least one system user is

```
knowledge of JetStream concepts
knowledge of the NATS Command Line Interface (CLI)
```
```
Command Description
nats account info Verify that JetStream is enabled on account
```
```
Command Description
nats server ls List known servers
nats server ping Ping all servers
nats server info Show information about a single server
nats server check Health check for NATS servers
```
## Troubleshooting tips

## nats account commands

## Basic nats server commands


**Command Description**
nats server report connections Report on connections
nats server report accounts Report on account activity
nats server report jetstream Report on JetStream activity

**Command Description**
nats server request jetstream Show JetStream details
nats server request subscriptions Show subscription information
nats server request variables Show runtime variables
nats server request connections Show connection details
nats server request routes Show route details
nats server request gateways Show gateway details
nats server request leafnodes Show leafnode details
nats server request accounts Show account details

**Command Description**
nats servercluster step-down Force a new leader election bstanding down the current meta leadery

### nats server report commands

### nats server request commands

### nats server cluster commands


```
Command Description
```
nats servercluster peer-remove (^) Removes a server from a JetStream cluster
**Command Description**
nats traffic Monitor NATS trac. ( **Experimental command** )
Testing your setup

### Experimental commands

## Further troubleshooting references


# Super-cluster with Gateways

Gateways enable connecting one or more clusters together into a full mesh; they allow
the formation of superclusters from smaller clusters. Cluster and Gateway protocols
listen on different ports. Clustering is used for adjacent servers; gateways are for joining
clusters together.
Gateway conguration is similar to clustering:

Unlike clusters, gateways:

Gateways exist to:

If gateways are to be used in a cluster, **all** servers of this cluster need to have a gateway
conguration with the **same name**. Furthermore, every gateway node needs to be able to
**connect to any** other gateway node and vice versa. Everything else is considered a
misconguration.

```
gateways have a dedicated port where they listen for gateway requests
gateways gossip gateway nodes and remote discovered gateways
```
```
have names, specifying the cluster they are in
don't form a full mesh between gatewainstead y nodes, but form a full mesh between cluster
are bound by uni-directional connections
don't gossip gateway nodes to clients
```
```
reduce the number of connections required between servers
optimize the interest graph propagation
```
## Gateways

## Gateway Connections


A nats-server in a gateway role will specify a port where it will accept gateway
connections. If the conguration species other external gateways, the gateway will
create one outbound gateway connection for each gateway in its conguration. It will
also gossip other gateways it knows or discovers. Fewer external gateways mean less
conguration. Yet, the ability to discover more gateways and gateway nodes depends on
these servers running. This is similar to seed server in cluster. It is recommended to have
all seed server of a cluster listed in the gateways section.
If the local cluster has three gateway nodes, this means there will be three outbound
connections from the local cluster to each external gateway cluster.

```
In the example above cluster A has congured gateway connections for B (solid
lines). B has discovered gateway connections to A (dotted lines). Note that the
number of outgoing connections always matches the number of gateways with the
same name.
```
```
Gateway Connections
```
##### A1

##### A2

##### A3

##### B1

##### B2


```
In this second example, again congured connections are shown with solid lines and
discovered gateway connections are shown using dotted lines. Gateways A and C
were both discovered via gossiping; B discovered A and A discovered C.
```
A key point in the description above is that each node in the cluster will make a
connection to a single node in every remote cluster — a difference from the clustering
protocol, where every node is directly connected to all other nodes.
For those mathematically inclined, cluster connections are N(N-1)/2 where N is the
number of nodes in the cluster. On gateway congurations, outbound connections are the
summation of Ni(M-1) where Ni is the number of nodes in a gateway i, and M is the
total number of gateways. Inbound connections are the summation of U-Ni where U is
the sum of all gateway nodes in all gateways, and N is the number of nodes in a gateway
i. It works out that both inbound and outbound connection counts are the same.
The number of connections required to join clusters using clustering vs. gateways is
apparent very quickly. For 3 clusters, with N nodes:

```
Gateway Discovered Gateways
```
##### A1

##### A2

##### A3

##### B1

##### B2

##### C1

```
Nodes per Cluster Full Mesh Conns Gateway Conns
1 3 6
```

A cluster section is not needed for gateways, they work with single server as well. Yet,
they start to be useful when participating cluster consist of more than one server and
they reduce the number of connections.

Messages from clients directly connected to a gateway node will be sent along outgoing
gateway connections according to the following three interest propagation mechanisms:

Local interest permitting, the receiving gateway node sends the messages directly to its
subscribing clients as well as server within the cluster.

When a publisher in A publishes "foo", the A gateway will check if cluster B has registered
no interest in "foo". If not, it forwards "foo" to B. If upon receiving "foo", B has no
subscribers on "foo", B will send a gateway protocol message to A expressing that it has
no interest on "foo", preventing future messages on "foo" from being forwarded.

```
Nodes per Cluster Full Mesh Conns Gateway Conns
2 15 12
3 36 18
4 66 24
5 105 30
30 4005 180
```
```
Optimistic Mode
Interest-only Mode
Queue Subscriptions
```
## Interest Propagation

### Optimistic Mode


Should a subscriber on B create a subscription to "foo", B knowing that it had previously
rejected interest on foo, will send a gateway protocol message to cancel its previous no
interest on "foo" in A.

When a gateway on A sends many messages on various subjects for which B has no
interest. B sends a gateway protocol message for A to stop sending optimistically, and
instead send if there's known interest in the subject. As subscriptions come and go on B,
B will update its subject interest with A.

When a queue subscriber creates a new subscription, the gateway propagates the
subscription interest to other gateways. The subscription interest is only propagated
once per Account and subject. When the last queue subscriber is gone, the cluster
interest is removed.
Queue subscriptions work on Interest-only Mode to honor NATS' queue semantics across
the Super Cluster. For each queue group, a message is only delivered to a single queue
subscriber. If the same queue group exists in multiple clusters, the server will pick one
member from the queue group in its cluster, only sending to a different cluster if there is
no interest in its cluster. In other words, a server will always try to serve local queue
subscribers rst and only failover when a local queue subscriber is not found. The server
will pick the cluster with the lowest RTT.

The Gateway Conguration document describes all the options available to gateways.

### Interest-only Mode

### Queue Subscriptions

### Gateway Configuration


# Configuration

The gateway conguration block is similar to a cluster block:

One difference is that instead of routes you specify gateways. As expected self-
gateway connections are ignored, so you can share gateway congurations with minimal
fuss.
Starting a server:

```
gateway { name: "A"
listen: "localhost:7222" authorization {
user: gwu password: gwp
}
gateways: [ {name: "A", url: "nats://gwu:gwp@localhost:7222"},
{name: "B", url: "nats://gwu:gwp@localhost:7333"}, {name: "C", url: "nats://gwu:gwp@localhost:7444"},
]}
```
```
nats-server -c A.conf
```

Once all the gateways are up, these clusters of one will forward messages as expected:

On a different session...

The subscriber should print

```
[85803] 2019/05/07 10:50:55.902474 [INF] Starting nats-server version 2.0[85803] 2019/05/07 10:50:55.903669 [INF] Gateway name is A
[85803] 2019/05/07 10:50:55.903684 [INF] Listening for gateways connectio[85803] 2019/05/07 10:50:55.903696 [INF] Address for gateway "A" is local
[85803] 2019/05/07 10:50:55.903909 [INF] Listening for client connections [85803] 2019/05/07 10:50:55.903914 [INF] Server id is NBHUDBF3TVJSWCDPG2H
[85803] 2019/05/07 10:50:55.903917 [INF] Server is ready[85803] 2019/05/07 10:50:56.830669 [INF] 127.0.0.1:50892 - gid:2 - Proces
[85803] 2019/05/07 10:50:56.830673 [INF] 127.0.0.1:50891 - gid:1 - Proces[85803] 2019/05/07 10:50:56.831079 [INF] 127.0.0.1:50892 - gid:2 - Inboun
[85803] 2019/05/07 10:50:56.831211 [INF] 127.0.0.1:50891 - gid:1 - Inboun[85803] 2019/05/07 10:50:56.906103 [INF] Connecting to explicit gateway "
[85803] 2019/05/07 10:50:56.906104 [INF] Connecting to explicit gateway "[85803] 2019/05/07 10:50:56.906404 [INF] 127.0.0.1:7333 - gid:3 - Creatin
[85803] 2019/05/07 10:50:56.906444 [INF] 127.0.0.1:7444 - gid:4 - Creatin[85803] 2019/05/07 10:50:56.906647 [INF] 127.0.0.1:7444 - gid:4 - Outboun
[85803] 2019/05/07 10:50:56.906772 [INF] 127.0.0.1:7333 - gid:3 - Outboun
```
```
nats sub -s localhost:4333 ">"
```
```
nats pub -s localhost:4444 foo bar
```
```
[#1] Received on [foo] : 'bar'
```
```
Property Description
name Name for this clusterthe same cluster, should specify the same name., all gateways belonging to^
```
reject_unknown_cluster If congurtrueed in , gatewagatewaysy will reject connections fr. It does so by checking if the cluster name,om cluster that are not (^)

## Gateway Configuration Block


**Property Description** provided by the incomming connection, exists as named gateway. This
effectively disables gossiping of new clustercongured gateway, thus cluster, from dynamically gr. It does not rowing.estrict a
gateways List of Gateway entries - see below.
host Interface where the gateway will listen for incoming gateway connections.
port Port where the gateway will listen for incoming gateway connections.
listen Combines host and port as <host>:<port>
tls A , verifycert_filetls congur is always enabled. Unless other will be the default client ceration map for securing gatewawise specied in a ticate. y connections.Certicate pitfallsgateway^.

```
advertise Hostporcontacted bNAT, or in the cloud when exposed bt <host>:<port>y other gatewa to advertise how this sery members. y a Network Load BalancerThis is useful in setups withver can be^.
```
```
connect_retries After how many failed connect attempts ta connection tretry. When enabled, attempts will be made once a second.o a discovered gateway. Default is o give up establishing 0 , do not
This, does not apply to explicitly congured gateways.
```
```
authorization
```
Authorizationused, it denes the authentication mechanism this ser map for gateways. When a single usernamever expects, and how/password is (^)
this server will authenticate itself when establishing a connection tdiscovered gateway. This will not be used for gateways explicitly listed in o a
authentication mode, either use the same crgateways and therefore have to be provided as paredentials thrt of the URL. With thisoughout the
system or list econguration map species very gateway explicitly on everify_and_mapvery server. If the only provide the expected tls
map to the same username. Here different certicates can be used, but theusername. The authorization map also allows for y do have to^
supported and will prtimeout which is honorevent the sered but usersver from starting. The and token congurpermissionsation are not
blockisignored

### Gateway Entry


The gateways conguration block is a list of gateway entries with the following
properties:

By using urls and an array, you can specify a list of endpoints that form part of a
cluster as below. A NATS Server will pick one of those addresses randomly and only
establish a single outbound gateway connection to one of the members from another
cluster:

In addition to the normal TLS conguration advice, bear in mind that TLS keys and
certicates for multiple clusters, or servers in different locations, rarely rotate at the exact
same time and that Certicate Authorities do roll between multiple Intermediate
certicates.

```
Property Description
name Gateway name.
url Hostporbe reached. If multiple IPs art <host>:<port> describing where returned, one is re the remote gatewaandomly selected.y can^
urls A list of url strings.
```
```
tls
```
A connection. If the ttls conguration mapop-level for creating a securgateway{} tls block containse gateway (^)
certicates that hato omit this one and the serve both client and server will use the cerver purposes, it is possibleticates from the
gateway{tls{}} section. See additional advice below in TLS Entry.
gateway { name: "DC-A"
listen: "localhost:7222"
gateways: [ {name: "DC-A", urls: ["nats://localhost:7222", "nats://localhost:
{name: "DC-B", urls: ["nats://localhost:7332", "nats://localhost: {name: "DC-C", urls: ["nats://localhost:7442", "nats://localhost:
]}

### TLS Entry


If using a certicate bundle which accompanied the issuance of a certicate then the CA
in that bundle will typically be for just that certicate. Using only that CA as the CA for
gateway authentication is ill-advised. You should ensure that you allow for rolling
between Certicate Authorities, even if only between multiple CAs from the same
organization entity, and use a separate certicate bundle for verication of peers. This
way when DC-B rolls before DC-A, it will not be cut off from your supercluster.

When running in a private network ( such as VPC in the cloud ), you'd probably want to
setup a Load Balancer to expose the gateway port. In this case, it's important to set the
advertise value of each NATS server to the Load Balancer hostname and port serving
them. Otherwise, they will advertise their own private IP and port, and can generate
failures and downtime if any reconnection is required.

### Gateway behind Load Balancer


# Leaf Nodes

A Leaf Node extends an existing NATS system of any size, optionally bridging both
operator and security domains. A leafnode server will transparently route messages as
needed from local clients to one or more remote NATS system(s) and vice versa. The leaf
node authenticates and authorizes clients using a local policy. Messages are allowed to
ow t o the cluster or into the leaf node based on leaf node connection permissions of
either.
Leaf nodes are useful in IoT and edge scenarios and when the local server trac should
be low RTT and local unless routed to the super cluster. NATS' queue semantics are
honored across leaf connections by serving local queue consumers rst.

Unlike cluster or gateway nodes, leaf nodes do not need to be reachable themselves and
can be used to explicitly congure any acyclic graph topologies.
If a leaf node connects to a cluster, it is recommended to congure it with knowledge of
**all** seed servers and have **each** seed server accept connections from leaf nodes. Should
the remote cluster's conguration change, the discovery protocol will gossip peers
capable of accepting leaf connections. A leaf node can have multiple remotes, each
connecting to a different cluster. Each URL in a remote needs to point to the same cluster.
If one node in a cluster is congured as leaf node, **all** nodes need to. Likewise, if one
server in a cluster accepts leaf node connections, **all** servers need to.

```
Leaf Nodes are an important component as a way to bridge trac between local
NATS servers you control and servers that are managed by a third-party. Synadia's
NGS allows accounts to use leaf nodes, but gain accessibility to the global network to
inexpensively connect geographically distributed servers or small clusters.
```
```
Clients trequired)o leaf nodes authenticate locally (or just connect if authentication is not
Trac between the leaf node and the cluster assumes the rconguration used to create the leaf connection. estrictions of the user
Subjects that the user is allowed to publish are exported to the cluster.
Subjects the user is allowed to subscribe to, are imported into the leaf node.
```

LeafNode Conguration Options

The main server is just a standard NATS server. Clients to the main cluster are just using
token authentication, but any kind of authentication can be used. The server allows leaf
node connections at port 7422 (default port):

Start the server:

Output extract

We create a replier on the server to listen for requests on 'q', which it will aptly respond
with '42':

The leaf node, allows local clients to connect to through port 4111, and doesn't require
any kind of authentication. The conguration species where the remote cluster is
located, and species how to connect to it (just a simple token in this case):

```
leafnodes { port: 7422
}authorization {
token: "s3cr3t"}
```
```
nats-server -c /tmp/server.conf
```
```
...[5774] 2019/12/09 11:11:23.064276 [INF] Listening for leafnode connection
...
```
```
nats reply -s nats://s3cr3t@localhost q 42
```
## LeafNode Configuration Tutorial


In the case where the remote leaf connection is connecting with tls:

Note the leaf node conguration lists a number of remotes. The url species the port
on the server where leaf node connections are allowed.
Start the leaf node server:

Output extract

Connect a client to the leaf server and make a request to 'q':

```
listen: "127.0.0.1:4111"leafnodes {
remotes = [ {
url: "nats://s3cr3t@localhost" },
]}
```
```
listen: "127.0.0.1:4111"leafnodes {
remotes = [ {
url: "tls://s3cr3t@localhost" },
]}
```
```
nats-server -c /tmp/leaf.conf
```
```
....[3704] 2019/12/09 09:55:31.548308 [INF] Listening for client connections
...[3704] 2019/12/09 09:55:31.549404 [INF] Connected leafnode to "localhost"
```
```
nats req -s nats://127.0.0.1:4111 q ""
```

In this example, we connect a leaf node to Synadia's NGS. Leaf nodes are supported on
free developer and paid accounts. To use NGS, ensure that you've signed up and have an
account loaded on your local system. It takes less than 30 seconds to grab yourself a
free account to follow along if you don't have one already!
The nsc tool can operate with many accounts and operators, so it's essential to make
sure you're working with the right operator and account. You can set the account using
the nsc tool like below. The DELETE_ME account is used as an example, which is
registered with NGS as a free account.

```
Published [q] : ''Received [_INBOX.Ua82OJamRdWof5FBoiKaRm.gZhJP6RU] : '42'
```
## Leaf Node Example Using a Remote Global

## Service


The nsc tool is aware of the account, so let's proceed to create a user for our example.

Let's craft a leaf node connection much like we did earlier:

```
❯❯ nscnsc envdescribe -a DELETE_ME account
+------------------------------------------------------------------------| Account Details
+---------------------------+--------------------------------------------| Name | DELETE_ME
|| AccountIssuer IDID || ABF3NX7FJLDCUO5QXBH56PV6EU4PR5HFCUJBXAG57AKODSKBNDIT3LTZWFSRAWOBXSBZ7VZCDQVU6TBJX3TQGYX
|| IssuedExpires || 2023-03-02 18 :18:42 UTC
+---------------------------+--------------------------------------------| Max Connections | 10
```
|| MaxMax LeafData Node Connections || Not1.0 AllowedGB (1000000000 (^) bytes)
|| MaxMax ExportsImports || (^27)
|| MaxMax MsgSubscriptions Payload || 1.0 10 kB (1000 bytes)
|| ExportsDisallow Allows Bearer Wildcards Token || TrueFalse (^)
|+---------------------------+-------------------------------------------- Response Permissions | Not Set
|+---------------------------+-------------------------------------------- Jetstream | Disabled
|+---------------------------+-------------------------------------------- Exports | None
nsc add user leaftestuser
[ OK ] generated and stored user key "UB5QBEU4LU7OR26JEYSG27HH265QVUFGXYV[ OK ] generated user creds file "~/.nkeys/creds/synadia/leaftest/leaftes
[ OK ] added user "leaftestuser" to account "leaftest"


The default port for leaf nodes is 7422, so we don't have to specify it.
Let's start the leaf server:

Again, let's connect a replier, but this time to Synadia's NGS. NSC connects specifying the
credentials le:

And now let's make the request from the local host:

In some cases you may want to restrict what messages can be exported from the leaf
node or imported from the leaf connection. You can specify restrictions by limiting what

```
leafnodes { remotes = [
{ url: "tls://connect.ngs.global"
credentials: "/Users/alberto/.nkeys/creds/synadia/leaftest/leaf },
]}
```
```
nats-server -c /tmp/ngs_leaf.conf
...[4985] 2023/03/03 10:55:51.577569 [INF] Listening for client connections
...[4985] 2023/03/03 10:55:51.918781 [INF] Connected leafnode to "connect.ng
```
```
nsc reply q 42
```
```
nats-req q ""
Published [q] : ''Received [_INBOX.hgG0zVcVcyr4G5KBwOuyJw.uUYkEyKr] : '42'
```
## Leaf Authorization


the leaf connection client can publish and subscribe to. See NATS Authorization for how
you can do this.

As of NATS v2.10.0
Leafnode connections follow the model where when a TCP connection is created to the
server, the server will immediately send an INFO protocol message in clear text. This
INFO protocol provides metadata, including whether the server requires a secure
connection.
Some environments prefer not to want a server that is congured to accept TLS
connections for leafnodes having any trac sent in clear text. It was possible to by-pass
this using a websocket connection. However, if websocket is not desired, the accepting
and remote servers can be congured to perform a TLS handshake before sending the
INFO protocol message.

## TLS-first Handshake


# Configuration

The leaf node conguration block is used to congure incoming as well as outgoing leaf
node connections. Most properties are for the conguration of incoming connections.
The properties remotes and reconnect are for outgoing connections.
**Property Description**
host Interface where the server will listen for incoming leafnode connections.
port Port where the serleafnode connections (default is 7422).ver will listen for incoming^
listen Combines host and port as <host>:<port>
tls TLS conguration block (same as other nats-server tls conguration).
advertise Hostporcontacted bt <host>:<port>y leaf nodes. This is useful in cluster setups with N to advertise how this server can be^ AT.
no_advertise if true the server shouldn't be advertised to leaf nodes.
authorization Authorization block. **See Authorization Block section below**.
remotes List of where leafnode client connection can be made.remote entries specifying servers^
reconnect Interval in seconds at which rattempts to a remote server are made.econnect^
compression Congurto cluster res comproutes. Defaults tession of leafnode connections similaro s2_auto. See details here.^

## leafnodes Configuration Block

## TLS Block


As of NATS v2.10.0
The tls block for leafnodes conguration has an additional eld enabling a TLS-rst
handshake between the remote and the accepting server.

Since Leafnodes can connect to a variety of servers, the ability to indicate if the TLS
handshake should be done rst is congured in 2 places. The accepting side is in the
tls block of the leafnodes block.

With the above conguration, an older server, or a server that does not have the remote
conguration also congured with handshake_first: true, will fail to create a
leafnode connection because the accepting-side server will initiate the TLS handshake
while the soliciting side will wait for the INFO protocol to be received.

To indicate that a leafnode connection should perform the TLS handshake rst, it needs
to be congured in the remote conguration:

```
Property Description
handshake_first If to also hatrue on the accepting side, rve this setting conguremote leafnodes ared in the remotese required section.^
```
```
leafnodes { port: 7422
tls { handshake_first: true
# other TLS fields... }
}
```
### Accepting side

### Remote side


If the remote is congured as such but the server it is connecting to does not have
handshake_first: true congured, the connection will fail since the solicit side is
performing a TLS handshake but will receive an INFO protocol in clear.

```
leafnodes { remotes [
{ urls: ["tls://example:7422"]
tls: { handshake_first: true
# other TLS fields... }
} ]
}
```
```
Property Description
user Username for the leaf node connection.
password Password for the user entry.
account Account this leaf node connection should be bound to.
timeout Maximum number of seconds to wait for leaf node authentication.
users List of crnode connections. edentials and account t See User Block section below o bind to leaf^.
```
```
Property Description
user Username for the leaf node connection.
password Password for the user entry.
```
## Authorization Block

### Users Block


Here are some examples of using basic user/password authentication for leaf nodes
(note while this is using accounts it is not using JWTs)
Singleton mode:

With above conguration, if a soliciting server creates a Leafnode connection with url:
nats://leaf:secret@host:port, then the accepting server will bind the leafnode
connection to the account "TheAccount". This account need to exist otherwise the
connection will be rejected.
Multi-users mode:

With the above, if a server connects using leaf1:secret@host:port, then the accepting
server will bind the connection to account account1. If using leaf2 user, then the
accepting server will bind to connection to account2.

```
Property Description
account Account this leaf node connection should be bound to.
```
```
leafnodes { port: ...
authorization { user: leaf
password: secret account: TheAccount
}}
```
```
leafnodes { port: ...
authorization { users = [
{user: leaf1, password: secret, account: account1} {user: leaf2, password: secret, account: account2}
] }
}
```

If username/password (either singleton or multi-users) is dened, then the connecting
server MUST provide the proper credentials otherwise the connection will be rejected.
If no username/password is provided, it is still possible to provide the account the
connection should be associated with:

With the above, a connection without credentials will be bound to the account
"TheAccount".
If other form of credentials are used (jwt, nkey or other), then the server will attempt to
authenticate and if successful associate to the account for that specic user. If the user
authentication fails (wrong password, no such user, etc..) the connection will be also
rejected.

```
leafnodes { port: ...
authorization { account: TheAccount
}}
```
```
Property Description
url Leafnode URL (URL protocol should be nats-leaf).
urls Leafnode URL arre.g., urls: [ "nats-leaf:/ay. Suppor/host1:7422", "ts multiple URLs for disconats-leaf://host2:7422" ]very,^
```
```
account Accountaccount ton this account will be for name or JWo bind to this remote serT public kwarded to the remote serey identifying the localver. Any trac locallyver.^
credentials Credential le for connecting to the leafnode server.
tls A specied TLS congurTLS certicates when connecting/authenticating.ation block. Leafnode client will use^
```
## LeafNode remotes Entry Block


As of NATS Server v.2.9.0, for users embedding the NATS Server, it is possible to replace
the use of the credentials le by a signature callback which will sign the nonce and
provide the JWT in the CONNECT protocol. The RemoteLeafOpts has a new eld:

The callback denition is:

And example of how to use it can be found here

Since NATS 2.2.0, Leaf nodes support outbound WebSocket connections by specifying
ws as the scheme component of the remote server URLs:

```
Property Description
ws_compression If connecting with true or false) indicates tWebsocket protocol, this boolean (o the remote server that
it wishes to use compression. The default is false.
```
```
ws_no_masking If connecting with remote serThe default is ver that it wishes not tfalseWebsocket, which means that outbound fr protocol, this boolean indicates to mask outbound WebSocket frames.ames will be masko the^ ed.
```
```
SignatureCB SignatureHandler
```
```
// SignatureHandler is used to sign a nonce from the server while// authenticating with Nkeys. The callback should sign the nonce and
// return the JWT and the raw signature.type SignatureHandler func([]byte) (string, []byte, error)
```
```
leafnodes { remotes [
{urls: ["ws://hostname1:443", "ws://hostname2:443"]} ]
}
```
### Signature Handler

### Connecting using WebSocket protocol


Note that if a URL has the ws scheme, all URLs the list must be ws. You cannot mix and
match. Therefore this would be considered an invalid conguration:

Note that the decision to make a TLS connection is not based on wss:// (as opposed
to ws://) but instead in the presence of a TLS conguration in the leafnodes{} or the
specic remote conguration block.
To congure Websocket in the remote server, check the Websocket section.

```
remotes [ # Invalid configuration that will prevent the server from starting
{urls: ["ws://hostname1:443", "nats://hostname2:7422"]} ]
```
```
Property Description
cert_file TLS certicate le.
key_file TLS certicate key le.
ca_file TLS certicate authority le.
insecure Skip certicate verication.
verify If true, require and verify client certicates.
verify_and_map If values map certrue, require and verify client certicate values for authentication purposes.ticates and use^
cipher_suites When set, only the specied Values must match golang vTLS cipher suites will be allowed.ersion used to build the server.^
curve_preferences List of TLS cypher curves to use in order.
timeout TLS handshake timeout in fractional seconds.
```
### tls Configuration Block


# JetStream on Leaf Nodes

Using JetStream on Leaf Nodes

If you want to see a demonstration of the full range of this functionality, check out our
video
One of the use cases for a NATS server congured as a leaf node is to provide a local
NATS network even when the connection to a hub or the cloud is down. To support such
a disconnected use case with JetStream, independent JetStream islands are also
supported and available through the same NATS network.
The general issue with multiple, independent JetStreams, accessible from the same
client is that you need to be able to tell them apart. As an example, consider a leaf node
with a non-clustered JetStream in each server. You connect to one of them, but which
JetStream responds when you use the JetStream API $JS.API.>?
To disambiguate between servers, the option domain was added to the JetStream
conguration block. When using it, follow these rules: Every server in a cluster and super
cluster needs to have the same domain name. This means that domain names can only
change between two servers if they are connected via a leaf node connection. As a result
of this the JetStream API $JS.API.> will also be available under a disambiguated name
$JS.<domain>.API.>. Needless to say, domain names need to be unique.
There are reasons to connect system accounts on either end of your leaf node
connection. You probably don't want to connect your cloud and edge device system
accounts, but you might connect them when the only reason keeping you from using a
super cluster are r ewall rules.
The benets are:
Monitoring of all connected nats-servers
nats-account-resolver working on the entire network

## Leaf Nodes


When domain is set, JetStream-related trac on the system account is suppressed.
This is what causes JetStream not to be extended.
In addition, trac on $JS.API.> is also suppressed. This causes clients to use the local
JetStream that is available in the nats-servers they are connected to. To communicate
with another JetStream, that JetStream's unique domain specic prex
$JS.<domain>.API needs to be specied.
Please be aware that each domain is an independent name space. Meaning, inside the
same account it is legal to use the same stream name in different domains.
Furthermore, regular message ow is not restricted. Thus, if the same subject is
subscribed to by different streams in the same account in different domains, as long as
the underlying leaf node was connected at the time, each stream will store the message.
This can be resolved by using the same account but use different subjects in each
domain or use different accounts in each domain or isolate accounts used in leaf nodes.

```
Known issue: if you have more than one JetStream enabled leaf node in a different
cluster, the cluster you connect to also needs JetStream enabled and a domain set.
Known issue: when you intend to extend a central JetStream, by not supplying a
domain name in leaf nodes, that central JetStream needs to be in clustered mode.
```
Below is the cong needed to connect two JetStream enabled servers via a leaf node
connection. In the example, the system accounts are connected for demonstration
purposes (you do not have to do that).

```
extended JetStream cluster
```
### Configuration

```
accounts.conf Imported by Both Servers
```

To be started with nats-server -c hub.conf:

To be started with nats-server -c leaf.conf:

```
accounts { SYS: {
users: [{user: admin, password: admin}] },
ACC: { users: [{user: acc, password: acc}],
jetstream: enabled }
}system_account: SYS
```
```
port: 4222server_name: hub-server
jetstream { store_dir="./store_server"
domain=hub}
leafnodes { port: 7422
}include ./accounts.conf
```
```
hub.conf
```
```
leaf.conf
```

Because the system account is connected, you can obtain the JetStream server report
from both servers.

Create a stream named test subscribing to subject test in the JetStream domain, the
program is connected to. As a result, this stream will be created in the domain hub which
is the domain of the server listening on localhost:4222.

```
port: 4111server_name: leaf-server
jetstream { store_dir="./store_leaf"
domain=leaf}
leafnodes { remotes = [
{ urls: ["nats://admin:admin@0.0.0.0:7422"]
account: "SYS" },
{ urls: ["nats://acc:acc@0.0.0.0:7422"]
account: "ACC" }
]}
include ./accounts.conf
```
```
nats --server nats://admin:admin@localhost:4222 server report jetstream
```
#### ╭│───────────────────────────────────────────── JetStream Summary

#### ├─────────────┬─────────────┬────────┬────────│ Server │ Cluster │ Domain │ Streams │ Consumers │ Messages │

#### ├─────────────┼─────────────┼────────┼────────│ leaf-server │ leaf-server │ leaf │ 0 │ 0 │ 0 │

#### │├─────────────┼─────────────┼────────┼──────── hub-server │ │ hub │ 0 │ 0 │ 0 │

#### │╰─────────────┴─────────────┴────────┴──────── │ │ │ 0 │ 0 │ 0 │

### Usage


To create a stream in a different domain while connected somewhere else, just provide
the js-domain argument. While connected to the same server as before, now the
stream is created in leaf.

```
nats --server nats://acc:acc@localhost:4222 stream add
? Stream Name test? Subjects to consume test
? Storage backend file? Retention Policy Limits
? Discard Policy Old? Stream Messages Limit -1
? Per Subject Messages Limit -1? Message size limit -1
? Maximum message age limit -1? Maximum individual message size -1
? Duplicate tracking time window 2m? Replicas 1
Stream test was created
Information for Stream test created 2021-06-28T12:52:29-04:00
Configuration:
Subjects: test Acknowledgements: true
Retention: File - Limits Replicas: 1
Discard Policy: Old Duplicate Window: 2m0s
Maximum Messages: unlimited Maximum Bytes: unlimited
Maximum Age: 0.00s Maximum Message Size: unlimited
Maximum Consumers: unlimited
State:
Messages: 0 Bytes: 0 B
FirstSeq: 0 LastSeq: 0
Active Consumers: 0
```

Publish a message so there is something to retrieve.

```
nats --server nats://acc:acc@localhost:4222 stream add --js-domain leaf
? Stream Name test? Subjects to consume test
? Storage backend file? Retention Policy Limits
? Discard Policy Old? Stream Messages Limit -1
? Per Subject Messages Limit -1? Message size limit -1
? Maximum message age limit -1? Maximum individual message size -1
? Duplicate tracking time window 2m? Replicas 1
Stream test was created
Information for Stream test created 2021-06-28T12:59:18-04:00
Configuration:
Subjects: test Acknowledgements: true
Retention: File - Limits Replicas: 1
Discard Policy: Old Duplicate Window: 2m0s
Maximum Messages: unlimited Maximum Bytes: unlimited
Maximum Age: 0.00s Maximum Message Size: unlimited
Maximum Consumers: unlimited
State:
Messages: 0 Bytes: 0 B
FirstSeq: 0 LastSeq: 0
Active Consumers: 0
```
```
nats --server nats://acc:acc@localhost:4222 pub test "hello world"
```

Because both streams subscribe to the same subject, each one now reports one
message. This is done to demonstrate the issue. If you want to avoid that, you need to
either use different subjects, different accounts, or one isolated account.

Output

In order to copy a stream from one domain into another, specify the JetStream domain
when creating a mirror. If you want to connect a leaf to the hub and get commands,
even when the leaf node connection is oine, mirroring a stream located in the hub is the
way to go.

```
nats --server nats://acc:acc@localhost:4222 stream report --js-domain le
Obtaining Stream stats
```
#### ╭│───────────────────────────────────────────── Stream Report

#### ├────────┬─────────┬───────────┬──────────┬───│ Stream │ Storage │ Consumers │ Messages │ Bytes │ Lost │ Deleted │ R

#### ├────────┼─────────┼───────────┼──────────┼───│ test │ File │ 0 │ 1 │ 45 B │ 0 │ 0 │

#### ╰────────┴─────────┴───────────┴──────────┴───

```
nats --server nats://acc:acc@localhost:4222 stream report --js-domain hu
```
```
Obtaining Stream stats
```
#### ╭│───────────────────────────────────────────── Stream Report

#### ├────────┬─────────┬───────────┬──────────┬───│ Stream │ Storage │ Consumers │ Messages │ Bytes │ Lost │ Deleted │ R

#### ├────────┼─────────┼───────────┼──────────┼───│ test │ File │ 0 │ 1 │ 45 B │ 0 │ 0 │

#### ╰────────┴─────────┴───────────┴──────────┴───

```
nats --server nats://acc:acc@localhost:4222 stream add --js-domain hub -
```
**Copying across domains via source or mirror**


Similarly, if you want to aggregate streams located in any number of leaf nodes use
source. If the streams located in each leaf are used for the same reasons, it is
recommended to aggregate them in the hub for processing via source.

```
? Stream Name backup-test-leaf? Storage backend file
? Retention Policy Limits? Discard Policy Old
? Stream Messages Limit -1? Message size limit -1
? Maximum message age limit -1? Maximum individual message size -1
? Replicas 1? Adjust mirror start No
? Import mirror from a different JetStream domain Yes? Foreign JetStream domain name leaf
? Delivery prefixStream backup-test-leaf was created
Information for Stream backup-test-leaf created 2021-06-28T14:00:43-04:00
Configuration:
Acknowledgements: true Retention: File - Limits
Replicas: 1 Discard Policy: Old
Duplicate Window: 2m0s Maximum Messages: unlimited
Maximum Bytes: unlimited Maximum Age: 0.00s
Maximum Message Size: unlimited Maximum Consumers: unlimited
Mirror: test, API Prefix: $JS.leaf.API, Delivery Prefix:
State:
Messages: 0 Bytes: 0 B
FirstSeq: 0 LastSeq: 0
Active Consumers: 0
```
```
nats --server nats://acc:acc@localhost:4222 stream add --js-domain hub -
```

source as well as mirror take a copy of the messages. Once copied, accessing the
data is independent of the leaf node connection being online. Copying this way also
avoids having to run a dedicated program of your own. This is the recommended way to
exchange persistent data across domains.

```
? Stream Name aggregate-test-leaf? Storage backend file
? Retention Policy Limits? Discard Policy Old
? Stream Messages Limit -1? Message size limit -1
? Maximum message age limit -1? Maximum individual message size -1
? Duplicate tracking time window 2m? Replicas 1
? Adjust source "test" start No? Import "test" from a different JetStream domain Yes
? test Source foreign JetStream domain name leaf? test Source foreign JetStream domain delivery prefix
Stream aggregate-test-leaf was created
Information for Stream aggregate-test-leaf created 2021-06-28T14:02:36-04
Configuration:
Acknowledgements: true Retention: File - Limits
Replicas: 1 Discard Policy: Old
Duplicate Window: 2m0s Maximum Messages: unlimited
Maximum Bytes: unlimited Maximum Age: 0.00s
Maximum Message Size: unlimited Maximum Consumers: unlimited
Sources: test, API Prefix: $JS.leaf.API, Delivery Prefix:
State:
Messages: 0 Bytes: 0 B
FirstSeq: 0 LastSeq: 0
Active Consumers: 0
```

All of the above happened in the same account. To share domain access across
accounts the account.conf from above needs to be modied and the server restarted
or reloaded. This example exports the consumer and FC API as well as a delivery
subject which is used by the internal push consumer created by source and mirror.
In support of another example on how to share a durable pull consumer for client access
across domains and accounts, the NEXT and ACK API are exported as well.

```
Known issue: Currently, across accounts, push consumer are not supported.
```
On import, the JetStream API prex $JS.hub.API is renamed to JS.test@hub.API.
This is to, once more, disambiguate which JetStream a client in the importing account
might want to interact with. When using domains, the general recommendation is to

```
nats --server nats://acc:acc@localhost:4222 stream report --js-domain hu
Obtaining Stream stats
```
#### ╭│───────────────────────────────────────────── Stream Report

#### ├─────────────────────┬─────────┬───────────┬─│ Stream │ Storage │ Consumers │ Messages │ Bytes │ Lost │

#### ├─────────────────────┼─────────┼───────────┼─│ backup-test-leaf │ File │ 0 │ 1 │ 45 B │ 0 │

#### ││ test aggregate-test-leaf ││ File File ││ 0 0 ││ 1 1 ││ 45 B 98 B ││ 0 0 ││

#### ╰─────────────────────┴─────────┴───────────┴─

#### ╭│───────────────────────────────────────────── Replication Report

#### ├─────────────────────┬────────┬──────────────│ Stream │ Kind │ API Prefix │ Source Stream │ Active

#### ├─────────────────────┼────────┼──────────────│ backup-test-leaf │ Mirror │ $JS.leaf.API │ test │ 0.21s

#### │╰─────────────────────┴────────┴────────────── aggregate-test-leaf │ Source │ $JS.leaf.API │ test │ 1.23s

**Cross account & domain import**


export the domain specic API $JS.<domain>.API as this allows you to pin the export
to a particular domain.
Furthermore, the delivery subject is extended on import. This is to allow for easier export
into multiple accounts.
This example also exports the absolute minimum necessary. It is possible to give access
to the entire consumer API $JS.hub.API.CONSUMER.> or the entire API in a domain
$JS.hub.API.> or the entire API $JS.API.> wherever the importing client connects.


**Copying via source and mirror**
Once the servers have been restarted or reloaded, a mirror can be created as follows
(same applies to source): On import from a different account the renamed prex

```
accounts { SYS: {
users: [{user: admin, password: admin}] },
ACC: { users: [{user: acc, password: acc}],
jetstream: enabled exports: [
# minimum export needed to allow source/mirror to create a co {service: "$JS.hub.API.CONSUMER.CREATE.*", response_type: "st
# minimum export needed for push consumer. This includes sour {stream: "deliver.acc.hub.>"}
# minimum export needed for durable pull consumer `dur` in st {service: "$JS.hub.API.CONSUMER.MSG.NEXT.aggregate-test-leaf.
# minimum export needed to ack messages for durable consumer {service: "$JS.ACK.aggregate-test-leaf.dur.>"}
# minimum export needed for flow control of source/mirror {service: "$JS.FC.aggregate-test-leaf.dur.>"}
] }
IMPORT_MIRROR: { users: [{user: import_mirror, password: import_mirror}],
jetstream: enabled imports: [
{service: {account: ACC, subject: "$JS.hub.API.CONSUMER.CREAT {service: {account: ACC, subject: "$JS.FC.aggregate-test-leaf
{stream: {account: ACC, subject: deliver.acc.hub.import_mirro ]
} # As of now, cross account, only pull consumer are supported.
IMPORT_CLIENT: { users: [{user: import_client, password: import_client}],
jetstream: enabled imports: [
{service: {account: ACC, subject: "$JS.hub.API.CONSUMER.MSG.N {service: {account: ACC, subject: "$JS.ACK.aggregate-test-lea
] }
}system_account: SYS
```

JS.acc@hub.API is provided. In addition, the delivery subject name is extended to also
include the importing domain and stream. This makes it unique to that particular import.
If every delivery prex follows the pattern
<static type>.<exporting account>.<exporting domain>.<importing account>.<importing domain>.<importing domain>.<importing stream name>
overlaps caused by multiple imports are avoided.

```
nats --server nats://import_mirror:import_mirror@localhost:4222 stream a
```

A subsequent check shows that the one message stored in the stream aggregate in
account ACC got copied to the new stream in the account IMPORTER.

```
? Stream Name aggregate-test-leaf-from-acc? Storage backend file
? Retention Policy Limits? Discard Policy Old
? Stream Messages Limit -1? Message size limit -1
? Maximum message age limit -1? Maximum individual message size -1
? Replicas 1? Adjust mirror start No
? Import mirror from a different JetStream domain No? Import mirror from a different account Yes
? Foreign account API prefix JS.acc@hub.API? Foreign account delivery prefix deliver.acc.hub.import_mirror.hub.aggre
Stream aggregate-test-leaf-from-acc was created
Information for Stream aggregate-test-leaf-from-acc created 2021-06-28T16
Configuration:
Acknowledgements: true Retention: File - Limits
Replicas: 1 Discard Policy: Old
Duplicate Window: 2m0s Maximum Messages: unlimited
Maximum Bytes: unlimited Maximum Age: 0.00s
Maximum Message Size: unlimited Maximum Consumers: unlimited
Mirror: aggregate-test-leaf, API Prefix: JS.acc@hub.API, D
State:
Messages: 0 Bytes: 0 B
FirstSeq: 0 LastSeq: 0
Active Consumers: 0
```
```
nats --server nats://import_mirror:import_mirror@localhost:4222 stream r
```

**Direct access of a durable pull consumer**
The modied accounts.conf also includes a separate import for an existing pull
consumer. Let's create a consumer by the name dur in the stream
aggregate-test-leaf in the account acc.

```
Obtaining Stream stats
```
#### ╭│───────────────────────────────────────────── Stream Report

#### ├──────────────────────────────┬─────────┬────│ Stream │ Storage │ Consumers │ Messages │ Bytes

#### ├──────────────────────────────┼─────────┼────│ aggregate-test-leaf-from-acc │ File │ 0 │ 1 │ 98 B

#### ╰──────────────────────────────┴─────────┴────

#### ╭│───────────────────────────────────────────── Replication Report

#### ├──────────────────────────────┬────────┬─────│ Stream │ Kind │ API Prefix │ Source Stream

#### ├──────────────────────────────┼────────┼─────│ aggregate-test-leaf-from-acc │ Mirror │ JS.acc@hub.API │ aggregate-tes

#### ╰──────────────────────────────┴────────┴─────

```
nats --server nats://acc:acc@localhost:4222 consumer add --js-domain hu
```

? Consumer name dur? Delivery target (empty for Pull Consumers)
? Start policy (all, new, last, 1h, msg sequence) all? Replay policy instant
? Filter Stream by subject (blank for all)? Maximum Allowed Deliveries -1
? Maximum Acknowledgements Pending 0? Select a Stream aggregate-test-leaf
Information for Consumer aggregate-test-leaf > dur created 2021-06-28T17:
Configuration:
Durable Name: dur Pull Mode: true
Deliver All: true Ack Policy: Explicit
Ack Wait: 30s Replay Policy: Instant
Max Ack Pending: 20,000 Max Waiting Pulls: 512

State:
Last Delivered Message: Consumer sequence: 0 Stream sequence: 0 Acknowledgment floor: Consumer sequence: 0 Stream sequence: 0
Outstanding Acks: 0 out of maximum 20000 Redelivered Messages: 0
Unprocessed Messages: 1 Waiting Pulls: 0 of maximum 512

nats --server nats://acc:acc@localhost:4222 stream report --js-domain hu


Output

To retrieve the messages stored in the domain hub using nats while connected to the
leaf node, provide the correct stream and durable name as well as the API prex
JS.acc@hub.API

```
Obtaining Stream stats
```
#### ╭│───────────────────────────────────────────── Stream Report

#### ├─────────────────────┬─────────┬───────────┬─│ Stream │ Storage │ Consumers │ Messages │ Bytes │ Lost │

#### ├─────────────────────┼─────────┼───────────┼─│ backup-test-leaf │ File │ 0 │ 1 │ 45 B │ 0 │

#### ││ test aggregate-test-leaf ││ File File ││ 0 1 ││ 1 1 ││ 45 B 98 B ││ 0 0 ││

#### ╰─────────────────────┴─────────┴───────────┴─

#### ╭│───────────────────────────────────────────── Replication Report

#### ├─────────────────────┬────────┬──────────────│ Stream │ Kind │ API Prefix │ Source Stream │ Active

#### ├─────────────────────┼────────┼──────────────│ backup-test-leaf │ Mirror │ $JS.leaf.API │ test │ 1.85s

#### │╰─────────────────────┴────────┴────────────── aggregate-test-leaf │ Source │ $JS.leaf.API │ test │ 1.85s

```
nats --server nats://acc:acc@localhost:4222 consumer report --js-domain
```
#### ? Select a Stream aggregate-test-leaf╭─────────────────────────────────────────────

#### │├──────────┬──────┬────────────┬──────────┬─── Consumer report for aggregate-test-leaf with 1

#### │├──────────┼──────┼────────────┼──────────┼─── Consumer │ Mode │ Ack Policy │ Ack Wait │ Ack Pending │ Redelivered │

#### │╰──────────┴──────┴────────────┴──────────┴─── dur │ Pull │ Explicit │ 30.00s │ 0 │ 0 │

```
nats --server nats://import_client:import_client@localhost:4111 consumer
```

This works similarly when writing your own client. To avoid waiting for the ack timeout, a
new message is sent on test from where it is copied into aggregate-test-leaf.

The client is connected to the leaf node and receives the message just sent.

```
[17:44:16] subj: test / tries: 1 / cons seq: 1 / str seq: 1 / pending: 0
Headers:
Nats-Stream-Source: test:mSx7q4yJ 1
Data:
hello world
Acknowledged message
nats --server nats://acc:acc@localhost:4222 consumer report --js-domain
```
#### ? Select a Stream aggregate-test-leaf╭─────────────────────────────────────────────

#### │├──────────┬──────┬────────────┬──────────┬─── Consumer report for aggregate-test-leaf with 1

#### │├──────────┼──────┼────────────┼──────────┼─── Consumer │ Mode │ Ack Policy │ Ack Wait │ Ack Pending │ Redelivered │

#### │╰──────────┴──────┴────────────┴──────────┴─── dur │ Pull │ Explicit │ 30.00s │ 0 │ 0 │

```
nats --server nats://acc:acc@localhost:4222 pub test "hello world 2"
```
```
./main nats://import_client:import_client@localhost:4111
starting&{Sequence:{Consumer:3 Stream:3} NumDelivered:1 NumPending:0 Timestamp:20
hello world 2nats: timeout
^Cnats: timeout
```

There the API prex is communicated with setting the option
nats.APIPrefix("JS.acc@hub.API") when obtaining the JetStream object. Because
the API access is limited, the subscribe call provides the option
nats.Bind("aggregate-test-leaf", "dur") which prevents calls to infer the stream
and durable name.


package main
import "fmt" (
"os""os/signal"
"syscall""time"

(^) ) "github.com/nats-io/nats.go"
func nc, err main() {:= nats.Connect(os.Args[ 1 ], nats.Name("JS test"))
deferif err nc.!=Close nil {()
fmt. returnPrintf("nats connect: %v\n", err)
} js, err := nc.JetStream(nats.APIPrefix("JS.acc@hub.API"))
(^) fmt.if err !=Printf nil {("JetStream: %v\n", err)
if js return== nil {
} }
s, err if err :=!= js. nilPullSubscribe { ("", "dur", nats.Bind("aggregate-test-leaf"
fmt. returnPrintf("PullSubscribe: %v\n", err)
}
shutdown signal.Notify:= make(shutdown, os.Interrupt, syscall.SIGTERM)(chan os.Signal, 1 )
fmt. for {Printf("starting\n")
selectcase <- {shutdown:
(^) defaultreturn:
(^) fmt.if m, err Println:= s.Fetch(err)( 1 , nats.MaxWait(time.Second)); err != ni
} else {
(^) fmt.if meta, err Printf:=( m["%+v 0 ].\n"Metadata, meta)(); err == nil {
}


A push subscriber will need a similar setup. It will require the ACK subject. However,
instead of exporting/importing the NEXT subject, the delivery subject shown for
source/mirror needs to be used.

JetStream Leaf Nodes demo and its companion video recording.

```
} fmt.Println(string(m[ 0 ].Data))
```
(^) fmt.if err :=Printf m[ 0 ].(Ack"ack error: (); err !=%+v nil\n" {, err)
} }
} }
}

## See Also


# Securing NATS

The NATS server provides several forms of security:
Connections can be encrypted with TLS
Client connections can require authentication
Clients can require authorization for subjects they publish or subscribe to


# Enabling TLS

The NATS server uses modern TLS semantics to encrypt client, route, and monitoring
connections. Server conguration revolves around a tls map, which has the following
properties:
**Property Description**
cert_file TLS certicate le.
key_file TLS certicate key le.
ca_file TLS When not prto the system trust stcerticate authority leesent, defaultore.^.^

```
cipher_suites
```
```
When set, only the specied cipher suites will be allowed.TLS
Values must match the golangversion used to build the server.
curve_preferences List of TLS ciphercurves to use in or^ der.
```
```
insecure
```
```
Skip cerThis only applies tticate verication.o
```
outgoing connections, NOincoming client connections.T (^)
**NOT Recommended**
timeout TLS handshakin fractional seconds.Default set to e 2 timeout seconds.^
verify
If client certrue, require and ticates. To supporverify (^) t
use by Browser, this optiondoes not apply to monitoring.
verify_and_map If client certrue, require and verifyticates and
map certicate values for


**Property Description** authentication purposes. Does
not apply to monitoring either.

```
verify_cert_and_check_know
n_urls
```
Only settable in a non clientcontext where verify: true (^) is
the default (The incoming connectionscluster/gateway ).
certicate 's X509v3 Subject
Alternative NameDNS entries will be matched
against all urls in theconguration context that (^)
contains this tls map. If a matchis found, the connection is
accepted and rMeaning for gatewaejected otherys we willwise.
match all DNS entries in thecerticate against all gatewa (^) y
urls. For clusteragainst all route urls. As a, we will match
consequence of this, dynamiccluster growth may require cong (^)
changes in other clusters wherthis ag is true. DNS name e
checking is perto rfc6125. Only the full wildcarformed accordingd
most label. * is supporThis would be oneted for the left^
way to keep cluster grexible. owth
pinned_certs List of hex-encoded SHDER encoded public keyA256 of
ngerprints. When prduring the TLS handshakesent,e, the (^)
provided ceris required to be present in theticate 's ngerprint
list or the connection is closed.This sequence of commands
generates an entrcerticate: `openssl x509 -noout -y for a provided
pubkey -in
opensslpkey -pubin -
outform DER
openssl dgst-sha256`.


The simplest conguration:

Or by using server options:

Notice that the log indicates that the client connections will be required to use TLS. If you
run the server in Debug mode with -D or -DV, the logs will show the cipher suite
selection for each connected client:

When a tls section is specied at the root of the conguration, it also affects the
monitoring port if https_port option is specied. Other sections such as cluster can
specify a tls block.

```
Property Description
```
```
tls: { cert_file: "./server-cert.pem"
key_file: "./server-key.pem"}
```
```
nats-server --tls --tlscert=./server-cert.pem --tlskey=./server-key.pem
[21417] 2019/05/16 11:21:19.801539 [INF] Starting nats-server version 2.0[21417] 2019/05/16 11:21:19.801621 [INF] Git commit [not set]
[21417] 2019/05/16 11:21:19.801777 [INF] Listening for client connections [21417] 2019/05/16 11:21:19.801782 [INF] TLS required for client connecti
[21417] 2019/05/16 11:21:19.801785 [INF] Server id is ND6ZZDQQDGKYQGDD6QN[21417] 2019/05/16 11:21:19.801787 [INF] Server is ready
```
```
[22242] 2019/05/16 11:22:20.216322 [DBG] 127.0.0.1:51383 - cid:1 - Client [22242] 2019/05/16 11:22:20.216539 [DBG] 127.0.0.1:51383 - cid:1 - Starti
[22242] 2019/05/16 11:22:20.367275 [DBG] 127.0.0.1:51383 - cid:1 - TLS ha[22242] 2019/05/16 11:22:20.367291 [DBG] 127.0.0.1:51383 - cid:1 - TLS ve
```
## TLS Timeout


The timeout setting enables you to control the amount of time that a client is allowed
to upgrade its connection to tls. If your clients are experiencing disconnects during TLS
handshake, you'll want to increase the value, however, if you do be aware that an
extended timeout exposes your server to attacks where a client doesn't upgrade to TLS
and thus consumes resources. Conversely, if you reduce the TLS timeout too much,
you are likely to experience handshake errors.

The ca_file le should contain one or more Certicate Authorities in PEM format, in a
bundle. This is a common format.
When a certicate is issued, it is often accompanied by a copy of the intermediate
certicate used to issue it. This is useful for validating that certicate. It is not
necessarily a good choice as the only CA suitable for use in verifying other certicates a
server may see.
Do consider though that organizations issuing certicates will change the intermediate
they use. For instance, a CA might issue intermediates in pairs, with an active and a
standby, and reserve the right to switch to the standby without notice. You probably
would want to trust both of those for the ca_file directive, to be prepared for such a
day, and then after the rst CA has been compromised you can remove it. This way the
roll from one CA to another will not break your NATS server deployment.

Explaining Public key infrastructure, Certicate Authorities (CA) and x509 certicates fall
well outside the scope of this document. So does an explanation on how to obtain a

```
tls: { cert_file: "./server-cert.pem"
key_file: "./server-key.pem" # clients will fail to connect (value is too low)
timeout: 0.0001}
```
## Certificate Authorities

## Self Signed Certificates for Testing


properly trusted certicates.
If anybody outside your organization needs to connect, get certs from a public certicate
authority. Think carefully about revocation and cycling times, as well as automation, when
picking a CA. If arbitrary applications inside your organization need to connect, use a cert
from your in-house CA. If only resources inside a specic environment need to connect,
that environment might have its own dedicated automatic CA, eg in Kubernetes clusters,
so use that.
**Only** for **testing** purposes does it make sense to generate self-signed certicates, even
your own CA. This is a **short** guide on how to do just that and what to watch out for.

```
DO NOT USE these certicates in production!!!
```
As they should, these are **not trusted** by the system your server or clients are running on.
One option is to specify the CA in every client you are using. In case you make use of
verify, verify_and_map or verify_cert_and_check_known_urls you need to
specify ca_file in the server. If you are having a more complex setup involving cluster,
gateways or leaf nodes, ca_file needs to be present in tls maps used to connect to
the server with self-signed certicates. While this works for server and libraries from the
NATS ecosystem, you will experience issues when connecting with other tools such as
your Browser.
Another option is to congure your system's trust store to include self-signed
certicate(s). Which trust store needs to be congured depends on what you are testing.
This may be your OS for server and certain clients.
The runtime environment for other clients like Java, Python or Node.js.
Your browser for monitoring endpoints and websockets.

### Problems With Self Signed Certificates

**Missing in Relevant Trust Stores**


Please check your system's documentation on how to trust a particular self-signed
certicate.

Another common problem is failed identity validation. The IP or DNS name to connect to
needs to match a Subject Alternative Name (SAN) inside the certicate. Meaning, if a
client/browser/server connect via tls to 127.0.0.1, the server needs to present a
certicate with a SAN containing the IP 127.0.0.1 or the connection will be closed with
a handshake error.
When verify_cert_and_check_known_urls is specied, Subject Alternative Name
(SAN) DNS records are necessary. In order to successfully connect there must be an
overlap between the DNS records provided as part of the certicate and the urls
congured. If you dynamically grow your cluster and use a new certicate, this route or
gateway the server connects to will have to be recongured to include an url for the new
server. Only then can the new server connect. If the DNS record is a wildcard, matching
according to rfc6125 will be performed. Using certicates with a wildcard Subject
Alternative Name (SAN) and conguration with url(s) that would match are a way to keep
the exibility of dynamic cluster growth without conguration changes in other clusters.

When generating your certicate you need to make sure to include the right purpose for
which you want to use the certicate. This is encoded in key usage and extended key
usage. The necessary values for key usage depend on the ciphers used.
Digital Signature and Key Encipherment are an interoperable choice.
With respect to NATS the relevant values for extended key usage are:

```
connections. A NTLS WWW server authenticationATS server will need a cer - To authenticate as serticate containing this.ver for incoming^
connections. Only needed when connecting tTLS WWW client authentication - To authenticate as client for outgoingo a server where verify,^
cases, a Nverify_and_mapATS client will need a cer or verify_cert_and_check_known_urlsticate with this value. are specied. In these^
```
**Missing Subject Alternative Name**

**Wrong Key Usage**


Note that it's common practice for non-web protocols to use the TLS WWW authentication
elds, as a matter of history those have become embedded as generic options.

The simplest way to generate a CA as well as client and server certicates is mkcert. This
zero cong tool generates and installs the CA into your **local** system trust store(s) and
makes providing SAN straight forward. Check its documentation for installation and your
system's trust store. Here is a simple example:
Generate a CA as well as a certicate, valid for server authentication by localhost and
the IP ::1(-cert-file and -key-file overwrite default le names). Then start a
NATS server using the generated certicate.

Now you should be able to access the monitoring endpoint https://localhost:8222
with your browser.
https://127.0.0.1:8222 however should result in an error as 127.0.0.1 is not listed
as SAN. You will not be able to establish a connection from another computer either. For
that to work you have to provide appropriate DNS and/or IP SAN(s)
To generate certicates that work with verify and cluster/gateway/leaf_nodes
provide the -client option. It will cause the appropriate key usage for client

Leaf nodeconnecting N connections can be congurATS server will have to present a cered with verifyticate with this v as well. Then thealue too. (^)
Certicates containing both values are an option.
Clusterserver comes down t connections alwao timing and therys have verifyefore can't be individually congur enabled. Which server acts as client anded.
Certicates containing both values are a must.
Gatewayconnections can specify a separ connections always have ate cert. Certicates containing both vverify enabled. Unlike cluster outgoingalues ar (^) e an
option that reduce conguration.
mkcertmkcert -install-cert-file server-cert.pem -key-file server-key.pem localhost ::1
nats-server --tls --tlscert=server-cert.pem --tlskey=server-key.pem -ms 8

### Creating Self Signed Certificates for Testing


authentication to be added. This example also adds a SAN email for usage as user name
in verify_and_map.

```
Please note:
```
Examples in this document make use of the certicates generated so far. To simplify
examples using the CA certicate, copy rootCA.pem into the same folder where the
certicates were generated. To obtain the CA certicate 's location use this command:

Once you are done testing, remove the CA from your **local** system trust store(s).

Alternatively, you can also use openssl to generate certicates. This tool allows a lot
more customization of the generated certicates. It is **more complex** and does **not
manage** installation into the system trust store(s).
However, for inspecting certicates it is quite handy. To inspect the certicates from the
above example execute these commands:

```
mkcert -client -cert-file client-cert.pem -key-file client-key.pem localh
```
```
That client refers to connecting process, not necessarily a NATS client.
server authentication.mkcert -client will generate a certicate with key usage suitable for client and^
```
```
mkcert -CAROOT
```
```
mkcert -uninstall
```
```
opensslopenssl x509x509 -noout-noout -text-text -in-in server-cert.pemclient-cert.pem
```
## TLS-Terminating Reverse Proxies


Due to the nature of the TLS upgrade mechanism NATS uses, using a TLS-terminating
reverse proxy with NATS is not supported. However, there are workarounds that can be
used in the client libraries to make it work.

Provide a CustomDialer.

See: https://github.com/nats-io/nats.js/issues/369

See: https://github.com/nats-io/nats.rs/blob/main/async-nats/src/connector.rs

```
package io.nats.client.impl;
public @Override class TlsSocketDataPort extends SocketDataPort {
```
(^) superpublic void.connect connect(serverURI(String, serverURI conn, timeoutNanos);, NatsConnection conn, long timeout
(^) } this.upgradeToSecure();
}
Nats .server.connect(server)(new Options.Builder()
..dataPortTypebuild()) ("io.nats.client.impl.TlsSocketDataPort")

### nats.go

### nats.java

### nats.js

### nats.rs

### nats.net


See: https://github.com/nats-
io/nats.net/tree/main/src/Samples/TLSReverseProxyExample


# Authentication

NATS authentication is multi-level. All of the security modes have an accounts level with
users belonging to those accounts. The decentralized JWT Authentication also has an
operator to which the accounts belong.
Each account has its own independent subject namespace: a message published on
subject 'foo' in one account will not be seen by subscribers to 'foo' in other accounts.
Accounts can however dene exports and imports of subject(s) streams as well as
expose request-reply services between accounts. Users within an account will share the
same subject namespace but can be restricted to only be able to publish-subscribe to
specic subjects.

The NATS server provides various ways of authenticating clients:

Authentication deals with allowing a NATS client to connect to the server. Except for JWT
authentication, authentication and authorization are congured in the authorization
section of the conguration. With JWT authentication the account and user information
are stored in the resolver rather than in the server conguration le.

```
Token Authentication
Plain Text Username/Password credentials
Bcrypted Username/Password credentials
TLS Certicate
NKEY with Challenge
Decentralized JWT Authentication/Authorization
```
## Authentication Methods

## Authorization Map


The authorization block provides authentication conguration as well as
authorization:

A user conguration map species credentials and permissions options for a single
user:

```
Property Description
token Species a global tto the server (exclusive of user and passworoken that can be used to authenticated)^
user Species a single clients to the server (exclusive of token)global user name for^
password Species a single clients to the server (exclusive of global password fortoken^ )
users A list of and passworuser congurd credentials, specify a ation maps. For multiple usernameusers list.^
timeout Maximum number of seconds to wait for client authentication
auth_callout Enables the auth callout extension
```
```
Property Description
user username for client authentication. (Canalso be a user for tls authentication)^
password password for the user entry
nkey public nkey identifying an user
permissions permissions map conguring subjects accessible to the user
```
## User Configuration Map


# Tokens

Token authentication is a string that if provided by a client, allows it to connect. It is the
most straightforward authentication provided by the NATS server.
To use token authentication, you can specify an authorization section with the
token property set:

Token authentication can be used in the authorization section for clients and clusters.
Or start the server with the --auth ag:

A client can easily connect by specifying the server URL:

Tokens can be bcrypted enabling an additional layer of security, as the clear-text version
of the token would not be persisted on the server conguration le.
You can generate bcrypted tokens and passwords using the nats tool:

```
authorization { token: "s3cr3t"
}
```
```
nats-server --auth s3cr3t
```
```
nats sub -s nats://s3cr3t@localhost:4222 ">"
```
```
nats server passwd
? Enter password [? for help] **********************? Reenter password [? for help] **********************
$2a$11$PWIFAL8RsWyGI3jVZtO9Nu8.6jOxzxfZo7c/W0eLk017hjgUKWrhy
```
## Bcrypted Tokens


Here's a simple conguration le:

The client will still require the clear-text token to connect:

```
authorization { token: "$2a$11$PWIFAL8RsWyGI3jVZtO9Nu8.6jOxzxfZo7c/W0eLk017hjgUKWrhy"
}
```
```
nats sub -s nats://dag0HTXl4RGg7dXdaJwbC8@localhost:4222 ">"
```

# Username/Password

You can authenticate one or more clients using username and passwords; this enables
you to have greater control over the management and issuance of credential secrets.

You can also specify a single username/password by:

Username/password also supports bcrypted passwords using the nats tool. Simply
replace the clear text password with the bcrypted entries:

```
authorization: { user: a,
password: b}
```
```
> nats-server --user a --pass b
```
```
authorization: { users: [
{user: a, password: b}, {user: b, password: a}
]}
```
## Plain Text Passwords

## Single User

## Multiple users

## Bcrypted Passwords


And on the conguration le:

As you add/remove passwords from the server conguration le, you'll want your
changes to take effect. To reload without restarting the server and disconnecting clients,
do:

```
> nats server passwd? Enter password [? for help] **********************
? Reenter password [? for help] **********************
$2a$11$V1qrpBt8/SLfEBr4NJq4T.2mg8chx8.MTblUiTBOLV3MKDeAy.f7u
```
```
authorization: { users: [
{user: a, password: "$2a$11$V1qrpBt8/SLfEBr4NJq4T.2mg8chx8.MTblUiT ...
]}
```
```
> nats-server --signal reload
```
## Reloading a Configuration


# TLS Authentication

The server can require TLS certicates from a client. When needed, you can use the
certicates to:

```
Note: To simplify the common scenario of maintainers looking at the monitoring
endpoint, verify and verify_and_map do not apply to the monitoring port.
```
The examples in the following sections make use of the certicates you generated
locally.

The server can verify a client certicate using a CA certicate. To require verication, add
the option verify to the TLS conguration section as follows:

Or via the command line:

This option veries the client's certicate is signed by the CA specied in the ca_file
option. When ca_file is not present it will default to CAs in the system trust store. It
also makes sure that the client provides a certicate with the extended key usage
TLS Web Client Authentication.

```
Validate the client certicate matches a known or trusted CA
Extract information from a trusted certicate to provide authentication
```
```
tls { cert_file: "server-cert.pem"
key_file: "server-key.pem" ca_file: "rootCA.pem"
verify: true}
```
```
nats-server --tlsverify --tlscert=server-cert.pem --tlskey=server-key.pem
```
## Validating a Client Certificate


In addition to verifying that a specied CA issued a client certicate, you can use
information encoded in the certicate to authenticate a client. The client wouldn't have to
provide or track usernames or passwords.
To have TLS Mutual Authentication map certicate attributes to the user's identity use
verify_and_map as shown as follows:

```
Note that verify was changed to verify_and_map.
```
When present, the server will check if a Subject Alternative Name (SAN) maps to a user. It
will search all email addresses rst, then all DNS names. If no user could be found, it will
try the certicate subject.

```
Note: This mechanism will pick the user it nds rst. There is no conguration to
restrict this.
```
```
tls { cert_file: "server-cert.pem"
key_file: "server-key.pem" ca_file: "rootCA.pem"
# Require a client certificate and map user id from certificate verify_and_map: true
}
```
```
openssl x509 -noout -text -in client-cert.pem
Certificate:...
X509v3 extensions: X509v3 Subject Alternative Name:
DNS:localhost, IP Address:0:0:0:0:0:0:0:1, email:email@lo X509v3 Extended Key Usage:
TLS Web Client Authentication...
```
## Mapping Client Certificates To A User


The conguration to authorize this user would be as follow:

Use the RFC 2253 Distinguished Names syntax to specify a user corresponding to the
certicate subject:

```
Note that for this example to work you will have to modify the user to match what is in
your certicates subject. In doing so, watch out for the order of attributes!
```
The conguration to authorize this user would be as follows:

TLS timeout is described here.

```
authorization { users = [
{user: "email@localhost"} ]
}
```
```
openssl x509 -noout -text -in client-cert.pem
Certificate: Data:
... Subject: O=mkcert development certificate, OU=testuser@MacBook-Pr
...
```
```
authorization { users = [
{user: "OU=testuser@MacBook-Pro.local (Test User),O=mkcert developmen ]
}
```
## TLS Timeout


# TLS Authentication in clusters

When setting up clusters, all servers in the cluster, if using TLS, will both verify the
connecting endpoints and the server responses. So certicates are checked in both
directions. Certicates can be congured only for the server's cluster identity, keeping
client and server certicates separate from cluster formation.
TLS Mutual Authentication is the only way of securing routes.

```
cluster { listen: 127.0.0.1:4244
tls { # Route cert
cert_file: "./configs/certs/srva-cert.pem" # Private key
key_file: "./configs/certs/srva-key.pem" # Optional certificate authority verifying connected routes
# Required when we have self-signed CA, etc. ca_file: "./configs/certs/ca.pem"
} # Routes are actively solicited and connected to from this server.
# Other servers can connect to us if they supply the correct credential # in their routes definitions from above.
routes = [ nats://127.0.0.1:4246
]}
```

# NKeys

NKeys are a new, highly secure public-key signature system based on Ed25519.
With NKeys the server can verify identities without ever storing or ever seeing private
keys. The authentication system works by requiring a connecting client to provide its
public key and digitally sign a challenge with its private key. The server generates a
random challenge with every connection request, making it immune to playback attacks.
The generated signature is validated against the provided public key, thus proving the
identity of the client. If the public key is known to the server, authentication succeeds.

```
NKey is an excellent replacement for token authentication because a connecting
client will have to prove it controls the private key for the authorized public key.
```
To generate nkeys, you'll need the nk tool.

To generate a User NKEY:

The rst output line starts with the letter S for Seed. The second letter, U stands for
User. Seeds are private keys; you should treat them as secrets and guard them with care.
The second line starts with the letter U for User and is a public key which can be safely
shared.
To use nkey authentication, add a user, and set the nkey property to the public key of
the user you want to authenticate:

```
nk -gen user -pubout
SUACSSL3UAHUDXKFSNVUZRF5UHPMWZ6BFDTJ7M6USDXIEDNPPQYYYCU3VYUDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4
```
## Generating NKeys and Configuring the Server


Note that the user section sets the nkey property (user/password/token properties are
not needed). Add permission sections as required.

Now that you have a user nkey, let's congure a client to use it for authentication. As an
example, here are the connect options for the node client:

The client provides a function that it uses to parse the seed (the private key) and sign the
connection challenge.

```
authorization: { users: [
{ nkey: UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4 } ]
}
```
```
constconst NATSnkeys = =require require('nats'('ts-nkeys'); );
```
constconst nkey_seednc = NATS (^) .connect= ‘SUACSSL3UAHUDXKFSNVUZRF5UHPMWZ6BFDTJ7M6USDXIEDNPPQYYYC({
port nkey:: PORT'UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4', ,
sigCB// client loads seed safely from a file: function (nonce) {
// or some constant like `nkey_seed` defined in// the program
constreturn sk sk =.sign nkeys(nonce);.fromSeed(Buffer.from(nkey_seed));
}});
...

## Client Configuration


# Authentication Timeout

Much like the tls timeout option, authentication can specify a timeout value.
If a client doesn't authenticate to the server within the specied time, the server
disconnects the server to prevent abuses.
Timeouts are specied in seconds (and can be fractional).
As with TLS timeouts, long timeouts can be an opportunity for abuse. If setting the
authentication timeout, it is important to note that it should be longer than the
tls timeout option, as the authentication timeout includes the TLS upgrade time.
authorization: { timeout: 3
users: [ {user: a, password b},
{user: b, password a} ]
}


# Decentralized JWT

# Authentication/Authorization

With other authentication mechanisms, conguration for identifying a user and Account,
is in the server conguration le. JWT authentication leverages JSON Web Tokens (JWT)
to describe the various entities supported. When a client connects, servers verify the
authenticity of the request using NKeys, download account information and validate a
trust chain. Users are not directly tracked by the server, but rather veried as and
belonging to an Account. This enables the management of users, without requiring server
conguration updates.
Effectively, JWTs improve accounts and provide for a **distributed conguration paradigm**.
Previously each user (or client) needed to be known and authorized a priori in the server’s
conguration requiring an administrator to modify and update server congurations.
These chores are eliminated. User creation can even be performed by different entities
altogether.

```
Note: This scheme improves accounts. Functionalities like isolation or dening
exports/imports between accounts remain! It moves conguration of accounts,
exports/imports or users and their permissions away from the server into several
trusted JSON Web Token (JWT) that are managed separately, therefore removing the
need to congure these entities in each and every server. It furthermore adds
functionalities like expiration and revocation fore decentralized account management
```
JSON Web Tokens (JWT) are an open and industry standard RFC7519 method for
representing claims securely between two parties.

## Decentralized JWT

## Authentication/Authorization

## JSON Web Tokens


Claims are a fancy way of asserting information on a subject. In this context, a subject is
the entity being described (not a messaging subject). Standard JWT claims are typically
digitally signed and veried.
NATS further restricts JWTs by requiring that JWTs be:

NKeys Roles are:

Roles are hierarchical and form a chain of trust. Operators issue Accounts which in turn
issue Users. Servers trust specic Operators. If an account is issued by an operator that
is trusted, account users are trusted.

When a User connects to a server, it presents a JWT issued by its Account. The user
proves its identity by signing a server-issued cryptographic challenge with its private key.
The signature verication validates that the signature is attributable to the user's public
key. Next, the server retrieves the associated account JWT that issued the user. It veries
the User issuer matches the referenced account. Finally, the server checks that a trusted
Operator - one the server is congured with - issued the Account, completing the trust
chain verication.

```
Digitally signed always and only using Ed25519.
NATS adopts the conva public NKEY. ention that all Issuer and Subject elds in a JWT claim must be
Issuer and Subject must match specic roles depending on the claim NKeys.
```
```
Operators
Accounts
Users
```
**NKey Roles**

### The Authentication Process

### The Authorization Process


From an authorization point of view, the account provides information on messaging
subjects that are imported from other accounts (including any ancillary related
authorization) as well as messaging subjects exported to other accounts. Accounts can
also bear limits, such as the maximum number of connections they may have. A user
JWT can express restrictions on the messaging subjects to which it can publish or
subscribe.
When a new user is added to an account, the account conguration need not change, as
each user can and should have its own user JWT that can be veried by simply resolving
its parent account.

One crucial detail to keep in mind is that while in other systems JWTs are used as
sessions or proof of authentication, NATS JWTs are only used as conguration
describing:

Authentication is a public key cryptographic process — a client signs a nonce proving
identity while the trust chain and conguration provides the authorization.
The server is never aware of private keys but can verify that a signer or issuer indeed
matches a specied or known public key.
Lastly, all NATS JWTs (Operators, Accounts, Users and others) are expected to be signed
using the Ed25519 algorithm. If they are not, they are rejected by the system.

There is very little to congure on the nats-server to enable operator JWT security, once
the servers have been initially congured the authentication and authorization tasks are

```
the public ID of the entity
the public ID of the entity that issued it
capabilities of the entity
```
### JWTs and Privacy

### Decentralized Authentication and Authorization -

### Configuration and nsc


typically done by using the nsc administration tool locally and synchronizing with the
account resolvers built into the nats-server.
Conguration is broken up into separate steps. Depending on organizational needs these
are performed by the same or different entities.
Practically, JWT conguration is done using the nsc tool. It can be set up to issue NKeys
and corresponding JWTs for all nkey roles: Operator/Account/User (Example usage).
Despite Account and User creation not happening in server conguration, this model is a
centralized authentication and authorization setup.
Provided institutional trust, it is also possible to use nsc to import account or user public
NKeys and issue corresponding JWTs. This way an operator can issue account JWTs and
a separate entity can issue JWTs for user associated with it's account. Neither entity has
to be aware of the other's private Nkey. This not only allows users to be congured some
place other than servers, but also by different organizations altogether. Say
administrators of a NATS installation controlling operators, issuing account JWTs to
individual prod/dev teams managing their own user. This is a fully decentralized
authorization setup!
With an Operator JWT in place, the server needs to be congured to trust it by specifying
operator. Furthermore the server needs a way to obtain account JWTs. This done by
either defaulting to the resolver specied in the operator jwt or by manually specifying the
resolver. Depending on your conguration an account server needs to be in place

```
It is possible to mix JWT and NKEY/Account based Authentication/Authorization.
```
A lot more information is available in the In Depth Guide.

## Managing JWT authentication


# Account lookup using Resolver

The resolver conguration option is used in conjunction with NATS JWT
Authentication and nsc. The resolver option species a URL where the nats-server can
retrieve an account JWT. There are 3 resolver implementations:

```
If the operator JWT specied in operator contains an account resolver URL,
resolver only needs to be specied in order to overwrite that default.
```
The NATS based resolver is the preferred and easiest way to enable account lookup for
the nats servers. It is built-in into nats-server and stores the account JWTs in a local
(not shared) directory that the server has access to (i.e. you can't have more than one
nats-servers using the same directory. All the servers in the cluster or super-cluster
must be congured to use it, and they implement an 'eventually consistent' mechanism
via NATS and the system account to synchronize (or lookup) the account data between
themselves.
In order to avoid having to store all account JWT on every nats-server (i.e. if you have
a lot of accounts), this resolver has two sub types full and cache.
In this mode of operation administrators typically use the nsc CLI tool to
create/manage the JWTs locally, and use nsc push to push new JWTs to the nats-
servers' built-in resolvers, nsc pull to refresh their local copy of account JWTs, and
nsc revocations to revoke them.

```
NATS Based Resolvselection er which is the preferred option and should be your default
MEMORY if you want to statically dene the accounts in the server conguration
integration of NURL if you want tATS security with some external security system.o build your own account service, typically in order to have some^
```
## NATS Based Resolver


The Full resolver means that the nats-server stores all JWTs and exchanges them in
an eventually consistent way with other resolvers of the same type.

This resolver type also supports resolver_preload. When present, JWTs are listed and
stored in the resolver. There, they may be subject to updates. Restarts of the
nats-server will hold on to these more recent versions.
Not every server in a cluster needs to be set to full. You need enough to still serve your
workload adequately, while some servers are oine.

The Cache resolver means that the nats-server only stores a subset of the JWTs and
evicts others based on an LRU scheme. Missing JWTs are downloaded from the full
nats based resolver(s).

```
resolver type:: { full
# Directory in which account jwt will be storeddir: './jwt'
# In order to support jwt deletion, set to true# If the resolver type is full delete will rename the jwt.
# This is to allow manual restoration in case of inadvertent deletion# To restore a jwt, remove the added suffix .delete and restart or se
# To free up storage you must manually delete files with the suffix .allow_delete: false
# Interval at which a nats-server with a nats based account resolver w# it's state with one random nats based account resolver in the clust
# exchange jwt and converge on the same set of jwt.interval: "2m"
# limit on the number of jwt stored, will reject new jwt once limit ilimit: 1000
}
```
### Full

### Cache


The NATS based resolver utilizes the system account for lookup and upload of account
JWTs. If your application requires tighter integration you can make use of these subjects
for tighter integration.
To upload or update any generated account JWT without nsc, send it as a request to
$SYS.REQ.CLAIMS.UPDATE. Each participating full NATS based account resolver will
respond with a message detailing success or failure.
To serve a requested account JWT yourself and essentially implement an account server,
subscribe to $SYS.REQ.ACCOUNT.*.CLAIMS.LOOKUP and respond with the account JWT
corresponding to the requested account id (wildcard).

To migrate account data when you change from using the standalone (REST) account
server to the built-in NATS account resolver (or between NATS environments, or account
servers) you can use nsc:

1. Run your local machinensc pull to make sure you have a copy of all the account data in the server in
2. Recongure your servers to use the nats resolver instead of the URL resolver

3. Modify the 'REST URL: i.e. just copaccount sery the nats URLs frver URL' setting in yom the operour operator to the nats URL frator's 'service URLs' setting intom the oldo (^)
the account sernsc edit operator --account-jwt-server-url <nats://...>ver URLs.
resolver type:: { cache
# Directory in which account jwt will be storedir: "./"
# limit on the number of jwt stored, will evict old jwt once limit is limit: 1000
# How long to hold on to a jwt before discarding it. ttl: "2m"
}

### NATS Based Resolver - Integration

### Migrating account data


4. do account rnsc push -Aesolver to push your account data to the nats-servers using the built-in nats

You can also pass the account server URLs directly as a ag to the nsc pull and
nsc push commands.

The MEMORY resolver is statically congured in the server's conguration le. You would
use this mode if you would rather manage the account resolving 'by hand' through the
nat-servers' conguration les. The memory resolver makes use of the
resolver_preload directive, which species a map of public keys to account JWTs:

The MEMORY resolver is recommended when the server has a small number of accounts
that don't change very often.
For more information on how to congure a memory resolver, see this tutorial.

**NOTE:** The standalone NATS Account JWT Server is now legacy, please use the NATS
Based Resolver instead. However, the URL resolver option is still available in case you
want to implement your own version of an account resolver
The URL resolver species a URL where the server can append an account public key to
retrieve that account's JWT. Convention for standalone NATS Account JWT Servers is to
serve JWTs at: [http://localhost:9090/jwt/v1/accounts/.](http://localhost:9090/jwt/v1/accounts/.) For such a conguration
you would specify the resolver as follows:

```
resolverresolver_preload: MEMORY: {
ACSU3Q6LTLBVLGAQUONAGXJHVNWGSKKAUA7IY5TB4Z7PLEKSR5O6JTGR: eyJ0eXAiOiJqd3Q}
```
## MEMORY

## URL Resolver


```
Note that if you are not using a nats-account-server, the URL can be anything as long
as by appending the public key for an account, the requested JWT is returned.
```
If the server used requires client authentication, or you want to specify which CA is
trusted for the lookup of account information, specify resolver_tls. This tls
conguration map lets you further restrict TLS to the resolver.

```
resolver: URL(http://localhost:9090/jwt/v1/accounts/)
```

# Memory Resolver Tutorial

The MEMORY resolver is a server built-in resolver for account JWTs. If there are a small
number of accounts, or they do not change too often this can be a simpler conguration
that does not require an external account resolver. Server conguration reload is
supported, meaning the preloads can be updated in the server conguration and reloaded
without a server restart.
The basic conguration for the server requires:

Let's create the setup:

Add an account 'A'

Describe the account

```
The operator JWT
resolver set to MEMORY
account JWresolver_preloadTs. set to an object where account public keys are mapped to^
```
```
nsc add operator -n memory
Generated operator key - private key stored "~/.nkeys/memory/memory.nk"Success! - added operator "memory"
```
```
nsc add account --name A
Generated account key - private key stored "~/.nkeys/memory/accounts/A/A.Success! - added account "A"
```
```
nsc describe account -W
```
## Create Required Entities


Create a new user 'TA'

The nsc tool can generate a conguration le automatically. You provide a path to the
server conguration. The nsc tool will generate the server cong for you:

If you require additional settings, you may want to consider using include in your main
conguration, to reference the generated les. Otherwise, you can start a server and

#### ╭│───────────────────────────────────────────── Account Details

#### ├───────────────────────────┬─────────────────│ Name │ A

#### ││ Account ID Issuer ID ││ ACSU3Q6LTLBVLGAQUONAGXJHVNWGSKKAUA7IY5TB4Z7 ODWZJ2KAPF76WOWMPCJF6BY4QIPLTUIY4JIBLU4K3YD

#### ││ Issued Expires ││ 2019-04-30 20:21:34 UTC

#### ├───────────────────────────┼─────────────────│ Max Connections │ Unlimited

#### ││ Max Leaf Node Connections Max Data ││ Unlimited Unlimited

#### ││ Max Exports Max Imports ││ Unlimited Unlimited

#### ││ Max Msg Payload Max Subscriptions ││ Unlimited Unlimited

#### │├───────────────────────────┼───────────────── Exports Allows Wildcards │ True

#### ││ Imports Exports ││ None None

#### ╰───────────────────────────┴─────────────────

```
nsc add user --name TA
Generated user key - private key stored "~/.nkeys/memory/accounts/A/usersGenerated user creds file "~/.nkeys/memory/accounts/A/users/TA.creds"
Success! - added user "TA" to "A"
```
```
nsc generate config --mem-resolver --config-file /tmp/server.conf
```
## Create the Server Config


reference the generated conguration:

You can then test it.

While generating a conguration le is easy, you may want to craft one by hand to know
the details. With the entities created, and a standard location for the .nsc directory. You
can reference the operator JWT and the account JWT in a server conguration or the
JWT string directly. Remember that your conguration will be in
$NSC_HOME/nats/<operator_name>/<operator_name>.jwt for the operator. The
account JWT will be in
$NSC_HOME/nats/<operator_name>/accounts/<account_name>/<account_name>.jwt
For the conguration you'll need:

The format of the le is:

In this example this translates to:

```
nats-server -c /tmp/server.conf
```
```
The path to the operator JWT
A copy of the contents of the account JWT le
```
```
operator: <path to the operator jwt or jwt itself>resolver: MEMORY
resolver_preload: { <public key for an account>: <contents of the account jwt>
### add as many accounts as you want ...
}
```
## Manual Server Config


Save the cong at server.conf and start the server:

You can then test it.

To test the conguration, simply use one of the standard tools:

```
operator: /Users/synadia/.nsc/nats/memory/memory.jwtresolver: MEMORY
resolver_preload: {ACSU3Q6LTLBVLGAQUONAGXJHVNWGSKKAUA7IY5TB4Z7PLEKSR5O6JTGR: eyJ0eXAiOiJqd3Q
}
```
```
nats-server -c server.conf
```
```
nats pub --creds ~/.nkeys/creds/memory/A/TA.creds hello world
```
## Testing the Configuration


# Mixed

# Authentication/Authorization Setup

Mixing both nkeys static cong and decentralized JWT Authentication/Authorization is
possible but needs some preparation in order to be able to do it.
The way this can be done is by **rst** preparing a basic trusted operator setup that could
be used in the future, and then base from that conguration to create the NKEYS static
cong using the same shared public nkeys for the accounts and then use clustering
routes to bridge the two different auth setups during the transition.
For example, creating the following initial setup using NSC:

This will then generate something like the following:

```
nscnsc addadd accountuser --name--name SYSsys
nscnsc addadd accountuser -a --nameA --name A test
nscnsc addadd accountuser -a --nameB --name B test
```
```
nsc list accounts
```
#### ╭│───────────────────────────────────────────── Accounts │

#### ├──────┬──────────────────────────────────────│ Name │ Public Key │

#### ├──────┼──────────────────────────────────────│ A │ ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2 │

#### ││ B SYS ││ ACWOMQA7PZTKJSBTR7BF6TBK3D776734PWHWDKO7HFMQOM5BIOYPSYZZ ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO ││

#### ╰──────┴──────────────────────────────────────

```
nsc list users -a A
```

We could use this conguration as the initial starting conguration for an nkeys cong
now, where all the NKEYS users public nkeys are explicitly listed (centralized auth model).

By using nsc it is possible to create a mem based resolver for the trusted operator
setup:

#### ╭│───────────────────────────────────────────── Users │

#### ├──────┬──────────────────────────────────────│ Name │ Public Key │

#### ├──────┼──────────────────────────────────────│ test │ UAPOK2P7EN3UFBL7SBJPQK3M3JMLALYRYKX5XWSVMVYK63ZMBHTOHVJR │

#### ╰──────┴──────────────────────────────────────

```
port = 4222
cluster { port = 6222
# We will bridge two different servers with different auth models via r # routes [ nats://127.0.0.1:6223 ]
}
system_account = ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO
accounts { # Account A
ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2 { nkey: ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2
users = [ {nkey: "UAPOK2P7EN3UFBL7SBJPQK3M3JMLALYRYKX5XWSVMVYK63ZMBHTOHVJR" }
] }
# Account B ACWOMQA7PZTKJSBTR7BF6TBK3D776734PWHWDKO7HFMQOM5BIOYPSYZZ {
}
# Account SYS ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO {
}}
```

An example conguration from the second node with the trusted operator setup could
then be:

Even though they have different authorization mechanisms, these two servers are able to
route account messages because they share the same NKEY.
We have created at least one user, in this case with creds:

```
nsc generate config --mem-resolver --sys-account SYS
```
```
port = 4223
cluster { port = 6223
routes [ nats://127.0.0.1:6222 ]}
# debug = true# trace = true
# Operatoroperator = eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJQNDJBSkFTVVA
system_account = ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO
resolver = MEMORY
resolver_preload = { # Account "A"
ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2: eyJ0eXAiOiJqd
# Account "B" ACWOMQA7PZTKJSBTR7BF6TBK3D776734PWHWDKO7HFMQOM5BIOYPSYZZ: eyJ0eXAiOiJqd
# Account "SYS" ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO: eyJ0eXAiOiJqd
}
```

And this same user is able to connect to either one of the servers (bound to 4222 and
4223 respectively):
Subscriber Service:

```
-----BEGIN NATS USER JWT-----eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJNRkM3V1E1N0hKUE9aWUVOT
------END NATS USER JWT------************************* IMPORTANT *************************
NKEY Seed printed below can be used to sign and prove identity.NKEYs are sensitive and should be treated as secrets.
-----BEGIN USER NKEY SEED-----SUANVBWRHHFMGHNIT6UJHPN2TGVBVIILE7VPVNEQ7DGCJ26ZD2V3KAHT4M
------END USER NKEY SEED------*************************************************************
```

Requestor:

```
package main
import "log" (
```
(^) ) "github.com/nats-io/nats.go"
func opts main() {:= make([]nats.Option, 0 )
// Extract public nkey from seed//
// Public: UAPOK2P7EN3UFBL7SBJPQK3M3JMLALYRYKX5XWSVMVYK63ZMBHTOHVJR// Private: SUANVBWRHHFMGHNIT6UJHPN2TGVBVIILE7VPVNEQ7DGCJ26ZD2V3KAHT4
(^) nkey, err // := nats.NkeyOptionFromSeed("path/to/seed.nkey")
(^) log.if err !=Fatal nil {(err)
} opts = append(opts, nkey)
nc, err if err !=:= nats.nil { Connect("127.0.0.1:4222", opts...)
log. } Fatal(err)
nc. log.SubscribePrintf("test"("[Received] , func(m %q*nats, replying... \n".Msg){ , string(m.Data))
m. }) Respond([]byte("pong from nkeys based server"))
(^) } select {}


package main
import "log" (
"time"

(^) ) "github.com/nats-io/nats.go"
func nc, err main() {:= nats.Connect("127.0.0.1:4223", nats.UserCredentials("path/
(^) log.if err !=Fatal nil {(err)
}
(^) resp, err for range time.:=NewTicker nc.Request( 1 *( time.Second).C {"test", []byte("test"), 1 *time.Second)
(^) log.if err !=Println nil {("[Error]", err)
(^) } continue
log. } Println("[Received]", string(resp.Data))
}


# Authorization

The NATS server supports authorization using subject-level permissions on a per-user
basis. Permission-based authorization is available with multi-user authentication via the
users list.
Each permission species the subjects the user can publish to and subscribe to. The
parser is generous at understanding what the intent is, so both arrays and singletons are
processed. For more complex conguration, you can specify a permission object which
explicitly allows or denies subjects. The specied subjects can specify wildcards as well.
Permissions can make use of variables.
A special eld inside the authorization map is default_permissions. When present, it
contains permissions that apply to users that do not have permissions associated with
them.

The permissions map specify subjects that can be subscribed to or published by the
specied client.
**Property Description**
publish subject, list of subjects, or permission map the client can publish

```
subscribe
```
```
subject, list of subjects, or to. In this context it is possible tpermission mapo provide an optional queue name: the client can subscribe
These permissions can also use wildcar<subject> <queue> to express queue grds such as oup permissions.v2.* or^ >.
allow_responses boolean or denies publish tresponses mapo other subjects, howe, default is ver an explicit false. Enabling this implicitlypublish
allow on a subject will override this implicit deny for that subject.
```
## Permissions Configuration Map

## Permission Map


The permission map provides additional properties for conguring a permissions
map. Instead of providing a list of allowable subjects and optional queues, the
permission map allows you to explicitly list those you want toallow or deny. Both
lists can be provided. In case of overlap deny has priority.

**Important Note** It is important to not break request-reply patterns. In some cases (as
shown below) you need to add rules for the _INBOX.> pattern. If an unauthorized client
publishes or attempts to subscribe to a subject that has not been allow listed, the action
fails and is logged at the server, and an error message is returned to the client. The allow
responses option can simplify this.

The allow_responses option dynamically allows publishing to reply subjects and is
designed for service responders. When set to true, an implicit publish allow permission
is enforced which enables the service to have temporary permission to publish to the
reply subject during a request-reply exchange. If true, the client supports a one-time
publish. If allow_responses is a map, it allows you to congure a maximum number
of responses and how long the permission is valid.

```
Note, when publish allow or deny list. allow_responsesThe implication of this is that a r is enabled, the reply subject is not constreply subject can be prained to the ovided by
```
a client tsubject, but is temporo a service (responder) that does not haarily allowed given this option. If explicit contrve permission to explicitly publish on thatol over which subjects a (^)
client is allowed tlists under the publisho reply to, do not use permission map.allow_responses and instead dene allow/deny
**Property Description**
allow List of subject names that are allowed to the client
deny List of subjects that are denied to the client
**Property Description**
max The maximum number of response messages that can be published.

## Allow Responses Map


When allow_responses is set to true, it defaults to the equivalent of { max: 1 }
and no time limit.
**Important Note** When using nsc to congure your users, you can specify the
--allow-pub-response and --response-ttl to control these settings.

Here is an example authorization conguration that uses variables which denes four
users, three of whom are assigned explicit permissions.

```
Property Description
expires The amount of time the permission is vsuch as can be specied. Default doesn1s, 1m, 1h (1 second, minute, hour) etc't have a time limit.alid. Values^
```
## Examples

### Variables


```
default_permissions is a special entry. If dened, it applies to all users that don't
have specic permissions set.
```
authorization { default_permissions = {
publish = "SANDBOX.*" subscribe = ["PUBLIC.>", "_INBOX.>"]
} ADMIN = {
publish = ">" subscribe = ">"
} REQUESTOR = {
publish = ["req.a", "req.b"] subscribe = "_INBOX.>"
} RESPONDER = {
subscribe = ["req.a", "req.b"] publish = "_INBOX.>"
} users = [
{user: admin, password: $ADMIN_PASS, permissions: $ADMIN} {user: client, password: $CLIENT_PASS, permissions: $REQUESTOR}
{user: service, password: $SERVICE_PASS, permissions: $RESPONDER} {user: other, password: $OTHER_PASS}
]}

adminwildcard has >ADMIN to match any subject. permissions and can publish/subscribe on any subject. We use the

clientsubscribe t is a REQUESTORo anything that is a r and can publish response (equests on subjects _INBOX.>). req.a or req.b, and

servicesubscribe t is a o the request subjects and rRESPONDER to req.a and espond treq.b requests, so it needs to clients that can publish ro be able tequests to (^) o
prex req.a_INBOX. and req.b followed b. The reply subject is an inboy a generated string. The x. Typically inbo_INBOX.> subject matches allxes start with the^
subjects that begin with _INBOX..
other has no permissions granted and therefore inherits the default permission set.


```
Note that in the above example, any client with permission to subscribe to _INBOX.>
can receive all responses published. More sensitive installations will want to add or
subset the prex t o further limit subjects that a client can subscribe. Alternatively,
Accounts allow complete isolation limiting what members of an account can see.
```
Here's an example without variables, where the allow and deny options are specied:

Here's an example with allow_responses:

```
authorization: { users = [
{ user: admin
password: secret permissions: {
publish: ">" subscribe: ">"
} }
{ user: test
password: test permissions: {
publish: { deny: ">"
}, subscribe: {
allow: "client.>" }
} }
]}
```
### Allow/Deny Specified

### allow_responses


User a can ony subscribe to foo as part of the queue subscriptions queue. User b
has permissions for queue subscriptions as well as plain subscriptions. You can allow
plain subscriptions on foo but constrain the queues to which a client can join, as well as
preventing any service from using a queue subscription with the name *.prod:

```
authorization: { users: [
{ user: a, password: a }, { user: b, password: b, permissions: {subscribe: "q", allow_respo
{ user: c, password: c, permissions: {subscribe: "q", allow_respo { user: d, password: d, permissions: {subscribe: "q", publish: "x
]}
```
```
User a has no restrictions.
User other publish subjects arb can listen on qe denied implicitly when for requests and can only publish once tallow_responseso reply subjects. is set. All
User and the rc can listen on eply subject can be published at most for q for requests, but is able to return at most 5 r 1 minute. eply messages,
User subject d has the same behax as well, which overrides the implicit deny frvior as user b , except that it can explicitly publish tom allow_responses. o
```
### Queue Permissions


users = [ {
user: "a", password: "a", permissions: { sub: {
allow: ["foo queue"] }
} {
user: "b", password: "b", permissions: { sub: {
# Allow plain subscription foo, but only v1 groups or *.dev queue allow: ["foo", "foo v1", "foo v1.>", "foo *.dev"]

# Prevent queue subscriptions on prod groups deny: ["> *.prod"]
} }
]


# Multi Tenancy using Accounts

In modern microservice architecture it is common to share infrastructure - such as NATS

- between services. Accounts are securely isolated communication contexts that allow
multi-tenancy in a NATS deployment. They allow users to bifurcate technology from
business driven use cases, where data silos are created by design, not software
limitations. Furthermore, they facilitate the controlled exchange of information between
those data silos/Tenants/Accounts.

Accounts expand on the authorization foundation. With traditional authorization, all
clients can publish and subscribe to anything unless explicitly congured otherwise. To
protect clients and information, you have to carve the subject space and permission
clients carefully.
Accounts allow the grouping of clients, isolating them from clients in other accounts,
thus enabling multi-tenancy in the server. With accounts, the subject space is not globally
shared, greatly simplifying the messaging environment. Instead of devising complicated
subject name carving patterns, clients can use short subjects without explicit
authorization rules. System Events are an example of this isolation at work.
Accounts conguration is done in accounts map. The contents of an account entry
includes:
**Property Description**
users a list of user conguration maps
exports a list of export maps
imports a list of import maps

## Multi Tenancy using Accounts

## Accounts


The accounts list is a map, where the keys on the map are an account name.

```
In the most straightforward conguration above you have an account named A
which has a single user identied by the username a and the password a , and an
account named B with a user identied by the username b and the password b
.
These two accounts are isolated from each other. Messages published by users in
A are not visible to users in B.
The user conguration map is the same as any other NATS user conguration map.
You can use:
```
```
While the name account implies one or more users, it is much simpler and
enlightening to think of one account as a messaging container for one application.
Users in the account are simply the minimum number of services that must work
together to provide some functionality. In simpler terms, more accounts with few
(even one) clients is a better design topology than a large account with many users
with complex authorization conguration.
```
Messaging exchange between different accounts is enabled by exporting streams and
services from one account and importing them into another. Each account controls what

```
accounts: { A: {
users: [ {user: a, password: a}
] },
B: { users: [
{user: b, password: b} ]
},}
```
```
username/password
nkeys
and add permissions
```
### Exporting and Importing


is exported and imported.

```
The term to and should not be confused with a JetStrstream in the context of import and expoream stream (unfort account congurtunate collision of terms asation does not refer
the impormessages't/export between accounts predates JetStream), it is just a 'stream of (Core NATS)
```
The exports conguration list enable you to dene the services and streams that
others can import. Exported services and streams are expressed as an Export
conguration map. The imports conguration lists the services and streams that an
Account imports. Imported services and streams are expressed as an Import
conguration map.

The export conguration map binds a subject for use as a service or stream and
optionally denes specic accounts that can import the stream or service. Here are the
supported conguration properties:

**Streams** able to make requests fr are messages yom your applications but will be able tour application publishes. Importing applications wono consume messages't be (^)
you generate.
**Services** accounts t are messages yo make requests that arour application can consume and act on, enabling othere fullled by your account.
**Property Description**
stream A subject or subject with wildcaraccount will publish. (exclusive of ds that theservice^ )
service A subject or subject with wildcaraccount will subscribe to. (exclusive of ds that thestream^ )
accounts A list of account names that can imporspecied, the service or stream is public and any account can import the stream or service. If not^ t it.
response_type Indicates if a ra single or a esponse tstreamo a of messages. Pservice request consists ofossible values
**Export Configuration Map**


Here are some example exports:

Here's what A is exporting:

An import enables an account to consume streams published by another account or
make requests to services implemented by another account. All imports require a
corresponding export on the exporting account. Accounts cannot do self-imports.

```
Property Description
are: single or stream. (Default value is singleton)
```
```
accounts: { A: {
users: [ {user: a, password: a}
] exports: [
{stream: puba.>} {service: pubq.>}
{stream: b.>, accounts: [B]} {service: q.b, accounts: [B]}
] }
...}
```
```
a public stream on the wildcard subject puba.>
a public service on the wildcard subject pubq.>
a stream to account B on the wildcard subject b.>
a service to account B on the subject q.b
```
```
Property Description
stream Stream import source conguration. (exclusive of service)
service Service import source conguration (exclusive of stream)
```
**Import Configuration Map**


The prefix and to options are optional and allow you to remap the subject that is
used locally to receive stream messages from or publish service requests to. This way
the importing account does not depend on naming conventions picked by another.
Currently, a service import can not make use of wildcards, which is why the import
subject can be rewritten. A stream import may make use of wildcards. To retain
information contained in the subject, it can thus only be prex ed with prefix...
**Source Conguration Map**
The source conguration map describes an export from a remote account by specifying
the account and subject of the export being imported. This map is embedded in the
import conguration map:

```
Property Description
prefix A local subject primported stream. (applicable tex mapping for theo stream^ )
```
```
Property Description
account Account name owning the export.
subject The subject under which the stris made accessible to the impoream or serting accountvice^
```
**Import/Export Example**


Account B imports:

Account C imports the public service and stream from A , but also:

```
accounts: { A: {
users: [ {user: a, password: a}
] exports: [
{stream: puba.>} {service: pubq.>}
{stream: b.>, accounts: [B]} {service: q.b, accounts: [B]}
] },
B: { users: [
{user: b, password: b} ]
imports: [ {stream: {account: A, subject: b.>}}
{service: {account: A, subject: q.b}} ]
} C: {
users: [ {user: c, password: c}
] imports: [
{stream: {account: A, subject: puba.>}, prefix: from_a} {service: {account: A, subject: pubq.C}, to: Q}
] }
}
```
```
the private stream from A that only B can receive on b.>
the private service from A that only B can send requests on q.b
```
```
remaps the messages will hapuba.>ve their original subjects pr stream to be locally available under ex ed by from_afrom_a.puba.>.. The
remaps the to publish to pubq.CQ locally. service to be locally available under Q. Account C only needs
```

It is important to reiterate that:

Clients connecting without authentication can be associated with a particular user within
an account.

The above example shows how clients without authentication can be associated with the
user a within account A.

```
Please note that the no_auth_user will not work with nkeys. The user referenced
can also be part of the authorization block.
```
```
stream puba.> from A is visible to all external accounts that imports the stream.
service the full subject of wherpubq.> from e to send the rA is available to all external accounts so long as theequest. Typically an account will export ay know
wildcard service but then coorrequests will be answered. On our example, account dinate with a client account on specic subjects wherC access the service on e
pubq.C (but has mapped it for simplicity to Q ).
stream b.> is private, only account B can receive messages from the stream.
service q.b is private; only account B can send requests to the service.
When However, the server will remap C publishes a request to QQ to , local pubq.CC and forward the requests t clients will see Q messages.o account
A.
```
```
accounts: { A: {
users: [ {user: a, password: a}
] },
B: { users: [
{user: b, password: b} ]
}}
no_auth_user: a
```
### No Auth User


```
Despite no_auth_user being set, clients still need to communicate that they will not
be using credentials. The authentication timeout applies to this process as well. When
your connection is slow, you may run into this timeout and the resulting
Authentication Timeout error, despite not providing credentials.
```
It is possible to import/export messages stored in JetStream streams between accounts.
While it is possible to allow a client application in one account to access a stream
located in another account, in most use cases people want a setup where a stream in one
account is mirrored or sourced in another account (and the applications in that other
account simply use that mirrored/sourced stream in their account), as this is a more
'locked-down' way to share messages in streams between accounts, compared to letting
the client applications directly use a stream in another account.
There are two resources documenting and giving examples of how to do this:
Cross account JetStrdo this using simple static security (as is pream sourcing explains and has a walkthrobably the best one tough example of how to start with) and o
Connect Strthe 'operator' JWT-based security mode of opereams Cross Accounts explains how tation.o do the same thing but when using

```
Multi-tenancy and resource management
```
**Exporting and importing JetStream streams between accounts**

## See Also


# OCSP Stapling

Supported since NATS Server version 2.3
OCSP Stapling is honored by default for certicates that have the status_request Must-
Staple ag.
When a certicate is congured with OCSP Must-Staple, the NATS Server will fetch
staples from the congured OCSP responder URL that is present in a certicate. For
example, given a certicate with the following conguration:

The NATS server will make a request to the OCSP responder to fetch a new staple which
will then be presented to any TLS connection that is accepted by the server during the
TLS handshake.
OCSP Stapling can be explicitly enabled or disabled in the NATS Server by setting the
following ag in the NATS conguration le at the top-level:

**Note** : When OCSP Stapling is disabled, the NATS Server will not request staples even if
the certicate has the Must-Staple ag.

By default, the NATS Server will be running in OCSP auto mode. In this mode the server
will only fetch staples when the Must-Staple ag is congured in the certicate.

```
[ ext_ca ]...
authorityInfoAccess = OCSP;URI:http://ocsp.example.net:80tlsfeature = status_request
...
```
```
ocsp: false
```
## Advanced Configuration


There are other OCSP modes that control the behavior as to whether OCSP should be
enforced and the server should shutdown if the certicate runs with a revoked staple:

For example, in the following OCSP conguration, the mode is set to must. This means
that staples will be fetched only for certicates that have the Must-Staple ag enabled as
well, but in case of revocation the server will shutdown rather than run with a revoked
staple.
In this conguration, the url will also override the OCSP responder URL that may have
been congured in the certicate.

If staples are always required, regardless of the conguration of the certicate, you can
enforce the behavior as follows:

```
Mode Description Server shutdownswhen revoked^
auto Enables OCSP Stapling when the cerhas the must staple/status_request agticate^ No
must Enables OCSP Staping when the cerhas the must staple/status_request agticate^ Yes
always Enables OCSP Stapling for all certicates Yes
never Disables OCSP Stapling eag is present (same as ven if must stapleocsp: false)^ No
```
```
ocsp { mode: must
url: "http://ocsp.example.net"}
```
```
ocsp { mode: always
url: "http://ocsp.example.net"}
```
## Caching of Staples


When a store_dir is congured in the NATS Server, the directory will be used to cache
staples on disk to allow the server to resume in case of restarts without having to make
another request to the OCSP responder if the staple is still valid.

If JetStream is enabled, then the same store_dir will be reused and disk caching will
be automatically enabled.

```
ocsp: true
store_dir: "/path/to/store/dir"
tls { cert_file: "configs/certs/ocsp/server-status-request-url.pem"
key_file: "configs/certs/ocsp/server-status-request-url-key.pem" ca_file: "configs/certs/ocsp/ca-cert.pem"
timeout: 5}
```

# Auth Callout

As of NATS v2.10.0
Auth Callout is an opt-in extension for delegating client authentication and authorization
to an application-dened NATS service.
The motivation for this extension is to support applications using an alternate identity
and access management (IAM) backend as the source of truth for managing
users/applications/machines credentials and permissions. This could be services that
implement standard protocols such as LDAP, SAML, and OAuth, an ad-hoc database, or
even a le on disk.

Both centralized and decentralized authentication models are supported with slightly
different considerations and semantics.
There are three phases to leveraging auth callout:

```
Auth Callout
```
```
service implementation
migration considerations
```

```
Note, the setup and congurdeploying a service could cause issues for existing systems.ation is deliberately last since enabling the conguration before
```
Centralized auth refers to all authentication and authorization mechanisms that are
server cong le-based.

Refer to the end-to-end example to get oriented with a basic service implementation.
There are three key data structures:

```
Language support for these structures currently exists for Go in the nats-io/jwt package.
```
In this context, migration refers to the considerations and steps required to enable auth
callout for an existing system without causing interruption.
In the centralized model, existing users dened in the cong le will be ignored. The auth
service will need to handle authenticating all users as well as assigning the target
account and permissions. This includes the system account user(s) and an implicit "no
auth" user.

```
setup and conguration
```
```
authorization request claims
authorization response claims
user claims
```
## Centralized Auth

### Service implementation

### Migration considerations


As a result, prior to enabling auth callout, existing users and permissions must be ported
to the target backend. Once the service is deployed, the auth_callout conguration
can be enabled at which point client authentication will be delegated to the auth service.
Assuming the credentials are the same, clients should not experience interruption on
reconnect.

For centralized auth callout, conguration is declared in the auth_callout block under
the top-level authorization block.

The available properties in the auth_callout block include:

To generate the account issuer NKey, the nsc tool can be used.

```
authorization { auth_callout {
... }
}
```
```
Property Description
issuer The public kused for signing authorization paey of the designated NKyloads.ey^
auth_users The list of user names under that are designated auth callout users.account^
account The account containing the users that arauth callout users. Defaults to the global account (e designated$G^ ).
xkey Optional. (x25519) used for encrThe public key of a designated XKypting authorization paey^ yloads.
```
```
$ nsc generate nkey --accountSAANDLKMXL6CUS3CP52WIXBEDN6YJ545GDKC65U5JZPPV6WH6ESWUA6YAI
ABJHLOVMPA4CI6R5KLNGOB4GSLNIY7IOUPAJC4YFNDLQVIOBYQGUWVLA
```
### Setup and configuration


```
☝ Be sure to generate your own keypair! Don't use this in production.
```
This minimum conguration would use the implicit default account $G.

If an existing system using multiple accounts is being migrated to auth callout, then the
existing accounts conguration should remain with the users property removed
(since it will no longer be used after being ported).
For new setups, it is recommended to use explicit accounts, such as the following
conguration having the AUTH account for auth callout, APP (could be more) for
application account (instead of relying on the $G account), and SYS for the system
account.

```
authorization { users: [ { user: auth, password: auth } ]
auth_callout { issuer: ABJHLOVMPA4CI6R5KLNGOB4GSLNIY7IOUPAJC4YFNDLQVIOBYQGUWVLA
auth_users: [ auth ] }
}
```
```
accounts { AUTH: {
users: [ { user: auth, password: auth } ] }
APP: {} SYS: {}
}system_account: SYS
authorization { auth_callout {
issuer: ABJHLOVMPA4CI6R5KLNGOB4GSLNIY7IOUPAJC4YFNDLQVIOBYQGUWVLA auth_users: [ auth ]
account: AUTH }
}
```
**Multiple accounts**


The xkey property enables encrypting the request payloads. This is recommended as a
security best practice, but not required.
To generate an XKey, nsc can be used again.

```
☝ Again, don't use this and be sure to generate your own and keep the seed secret!
```
Incorporating the xkey, we have the following cong:

Coming soon!

```
$ nsc generate nkey --curveSXANPB47UINQR7EXT3BRP26A4LY2CMCDLTY2KX6BU3EGK2VZYREJ4IJRCE
XAMHJVPKHHPYZQQM2IVWXKJH36KDDZZMSJ32QKSQBUODFX4I4HARO4GL
```
```
accounts { AUTH: {
users: [ { user: auth, password: auth } ] }
APP: {} SYS: {}
}system_account: SYS
authorization { auth_callout {
issuer: ABJHLOVMPA4CI6R5KLNGOB4GSLNIY7IOUPAJC4YFNDLQVIOBYQGUWVLA auth_users: [ auth ]
account: AUTH xkey: XAMHJVPKHHPYZQQM2IVWXKJH36KDDZZMSJ32QKSQBUODFX4I4HARO4GL
}}
```
**Encryption**

## Decentralized Auth


When encryption is enabled, the server will generate a one-time use XKey keypair per
client connection/reconnect. The public key is included in the authorization request
claims which enables the auth callout service to encrypt the authorization response
payload when sending it back to the NATS server.

```
The one-time use kafter the rst response was reypair prevents replay attacks since the public keceived by the server or the timeout was rey will be threached.own away
```
Once the authorization request is prepared, it is encoded and encrypted using the
congured xkey public key. Once encrypted, the message is published for the auth
service to receive.
The auth service is expected to have the private key to decrypt the authorization request
before using the claims data. When preparing the response, the server-provided one-time
public xkey will be used to encrypt the response before sending back to the server.

The claims is a standard JWT structure with a nested object named nats containing the
following top-level elds:

```
used in the authorization rserver_id - An object describing the Nesponse. ATS server, include the id eld needed to be^
subjectuser_nkey of the authorization r - A user public NKesponse.ey generated by the NATS server which is used as the
client_info - An object describing the client attempting to connect.
```
## Reference

### Encryption

### Schema

**Authorization request claims**


message.connect_opts - An object containing the data sent by client in the CONNECT^
client_tls - An object containing any client certicates, if applicable.
Full JSON schema


{ "$schema": "https://json-schema.org/draft/2020-12/schema",
"$id": "authorization-request-claims", "properties": {
"aud": { "type": "string"
}, "exp": {
"type": "integer" },
"jti": { "type": "string"
}, "iat": {
"type": "integer" },
"iss": { "type": "string"
}, "name": {
"type": "string" },
"nbf": { "type": "integer"
}, "sub": {
"type": "string" },
"nats": { "properties": {
"server_id": { "properties": {
"name": { "type": "string"
}, "host": {
"type": "string" },
"id": { "type": "string"
}, "version": {
"type": "string" },
"cluster": { "type": "string"
},"tags": {


tags : { "items": {
"type": "string" },
"type": "array" },
"xkey": { "type": "string"
} },
"additionalProperties": false, "type": "object",
"required": [ "name",
"host", "id"
] },
"user_nkey": { "type": "string"
}, "client_info": {
"properties": { "host": {
"type": "string" },
"id": { "type": "integer"
}, "user": {
"type": "string" },
"name": { "type": "string"
}, "tags": {
"items": { "type": "string"
}, "type": "array"
}, "name_tag": {
"type": "string" },
"kind": { "type": "string"
}, "type": {
"type": "string"


}, "mqtt_id": {
"type": "string" },
"nonce": { "type": "string"
} },
"additionalProperties": false, "type": "object"
}, "connect_opts": {
"properties": { "jwt": {
"type": "string" },
"nkey": { "type": "string"
}, "sig": {
"type": "string" },
"auth_token": { "type": "string"
}, "user": {
"type": "string" },
"pass": { "type": "string"
}, "name": {
"type": "string" },
"lang": { "type": "string"
}, "version": {
"type": "string" },
"protocol": { "type": "integer"
} },
"additionalProperties": false, "type": "object",
"required": [ "protocol"


] },
"client_tls": { "properties": {
"version": { "type": "string"
}, "cipher": {
"type": "string" },
"certs": { "items": {
"type": "string" },
"type": "array" },
"verified_chains": { "items": {
"items": { "type": "string"
}, "type": "array"
}, "type": "array"
} },
"additionalProperties": false, "type": "object"
}, "request_nonce": {
"type": "string" },
"tags": { "items": {
"type": "string" },
"type": "array" },
"type": { "type": "string"
}, "version": {
"type": "integer" }
}, "additionalProperties": false,
"type": "object", "required": [


The claims is a standard JWT structure with a nested object named nats containing the
following top-level elds:

```
"server_id", "user_nkey",
"client_info", "connect_opts"
] }
}, "additionalProperties": false,
"type": "object", "required": [
"nats" ]
}
```
```
duration of the client connection.jwt - The encoded user claims JWT which will be used by the NATS server for the^
will be included log output.error - An error message sent back to the NATS server if authorization failed. This^
claim was issued bissuer_account - The public Nky a signing key.ey of the issuing account. If set, this indicates the^
Full JSON schema
```
**Authorization response claims**


{ "$schema": "https://json-schema.org/draft/2020-12/schema",
"$id": "https://github.com/nats-io/jwt/v2/authorization-respon "properties": {
"aud": { "type": "string"
}, "exp": {
"type": "integer" },
"jti": { "type": "string"
}, "iat": {
"type": "integer" },
"iss": { "type": "string"
}, "name": {
"type": "string" },
"nbf": { "type": "integer"
}, "sub": {
"type": "string" },
"nats": { "properties": {
"jwt": { "type": "string"
}, "error": {
"type": "string" },
"issuer_account": { "type": "string"
}, "tags": {
"items": { "type": "string"
}, "type": "array"
}, "type": {
"type": "string"},


The claims is a standard JWT structure with a nested object named nats containing the
following, notable, top-level elds:

```
}, "version": {
"type": "integer" }
}, "additionalProperties": false,
"type": "object" }
}, "additionalProperties": false,
"type": "object", "required": [
"nats" ]
}
```
```
claim was issued bissuer_account - The public Nky a signing key.ey of the issuing account. If set, this indicates the^
Full JSON schema
```
**User claims**


{ "$schema": "https://json-schema.org/draft/2020-12/schema",
"$id": "https://github.com/nats-io/jwt/v2/user-claims", "properties": {
"aud": { "type": "string"
}, "exp": {
"type": "integer" },
"jti": { "type": "string"
}, "iat": {
"type": "integer" },
"iss": { "type": "string"
}, "name": {
"type": "string" },
"nbf": { "type": "integer"
}, "sub": {
"type": "string" },
"nats": { "properties": {
"pub": { "properties": {
"allow": { "items": {
"type": "string" },
"type": "array" },
"deny": { "items": {
"type": "string" },
"type": "array" }
}, "additionalProperties": false,
"type": "object"},


}, "sub": {
"properties": { "allow": {
"items": { "type": "string"
}, "type": "array"
}, "deny": {
"items": { "type": "string"
}, "type": "array"
} },
"additionalProperties": false, "type": "object"
}, "resp": {
"properties": { "max": {
"type": "integer" },
"ttl": { "type": "integer"
} },
"additionalProperties": false, "type": "object",
"required": [ "max",
"ttl" ]
}, "src": {
"items": { "type": "string"
}, "type": "array"
}, "times": {
"items": { "properties": {
"start": { "type": "string"
}, "end": {
"type": "string"


} },
"additionalProperties": false, "type": "object"
}, "type": "array"
}, "times_location": {
"type": "string" },
"subs": { "type": "integer"
}, "data": {
"type": "integer" },
"payload": { "type": "integer"
}, "bearer_token": {
"type": "boolean" },
"allowed_connection_types": { "items": {
"type": "string" },
"type": "array" },
"issuer_account": { "type": "string"
}, "tags": {
"items": { "type": "string"
}, "type": "array"
}, "type": {
"type": "string" },
"version": { "type": "integer"
} },
"additionalProperties": false, "type": "object"
} },


"additionalProperties": false, "type": "object"
}


# Logging

The NATS server provides various logging options that you can set via the command line
or the conguration le.

The following logging operations are supported:

The -DV ag enables trace and debug for the server.

If -T false then log entries are not timestamped. Default is true.

```
-l, --log FILE File to redirect log output.-T, --logtime Timestamp log entries (default is true).
-s, --syslog Enable syslog as log method.-r, --remote_syslog Syslog server address.
-D, --debug Enable debugging output.-V, --trace Trace the raw protocol.
-VV Verbose trace (traces system account as w-DV Debug and Trace.
-DVV Debug and verbose trace (traces system a
```
```
nats-server -DV -m 8222 -user foo -pass bar
```
```
nats-server -DV -m 8222 -l nats.log
```
## Configuring Logging

## Command Line Options

**Debug and trace**

**Log file redirect**

**Timestamp**


You can congure syslog with UDP:

or syslog:

For example:

All of these settings are available in the conguration le as well.

Introduced in NATS Server v2.1.4, NATS allows for auto-rotation of log les when the size
is greater than the congured limit set in logfile_size_limit. The backup les will
have the same name as the original log le with the sux .yyyy.mm.dd.hh.mm.ss.micros.
You can also use NATS-included mechanisms with logrotate, a simple standard Linux
utility to rotate logs available on most distributions like Debian, Ubuntu, RedHat (CentOS),
etc., to make log rotation simple.
For example, you could congure logrotate with:

```
nats-server -r udp://localhost:514
```
```
nats-server -r syslog://<hostname>:<port>
```
```
syslog://logs.papertrailapp.com:26900
```
```
debug: falsetrace: true
logtime: falselogfile_size_limit: 1GB
log_file: "/tmp/nats-server.log"
```
**Syslog**

### Using the Configuration File

### Log Rotation


The rst line species the location that the subsequent lines will apply to.
The rest of the le species that the logs will rotate daily ("daily" option) and that 30 older
copies will be preserved ("rotate" option). Other options are described in logrorate
documentation.
The "postrotate" section tells NATS server to reload the log les once the rotation is
complete. The command `kill -SIGUSR1cat /var/run/nats-server.pid``` does not kill
the NATS server process, but instead sends it a signal causing it to reload its log les.
This will cause new requests to be logged to the refreshed log le.
The /var/run/nats-server.pid le is where NATS server stores the master process's
pid.

```
/path/to/nats-server.log { daily
rotate 30 compress
missingok notifempty
postrotate kill -SIGUSR1 `cat /var/run/nats-server.pid`
endscript}
```
```
The NATS server, in verbose mode, will log the rdoes not indicate the subscription is gone, only that the message was receipt of UNSUB messages, but thiseceived. The
removal has takDELSUB message in the log can be used ten place. o determine when the actual subscription^
```
## Some Logging Notes


# Enabling Monitoring

To monitor the NATS messaging system, nats-server provides a lightweight HTTP
server on a dedicated monitoring port. The monitoring server provides several endpoints,
providing statistics and other information.
The NATS monitoring endpoints support JSONP and CORS, making it easy to create
single page monitoring web applications.

```
Warning: nats-server does not have authentication/authorization for the
monitoring endpoint. When you plan to open your nats-server to the internet make
sure to not expose the monitoring port as well. By default monitoring binds to every
interface 0.0.0.0 so consider setting monitoring to localhost or have
appropriate rewall rules.
```
To enable the monitoring server, start the NATS server with the monitoring ag -m and
the monitoring port, or turn it on in the conguration le.

Example:

```
-m, --http_port PORT HTTP PORT for monitoring-ms,--https_port PORT Use HTTPS PORT for monitoring
```
```
nats-server -m 8222
[4528] 2019/06/01 20:09:58.572939 [INF] Starting nats-server version 2.0.[4528] 2019/06/01 20:09:58.573007 [INF] Starting http monitor on port 822
[4528] 2019/06/01 20:09:58.573071 [INF] Listening for client connections [4528] 2019/06/01 20:09:58.573090 [INF] nats-server is ready</td>
```
## NATS Server Monitoring

## Enabling monitoring from the command line


To test, run nats-server -m 8222, then go to [http://localhost:8222/](http://localhost:8222/)

You can also enable monitoring using the conguration le as follows:

Binding to localhost as well:

For example, to monitor this server locally, the endpoint would be
[http://localhost:8222/varz.](http://localhost:8222/varz.) It reports various general statistics.

In addition to writing custom monitoring tools, you can monitor nats-server in
Prometheus. The Prometheus NATS Exporter allows you to congure the metrics you
want to observe and store in Prometheus. There's a sample Grafana dashboard that you
can use to visualize the server metrics.

```
http_port: 8222
```
```
http: localhost:8222
```
### Enable monitoring from the configuration file

## Monitoring Tools


# MQTT

Supported since NATS Server version 2.2
NATS follows as closely as possible to the MQTT v3.1.1 specication. Refer to the MQTT
implementation overview in the nats-server repo.

MQTT support in NATS is intended to be an enabling technology allowing users to
leverage existing investments in their IoT deployments. Updating software on the edge or
endpoints can be onerous and risky, especially when embedded applications are involved.
In greeneld IoT deployments, when possible, we prefer NATS extended out to endpoints
and devices for a few reasons. There are signicant advantages with security and
observability when using a single technology end to end. Compared to MQTT, NATS is
nearly as lightweight in terms of protocol bandwidth and maintainer supported clients
eciently utilize resources so we consider NATS to be a good choice to use end to end,
including use on resource constrained devices.
In existing MQTT deployments or in situations when endpoints can only support MQTT,
using a NATS server as a drop-in MQTT server replacement to securely connect to a
remote NATS cluster or supercluster is compelling. You can keep your existing IoT
investment and use NATS for secure, resilient, and scalable access to your streams and
services.

For an MQTT client to connect to the NATS server, the user's account must be JetStream
enabled. This is because persistence is needed for the sessions and retained messages
since even retained messages of QoS 0 are persisted.

## When to Use MQTT

## JetStream Requirements


MQTT Topics are similar to NATS Subjects, but have distinctive differences.
MQTT topic uses " / " as a level separator. For instance foo/bar would translate to
NATS subject foo.bar. But in MQTT, /foo/bar/ is a valid subject, which, if simply
translated, would become .foo.bar., which is NOT a valid NATS Subject.
NATS Server will convert an MQTT topic following those rules:

Note: Prior to NATS Server v2.10.0, the character. was not supported. At version
v2.10.0 and above, the character. will be translated to //.
As indicated above, if an MQTT topic contains the character (or. prior to v2.10.0),
NATS will reject it, causing the connection to be closed for published messages, and
returning a failure code in the SUBACK packet for a subscriptions.

```
MQTT character NATS character(s) Topic (MQTT) Subject (NATS)
/ between two levels. foo/bar foo.bar
/ as rst level /. /foo/bar /.foo.bar
/ as last level ./ foo/bar/ foo.bar./
/ next to another ./ foo//bar foo./.bar
/ next to another /. //foo/bar /./.foo.bar
```
. // (see note below) foo.bar foo//bar
    Not Supported foo bar Not Supported

## MQTT Topics and NATS Subjects

### MQTT Wildcards


As in NATS, MQTT wildcards represent either multi or single levels. As in NATS, they are
allowed only for subscriptions, not for published messages.

The wildcard # matches any number of levels within a topic, which means that a
subscription on foo/# would receive messages on foo/bar, or foo/bar/baz, but
also on foo. This is not the case in NATS where a subscription on foo.> can receive
messages on foo/bar or foo/bar/baz, but not on foo. To solve this, NATS Server
will create two subscriptions, one on foo.> and one on foo. If the MQTT subscription
is simply on # , then a single NATS subscription on > is enough.
The wildcard + matches a single level, which means foo/+ can receive message on
foo/bar or foo/baz, but not on foo/bar/baz nor foo. This is the same with NATS
subscriptions using the wildcard *. Therefore foo/+ would translate to foo.*.

When an MQTT client creates a subscription on a topic, the NATS server creates the
similar NATS subscription (with conversion from MQTT topic to NATS subject) so that the
interest is registered in the cluster and known to any NATS publishers.
That is, say an MQTT client connects to server "A" and creates a subscription of
foo/bar, server "A" creates a subscription on foo.bar, which interest is propagated as
any other NATS subscription. A publisher connecting anywhere in the cluster and
publishing on foo.bar would cause server "A" to deliver a QoS 0 message to the MQTT
subscription.
This works the same way for MQTT publishers. When the server receives an MQTT
publish message, it is converted to the NATS subject and published, which means that
any matching NATS subscription will receive the MQTT message.

```
MQTT Wildcard NATS Wildcard
# >
+ *
```
## Communication Between MQTT and NATS


If the MQTT subscription is QoS1 and an MQTT publisher publishes an MQTT QoS1
message on the same or any other server in the cluster, the message will be persisted in
the cluster and routed and delivered as QoS 1 to the MQTT subscription.

When the server delivers a QoS 1 message to a QoS 1 subscription, it will keep the
message until it receives the PUBACK for the corresponding packet identier. If it does
not receive it within the "ack_wait" interval, that message will be resent.

This is the amount of QoS 1 messages the server can send to a subscription without
receiving any PUBACK for those messages. The maximum value is 65535.
The total of subscriptions' max_ack_pending on a given session cannot exceed 65535.
Attempting to create a subscription that would bring the total above the limit would result
in the server returning a failure code in the SUBACK for this subscription.
Due to how the NATS server handles the MQTT " # " wildcard, each subscription ending
with " # " will use 2 times the max_ack_pending value.

NATS Server will persist all sessions, even if they are created with the "clean session" ag,
meaning that sessions only last for the duration of the network connection between the
client and the server.
A session is identied by a client identier. If two connections try to use the same client
identier, the server, per specication, will close the existing connection and accept the
new one.

## QoS 1 Redeliveries

## Max Ack Pending

## Sessions


If the user incorrectly starts two applications that use the same client identier, this
would result in a very quick apping if the MQTT client has a reconnect feature and
quickly reconnects.
To prevent this, the NATS server will accept the new session and will delay the closing of
the old connection to reduce the apping rate.
Detection of the concurrent use of sessions also works in cluster mode.

When a server receives an MQTT publish packet with the RETAIN ag set (regardless of
its QoS), it stores the application message and its QoS, so that it can be delivered to
future subscribers whose subscriptions match its topic name.
When a new subscription is established, the last retained message, if any, on each
matching topic name will be sent to the subscriber.
A PUBLISH Packet with a RETAIN ag set to 1 and a payload containing zero bytes will be
processed as normal and sent to clients with a subscription matching the topic name.
Additionally any existing retained message with the same topic name will be removed
and any future subscribers for the topic will not receive a retained message.

NATS supports MQTT in a NATS cluster. The replication factor is automatically set based
on the size of the cluster.

If a client is connected to a server "A" in the cluster and another client connects to a
server "B" and uses the same client identier, server "A" will close its client connection
upon discovering the use of an active client identier.

## Retained Messages

## Clustering

### Connections with Same Client ID


Users should avoid this situation as this is not as easy and immediate as if the two
applications are connected to the same server.
There may be cases where the server will reject the new connection if there is no safe
way to close the existing connection, such as when it is in the middle of processing some
MQTT packets.

Retained messages are stored in the cluster and available to any server in the cluster.
However, this is not immediate and if a producer connects to a server and produces a
retained message and another connection connects to another server and starts a
matching subscription, it may not receive the retained message if the server it connects
to has not yet been made aware of this retained message.
In other words, retained messages in clustering mode is best-effort, and applications that
rely on the presence of a retained message should connect on the server that produced
them.

```
NATS messages published tmessages. o MQTT subscriptions are always delivered as QoS 0
MQTT published messages on tcause the connection to be closed. Propic names containing "````" or "esence of those characters in MQ.\" characters willTT
subscriptions will result in error code in the SUBACK packet.
MQTT wildcard # may cause the NATS server to create two subscriptions.
MQTT concurrthe existing one.ent sessions may result in the new connection to be evicted instead of
MQTT retained messages in clustering mode is best effort.
```
### Retained Messages

## Limitations

## See Also


Replace your MQTT Broker with NATS Server


# Configuration

MQTT Conguration Example
To enable MQTT support in the server, add an mqtt conguration block in the server's
conguration le like the following:


mqtt { # Specify a host and port to listen for websocket connections
# # listen: "host:port"
# It can also be configured with individual parameters, # namely host and port.
# # host: "hostname"
port: 1883
# TLS configuration. #
tls { cert_file: "/path/to/cert.pem"
key_file: "/path/to/key.pem"
# Root CA file #
# ca_file: "/path/to/ca.pem"
# If true, require and verify client certificates. #
# verify: true
# TLS handshake timeout in fractional seconds. #
# timeout: 2.0
# If true, require and verify client certificates and map certifi # values for authentication purposes.
# # verify_and_map: true
}
# If no user name is provided when an MQTT client connects, will defa # this user name in the authentication phase. If specified, this will
# override, for MQTT clients, any `no_auth_user` value defined in the # main configuration file.
# Note that this is not compatible with running the server in operato #
# no_auth_user: "my_username_for_apps_not_providing_credentials"
# See below to know what is the normal way of limiting MQTT clients # to specific users.
# If there are no users specified in the configuration, this simple a # block allows you to override the values that would be configured in
# equivalent block in the main section.#


# # authorization {
# # If this is specified, the client has to provide the same user # # and password to be able to connect.
# # username: "my_user_name" # # password: "my_password"
# # # If this is specified, the password field in the CONNECT packe
# # match this token. # # token: "my_token"
# # # This overrides the main's authorization timeout. For consiste
# # with the main's authorization configuration block, this is ex # # as a number of seconds.
# # timeout: 2.0 #}

# This is the amount of time after which a QoS 1 message sent to # a client is redelivered as a DUPLICATE if the server has not
# received the PUBACK packet on the original Packet Identifier. # The value has to be positive.
# Zero will cause the server to use the default value (30 seconds). # Note that changes to this option is applied only to new MQTT subscr
# # Expressed as a time duration, with "s", "m", "h" indicating seconds
# minutes and hours respectively. For instance "10s" for 10 seconds, # "1m" for 1 minute, etc...
# # ack_wait: "1m"

# This is the amount of QoS 1 messages the server can send to # a subscription without receiving any PUBACK for those messages.
# The valid range is [0..65535]. #
# The total of subscriptions' max_ack_pending on a given session cann # exceed 65535. Attempting to create a subscription that would bring
# the total above the limit would result in the server returning 0x80 # in the SUBACK for this subscription.
# Due to how the NATS Server handles the MQTT "#" wildcard, each # subscription ending with "#" will use 2 times the max_ack_pending v
# Note that changes to this option is applied only to new subscriptio #
# max_ack_pending: 100}


MQTT requires a server name to be set. Server names must be unique in a cluster or
super-cluster topology. The server name is set in the top-level section of the server
conguration. Here is an example:

In operator mode, all users need to provide a JWT in order to connect. In the standard
authentication procedure of this mode, NATS clients are required to sign a nonce sent
by the server using their private key (see JWTs and Privacy). MQTT clients cannot do that,
therefore, the JWT is used for authentication, removing the need of the seed. It means
that you need to pass the JWT token as the MQTT password and use any username
(except empty, since MQTT protocol requires a username to be set if a password is set).
The JWT has to have the Bearer boolean set to true, which can be done with nsc:

A new eld when conguring users allows you to restrict which type of connections are
allowed for a specic user.
Consider this conguration:

```
server_name: "my_mqtt_server"mqtt {
port: 1883 ...
}
```
```
nsc edit user --name U --account A --bearer
```
## Server name

## Authentication/Authorization of MQTT Users

### Operator mode

### Local mode


If an MQTT client were to connect and use the username foo and password foopwd, it
would be accepted. Now suppose that you would want an MQTT client to only be
accepted if it connected using the username bar and password barpwd, then you
would use the option allowed_connection_types to restrict which type of connections
can bind to this user.

The option allowed_connection_types (also can be named connection_types or
clients) as you can see is a list, and you can allow several types of clients. Suppose
you want the user bar to accept both standard NATS clients and MQTT clients, you
would congure the user like this:

The absence of allowed_connection_types means that all types of connections are
allowed (the default behavior).
The possible values are currently:

```
authorization { users [
{user: foo password: foopwd, permission: {...}} {user: bar password: barpwd, permission: {...}}
]}
```
```
authorization { users [
{user: foo password: foopwd, permission: {...}} {user: bar password: barpwd, permission: {...}, allowed_connection_ty
]}
```
```
authorization { users [
{user: foo password: foopwd, permission: {...}} {user: bar password: barpwd, permission: {...}, allowed_connection_ty
]}
```
```
STANDARD
WEBSOCKET
```

When an MQTT client creates a QoS 1 subscription, this translates to the creation of a
JetStream durable subscription. To receive messages for this durable, the NATS Server
creates a subscription with a subject such as $MQTT.sub. and sets it as the JetStream
durable's delivery subject.
Therefore, if you have set some permissions for the MQTT user, you need to allow
subscribe permissions on $MQTT.sub.>.
Here is an example of a basic conguration that sets some permissions to a user named
"mqtt". As you can see, the subscribe permission $MQTT.sub.> is added to allow this
client to create QoS 1 subscriptions.

```
LEAFNODE
MQTT
```
```
listen: 127.0.0.1:4222 jetstream: enabled
authorization { mqtt_perms = {
publish = ["baz"] subscribe = ["foo", "bar", "$MQTT.sub.>"]
} users = [
{user: mqtt, password: pass, permissions: $mqtt_perms, allowe ]
} mqtt {
listen: 127.0.0.1:1883 }
```
### Special permissions


# Configuring Subject Mapping

Supported since NATS Server version 2.2
Subject mapping is a very powerful feature of the NATS server, useful for canary
deployments, A/B testing, chaos testing, and migrating to a new subject namespace.

Subject mappings are dened and applied at the account level. If you are using static
account security you will need to edit the server conguration le, however if you are
using JWT Security (Operator Mode), then you need to use nsc or customer tools to edit
and push changes to you account.
NOTE: You can also use subject mapping as part of dening imports and exports
between accounts

In any of the static authentication modes the mappings are dened in the server
conguration le, any changes to mappings in the conguration le will take effect as
soon as a reload signal is sent to the server process (e.g. use
nats-server --signal reload).
The mappings stanza can occur at the top level to apply to the global account or be
scoped within a specic account.

## Configuring subject mapping

## Static authentication


When using the JWT authentication mode, the mappings are dened in the account's
JWT. Account JWTs can be created or modied either through the_ JWT API_ or using the
nsc CLI too. For more detailed information see nsc add mapping --help,
nsc delete mapping --help. Subject mapping changes take effect as soon as the
modied account JWT is pushed to the nats servers (i.e. nsc push).
Examples of using nsc to manage mappings:

```
mappings = {
# Simple direct mapping. Messages published to foo are mapped to bar. foo: bar
# remapping tokens can be done with $<N> representing token position. # In this example bar.a.b would be mapped to baz.b.a.
bar.*.*: baz.$2.$1
# You can scope mappings to a particular cluster foo.cluster.scoped : [
{ destination: bar.cluster.scoped, weight:100%, cluster: us-west-1 } ]
# Use weighted mapping for canary testing or A/B testing. Change dynam # at any time with a server reload.
myservice.request: [ { destination: myservice.request.v1, weight: 90% },
{ destination: myservice.request.v2, weight: 10% } ]
# A testing example of wildcard mapping balanced across two subjects. # 20% of the traffic is mapped to a service in QA coded to fail.
myservice.test.*: [ { destination: myservice.test.$1, weight: 80% },
{ destination: myservice.test.fail.$1, weight: 20% } ]
# A chaos testing trick that introduces 50% artificial message loss of # messages published to foo.loss
foo.loss.>: [ { destination: foo.loss.>, weight: 50% } ]}
```
### JWT authentication


The example of foo:bar is straightforward. All messages the server receives on subject
foo are remapped and can be received by clients subscribed to bar.

Wildcard tokens may be referenced via $<position>. For example, the rst wildcard
token is $1, the second is $2, etc. Referencing these tokens can allow for reordering.
With this mapping:

Messages that were originally published to bar.a.b are remapped in the server to
baz.b.a. Messages arriving at the server on bar.one.two would be mapped to
baz.two.one, and so forth.

Trac can be split by percentage from one subject to multiple subjects. Here's an
example for canary deployments, starting with version 1 of your service.

```
Add a new mapping: nsc add mapping --from "a" --to "b"
Modify an entrnsc add mapping --from "a" --to "b" --weight 50y, say to set a weight after the fact:
Add two entries frnsc add mapping --from "a" --to "c" --weight 50om one subject, set weights and execute multiple times:
Delete a mapping: nsc delete mapping --from "a"
```
```
bar.*.*: baz.$2.$1
```
## Simple Mapping

## Subject Token Reordering

## Weighted Mappings for A/B Testing or Canary

## Releases


Applications would make requests of a service at myservice.requests. The
responders doing the work of the server would subscribe to myservice.requests.v1.
Your conguration would look like this:

All requests to myservice.requests will go to version 1 of your service.
When version 2 comes along, you'll want to test it with a canary deployment. Version 2
would subscribe to myservice.requests.v2. Launch instances of your service (don't
forget about queue subscribers and load balancing).
Update the conguration le to redirect some portion of the requests made to
myservice.requests to version 2 of your service. In this case we'll use 2%.

You can reload the server at this point to make the changes with zero downtime. After
reloading, 2% of your requests will be serviced by the new version.
Once you've determined Version 2 stable switch 100% of the trac o ver and reload the
server with a new conguration.

Now shutdown the version 1 instances of your service.

```
myservice.requests: [ { destination: myservice.requests.v1, weight: 100% }
]
```
```
myservice.requests: [ { destination: myservice.requests.v1, weight: 98% },
{ destination: myservice.requests.v2, weight: 2% } ]
```
```
myservice.requests: [ { destination: myservice.requests.v2, weight: 100% }
]
```
## Traffic Shaping in Testing


Trac shaping is useful in testing. You might have a service that runs in QA that
simulates failure scenarios which could receive 20% of the trac t o test the service
requestor.

Alternatively, introduce loss into your system for chaos testing by mapping a percentage
of trac t o the same subject. In this drastic example, 50% of the trac published to
foo.loss.a would be articially dropped by the server.

You can both split and introduce loss for testing. Here, 90% of requests would go to your
service, 8% would go to a service simulating failure conditions, and the unaccounted for
2% would simulate message loss.

```
myservice.requests.*: [ { destination: myservice.requests.$1, weight: 80% },
{ destination: myservice.requests.fail.$1, weight: 20% } ]
```
```
foo.loss.>: [ { destination: foo.loss.>, weight: 50% } ]
```
```
myservice.requests: [ { destination: myservice.requests.v3, weight: 90% },
{ destination: myservice.requests.v3.fail, weight: 8% } # the remaining 2% is "lost"
]
```
## Artificial Loss


# System Events

NATS servers leverage Accounts support and generate events such as:

In addition the server supports a limited number of requests that can be used to query for
account connections, server stat summaries, and pinging servers in the cluster.
These events are enabled by conguring system_account and subscribing/requesting
using a system account user.
Accounts are used so that subscriptions from your applications, say > , do not receive
system events and vice versa. Using accounts requires either:

The system account publishes messages under well known subject patterns.
Server initiated events:

```
account connect/disconnect
authentication errors
server shutdown
server stat summary
```
```
Conguring authentication locally and listing one of the accounts in system_account
Or by using decentrTutorial. In this case alized authentication and authorization via system_account contains the account public kjwt as shown in this ey.
```
```
$SYS.ACCOUNT.<id>.CONNECT (client connects)
$SYS.ACCOUNT.<id>.DISCONNECT (client disconnects)
$SYS.ACCOUNT.<id>.SERVER.CONNS (connections for an account changed)
```
## Available Events and Services

## System Account


In addition other tools with system account privileges, can initiate requests (Examples
can be found here):

Monitoring endpoints as listed in the table below are accessible as system services using
the following subject pattern:

```
$SYS.SERVER.<id>.CLIENT.AUTH.ERR (authentication error)
$SYS.ACCOUNT.<id>.LEAFNODE.CONNECT (leaf node connects)
$SYS.ACCOUNT.<id>.LEAFNODE.DISCONNECT (leaf node disconnects)
$SYS.SERVER.<id>.STATSZ (stats summary)
```
```
$SYS.REQ.SERVER.<id>.STATSZ (request server stat summary)
$SYS.REQ.SERVER.PING (discover servers - will return multiple messages)
```
```
corresponding t$SYS.REQ.SERVER.<id>.<endpoint-name>o endpoint name.) (request server monitoring endpoint^
monitoring endpoint corr$SYS.REQ.SERVER.PING.<endpoint-name>esponding to endpoint name - will r (from all server, request sereturn multiple messages)ver^
Endpoint Endpoint Name
General Server Information VARZ
Connections CONNZ
Routing ROUTEZ
Gateways GATEWAYZ
Leaf Nodes LEAFZ
Subscription Routing SUBSZ
JetStream JSZ
Accounts ACCOUNTZ
```

Servers like nats-account-server publish system account messages when a claim is
updated, the nats-server listens for them, and updates its account information
accordingly:

With these few messages you can build useful monitoring tools:

To make use of System events, just using accounts, your conguration can look like this:

```
account specic monit"$SYS.REQ.ACCOUNT.<account-id>.<endpoint-name>oring endpoint corresponding to account id and endpoint name(from all server, request^
```
- will return multiple messages)
**Endpoint Endpoint Name**
Connections CONNZ
Leaf Nodes LEAFZ
Subscription Routing SUBSZ
JetStream JSZ
Account INFO

```
$SYS.ACCOUNT.<id>.CLAIMS.UPDATE
```
```
health/load of your servers
client connects/disconnects
account connections
authentication errors
```
## Local Configuration


Please note that applications now have to authenticate such that a connection can be
associated with an account. In this example username and password were chosen for
simplicity of the demonstration. Subscribe to all system events like this
nats sub -s nats://admin:changeit@localhost:4222 ">" and observe what
happens when you do something like
nats pub -s "nats://a:a@localhost:4222" foo bar. Examples on how to use
system services can be found here.

```
accounts: { USERS: {
users: [ {user: a, password: a}
] },
SYS: { users: [
{user: admin, password: changeit} ]
},}
system_account: SYS
```

# System Events &

# Decentralized JWT Tutorial

To enable and access system events, you'll have to:

Let's create an operator, system account and system account user:

Add the system account

Add a system account user

```
Create an Operator, Account and User
Run a NATS Account Server (or Memory Resolver)
```
```
nsc add operator -n SAOP
Generated operator key - private key stored "~/.nkeys/SAOP/SAOP.nk"Success! - added operator "SAOP"
```
```
nsc add account -n SYS
Generated account key - private key stored "~/.nkeys/SAOP/accounts/SYS/SYSuccess! - added account "SYS"
```
```
nsc add user -n SYSU
```
## Enabling System Events with Decentralized

## Authentication/Authorization

## Create an Operator, Account, User


By default, the operator JWT can be found in
~/.nsc/nats/<operator_name>/<operator.name>.jwt.

To vend the credentials to the nats-server, we'll use a nats-account-server. Let's start a
nats-account-server to serve the JWT credentials:

The server will by default vend JWT congurations on the an endpoint at:
http(s)://<server_url>/jwt/v1/accounts/.

The server conguration will need:

The only thing we don't have handy is the public key for the system account. We can get it
easy enough:

```
Generated user key - private key stored "~/.nkeys/SAOP/accounts/SYS/usersGenerated user creds file "~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds"
Success! - added user "SYSU" to "SYS"
```
```
nats-account-server -nsc ~/.nsc/nats/SAOP
```
```
The operator JWT - (~/.nsc/nats/<operator_name>/<operator.name>.jwt)
The URL wherhttp://localhost:9090/jwt/v1/accounts/e the server can resolve accounts ()
The public key of the system_account
```
```
nsc list accounts
```
### NATS-Account-Server

### NATS Server Configuration


Because the server has additional resolver implementations, you need to enclose the
server url like: URL(<url>).
Let's create server cong with the following contents and save it to server.conf:

Let's start the nats-server:

Let's add a subscriber for all the events published by the system account:

Very quickly we'll start seeing messages from the server as they are published by the
NATS server. As should be expected, the messages are just JSON, so they can easily be
inspected even if just using a simple nats sub to read them.
To see an account update:

The subscriber will print the connect and disconnect:

#### ╭│───────────────────────────────────────────── Accounts │

#### ├──────┬──────────────────────────────────────│ Name │ Public Key │

#### ├──────┼──────────────────────────────────────│ SYS │ ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF │

#### ╰──────┴──────────────────────────────────────

```
operator: /Users/synadia/.nsc/nats/SAOP/SAOP.jwtsystem_account: ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF
resolver: URL(http://localhost:9090/jwt/v1/accounts/)
```
```
nats-server -c server.conf
```
```
nats sub --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds ">"
```
```
nats pub --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds foo bar
```
## Inspecting Server Events


```
{ "server": {
"host""id": :"NBTGVY3OKDKEAJPUXRHZLKBCRH3LWCKZ6ZXTAJRS2RMYN3PMDRMUZWPR" "0.0.0.0", ,
"ver""seq":: "2.0.0-RC5" 32 , ,
```
(^) },"time": "2019-05-03T14:53:15.455266-05:00"
"acc""conns": "ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF": 1 , ,
(^) } "total_conns": 1
{ "server": {
"host""id": :"NBTGVY3OKDKEAJPUXRHZLKBCRH3LWCKZ6ZXTAJRS2RMYN3PMDRMUZWPR" "0.0.0.0", ,
"ver""seq":: "2.0.0-RC5" 33 , ,
(^) },"time": "2019-05-03T14:53:15.455304-05:00"
"client""start": {: "2019-05-03T14:53:15.453824-05:00",
"host""id": : 6 ,"127.0.0.1",
"acc""user": (^) :"ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF" "UACPEXCAZEYWZK4O52MEGWGK4BH3OSGYM3P3C3F3LF2NGNZUS24IVG36",,
"name""lang":: "NATS Sample Publisher""go", ,
"ver""stop": (^) :"1.7.0" "2019-05-03T14:53:15.45526-05:00",
} "sent", : {
"msgs""bytes": (^) : 1 , 3
} "received", : {
"msgs""bytes": (^) : 0 , 0
} "reason", : "Client Closed"
}

## User Services


For the active connection, get basic user information including the account name,
permissions, and expiry, if applicable. Note, this works with any connected user, not just a
system account user.

To discover servers in the cluster to get their ID and name, publish a request to
$SYS.REQ.SERVER.PING.IDZ.

```
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds \$SYS.RE
Published [$SYS.REQ.USER.INFO] : ''Received [_INBOX.DQD44ugVt0O4Ur3pWIOOD1.WQOBevoq] : '{
"user": "UACPEXCAZEYWZK4O52MEGWGK4BH3OSGYM3P3C3F3LF2NGNZUS24IVG36", "account": "ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF"
}'
```
```
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds \$SYS.RE
Published [$SYS.REQ.SERVER.PING.IDZ] : ''Received [_INBOX.DQD44ugVt0O4Ur3pWIOOD1.WQOBevoq] : '{
"host": "0.0.0.0", "id": "NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMCKQTGE5PUL",
"name": "n1"}'
```
### $SYS.REQ.USER.INFO - Request Connected User

### Information

## System Services

### $SYS.REQ.SERVER.PING.IDZ - Discovering Servers

### $SYS.REQ.SERVER.PING - Discovering Servers + Stats


To discover servers in the cluster, and get a small health summary, publish a request to
$SYS.REQ.SERVER.PING. Note that while the example below uses nats-req, only the
rst answer for the request will be printed. You can easily modify the example to wait
until no additional responses are received for a specic amount of time, thus allowing for
all responses to be collected.

```
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds \$SYS.RE
Published [$SYS.REQ.SERVER.PING] : ''Received [_INBOX.G5mbsf0k7l7nb4eWHa7GTT.omklmvnm] : '{
"server": { "host": "0.0.0.0",
"id": "NCZQDUX77OSSTGN2ESEOCP4X7GISMARX3H4DBGZBY34VLAI4TQEPK6P6", "ver": "2.0.0-RC9",
"seq": 47, "time": "2019-05-02T14:02:46.402166-05:00"
}, "statsz": {
"start": "2019-05-02T13:41:01.113179-05:00", "mem": 12922880,
"cores": 20, "cpu": 0,
"connections": 2, "total_connections": 2,
"active_accounts": 1, "subscriptions": 10,
"sent": { "msgs": 7,
"bytes": 2761 },
"received": { "msgs": 0,
"bytes": 0 },
"slow_consumers": 0 }
}'
```
### $SYS.REQ.SERVER.<id>.STATSZ - Requesting Server Stats

### Summary


If you know the server id for a particular server (such as from a response to
$SYS.REQ.SERVER.PING), you can query the specic server for its health information:

If proling is enabled for a server, this service enables requesting it from the server. The
request payload must specify the name of the prole being requested with an optional
debug level, including:

```
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds \$SYS.RE
Published [$SYS.REQ.SERVER.NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMReceived [_INBOX.DQD44ugVt0O4Ur3pWIOOD1.WQOBevoq] : '{
"server": { "host": "0.0.0.0",
"id": "NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMCKQTGE5PUL", "ver": "2.0.0-RC5",
"seq": 25, "time": "2019-05-03T14:34:02.066077-05:00"
}, "statsz": {
"start": "2019-05-03T14:32:19.969037-05:00", "mem": 11874304,
"cores": 20, "cpu": 0,
"connections": 2, "total_connections": 4,
"active_accounts": 1, "subscriptions": 10,
"sent": { "msgs": 26,
"bytes": 9096 },
"received": { "msgs": 2,
"bytes": 0 },
"slow_consumers": 0 }
}'
```
### $SYS.REQ.SERVER.<id>.PROFILEZ - Request Profiling

### Information


Sending a request to this service will attempt to hot reload the server conguration, akin
to nats-server --signal reload. If there are errors with the new conguration, they
will be returned in an error eld in the response.

```
allocs - 0, 1
block - 0
goroutine - 0, 1, 2
heap - 0, 1
mutex - 0
threadcount - 0
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds \$SYS.RE
Published [$SYS.REQ.SERVER.NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCM "name": "heap",
"debug": 1}'
Received [_INBOX.DQD44ugVt0O4Ur3pWIOOD1.WQOBevoq] : '{ "profile": "<base64-encoded profile output>"
}'
```
```
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds \$SYS.RE
Published [$SYS.REQ.SERVER.NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMReceived [_INBOX.DQD44ugVt0O4Ur3pWIOOD1.WQOBevoq] : '{
"server": { "host": "0.0.0.0",
"id": "NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMCKQTGE5PUL", "ver": "2.10.0-RC5",
"seq": 25, "time": "2023-09-19T14:34:02.066077-04:00"
}}'
```
### $SYS.REQ.SERVER.<id>.RELOAD - Hot Reload Configuration


# WebSocket

Supported since NATS Server version 2.2
WebSocket support can be enabled in the server and may be used alongside the
traditional TCP socket connections. TLS, compression and Origin Header checking are
supported.
**Important**

NATS Suppor(https://tools.ietf.orts only Wg/html/rebSocket data frfc6455#section-5.6ames in Binar). The server will alway, not Text formatys send in Binar (^) y
and your clients MUST send in Binary too.
For writers of client librNATS protocol (actually will generaries: a WebSocket frame is not guarally not). Any data from a frame must be goinganteed to contain a full
through a parser that can handle partial protocols. See the protocol description here.


# Configuration

WebSocket Conguration Example
To enable WebSocket support in the server, add a websocket conguration block in the
server's conguration le like the following:


websocket { # Specify a host and port to listen for websocket connections
# # listen: "host:port"

# It can also be configured with individual parameters, # namely host and port.
# # host: "hostname"
port: 443
# This will optionally specify what host:port for websocket # connections to be advertised in the cluster.
# # advertise: "host:port"

# TLS configuration is required by default #
tls { cert_file: "/path/to/cert.pem"
key_file: "/path/to/key.pem" }

# For test environments, you can disable the need for TLS # by explicitly setting this option to `true`
# # no_tls: true

# [Cross-origin resource sharing option](https://developer.mozilla.or #
# IMPORTANT! This option is used only when the http request presents # header, which is the case for web browsers. If no Origin header is
# this check will not be performed. #
# When set to `true`, the HTTP origin header must match the request’s # The default is `false`.
# # same_origin: true

# [Cross-origin resource sharing option](https://developer.mozilla.or #
# IMPORTANT! This option is used only when the http request presents # header, which is the case for web browsers. If no Origin header is
# this check will not be performed. #
# List of accepted origins. When empty, and `same_origin` is `false`, # This list specifies the only accepted values for the client's reque
# host and port must match. By convention, the absence of TCP port in


# host and port must match. By convention, the absence of TCP port in # for an "http://" scheme, and 443 for "https://".
# # allowed_origins [
# "http://www.example.com" # "https://www.other-example.com"
# ]
# This enables support for compressed websocket frames # in the server. For compression to be used, both server
# and client have to support it. #
# compression: true
# This is the total time allowed for the server to # read the client request and write the response back
# to the client. This includes the time needed for the # TLS handshake.
# # handshake_timeout: "2s"

# Name for an HTTP cookie, that if present will be used as a client JW # If the client specifies a JWT in the CONNECT protocol, this option
# The cookie should be set by the HTTP server as described [here](htt # This setting is useful when generating NATS `Bearer` client JWTs as
# result of some authentication mechanism. The HTTP server after corr # authentication can issue a JWT for the user, that is set securely p
# access by unintended scripts. Note these JWTs must be [NATS JWTs](h #
# jwt_cookie: "my_jwt_cookie_name"
# If no user name is provided when a websocket client connects, will # this user name in the authentication phase. If specified, this will
# override, for websocket clients, any `no_auth_user` value defined i # main configuration file.
# Note that this is not compatible with running the server in operato #
# no_auth_user: "my_username_for_apps_not_providing_credentials"
# See below to know what is the normal way of limiting websocket clie # to specific users.
# If there are no users specified in the configuration, this simple a # block allows you to override the values that would be configured in
# equivalent block in the main section. #
# authorization { # # If this is specified, the client has to provide the same user
# # and password to be able to connect. # # username: "my_user_name"


NATS supports different forms of authentication for clients connecting over WebSocket:

You can get some more information about how applications connecting over WebSocket
can use those different forms of authentication here

A new eld when conguring users allows you to restrict which type of connections are
allowed for a specic user.
Consider this conguration:

```
# # password: "my_password" #
# # If this is specified, the password field in the CONNECT has t # # match this token.
# # token: "my_token" #
# # This overrides the main's authorization timeout. For consiste # # with the main's authorization configuration block, this is ex
# # as a number of seconds. # # timeout: 2.0
#}}
```
```
username/password
token
NKEYS
client certicates
JWTs
```
## Authorization of WebSocket Users

### Authentication

### Restricting connection types


If a WebSocket client were to connect and use the username foo and password
foopwd, it would be accepted. Now suppose that you would want the WebSocket client
to only be accepted if it connected using the username bar and password barpwd,
then you would use the option allowed_connection_types to restrict which type of
connections can bind to this user.

The option allowed_connection_types (also can be named connection_types or
clients) as you can see is a list, and you can allow several types of clients. Suppose
you want the user bar to accept both standard NATS clients and WebSocket clients, you
would congure the user like this:

The absence of allowed_connection_types means that all types of connections are
allowed (the default behavior).
The possible values are currently:

```
authorization { users [
{user: foo password: foopwd, permission: {...}} {user: bar password: barpwd, permission: {...}}
]}
```
```
authorization { users [
{user: foo password: foopwd, permission: {...}} {user: bar password: barpwd, permission: {...}, allowed_connection_ty
]}
```
```
authorization { users [
{user: foo password: foopwd, permission: {...}} {user: bar password: barpwd, permission: {...}, allowed_connection_ty
]}
```
```
STANDARD
WEBSOCKET
```

You can congure remote Leaf node connections so that they connect to the Websocket
port instead of the Leaf node port. See Leafnode section.

When running on Docker, WebSocket is not enabled by default, so you'll have to create a
conguration le with the minimal entries, such as:

Assuming the conguration was stored in /tmp/nats.conf, you can start docker as
follows:

```
LEAFNODE
MQTT
```
```
websocket {
port: 8080 no_tls: true
}
```
```
docker run -it --rm -v /tmp:/container -p 8080 :8080 nats -c /container/n
```
## Leaf nodes connections

## Docker


