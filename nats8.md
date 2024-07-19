# Installing, running and

# deploying a NATS Server

The NATS Server is highly optimized and its binary is very compact (less than 20 MB), it
can run on any machine from a lowly Raspberry Pi to the largest of servers, in the cloud,
on premise or at the edge, on bare metal, on VMs or in containers.

NATS Servers can cluster together to provide fault-tolerance and scalability. NATS servers
can run as Leaf Nodes connecting to a cluster transparently proxying and expanding
NATS and JetStream service to any location, be it a VPC, a VLAN, a remote oce, a single
machine at home or even an edge location with partial connectivity.

NATS clusters in different regions or cloud providers can be connected together by
gateways that lter unneeded trac and allow for disaster recovery and queued
distributed processing with geo-location anity.

Even better, if you have build your application with NATS you don't need to run your own
server, cluster, or super-cluster and can simply leverage Synadia's NGS, the global NATS
super-cluster service provided by Synadia.

This section of the documentation explains how to install, deploy, run, congure,
orchestrate, test and manage NATS servers.


# Installing a NATS Server

NATS philosophy is simplicity. Installation is just decompressing a zip le and copying
the binary to an appropriate directory; you can also use your favorite package manager.
Here's a list of different ways you can install or run NATS:

See also installing the NATS client

The following table indicates the current supported NATS server build combinations for
operating systems and architectures.

```
Command Line
Docker
Kubernetes
Package Manager
Release Zip
Development Build
```
```
Operating System Architectures Status
Darwin (macOS) amd64, arm64 Stable
Linux amd64, 386, arm6, arm7, arm64, mips64le, s390x Stable
Windows amd64, 386, arm6, arm7, arm64 Stable
FreeBSD amd64 Stable
NetBSD - Experimental
IBM z/OS - Experimental
```
## Supported operating systems and architectures


Note, not all installation methods below have distributions for all OS and architecture
combinations.

The NATS server itself has minimal hardware requirements to support small edge
devices, but can take advantage of more resources if available.

CPU should be considered in accepting TLS connections. After a network partition, every
disconnected client will attempt to connect to a NATS server in the cluster
simultaneously, so CPU on those servers will momentarily spike. When there are many
clients this can be mitigated with reconnect jitter settings, and errors can be reduced with
longer TLS timeouts, and scaling up cluster sizes.

We highly recommend testing to see if smaller, cheaper machines suce for your
workload - often they do! We suggest starting here and adjusting resources after load
testing specic to your environment. When using cloud provider instance types make
sure the node has a sucient NIC to support the required bandwidth for the application
needs.

For high throughput use cases, the network interface card (NIC) or the available
bandwidth are often the bottleneck, so ensure the hardware or cloud provider instance
types are sucient for your needs.

The table below notes the minimum number of cores and memory with the different
combinations of publishers, subscribers, and message rate where the single server or
cluster remained stable (not slow nor hitting an out-of-memory). These were tested inside
containers with GOMEMLIMIT set to 90% of the memory allocation and with a 2021-era
CPU and SSD for JetStream storage.

All message rates are per second.

## Hardware requirements

### Core NATS


This table follows the same pattern as above, however the published messages are being
received by a stream using le storage with the one replica or three (for a cluster size of
three). The subscriber is relying on a "pull consumer" for fetching messages.

```
Cluster Size CPU cores Memory Subscribers Publishers
```
```
1 1 32 MiB 1 100
1 1 64 MiB 1 1000
3 1 32 MiB 1 1000
3 1 64 MiB 1 1000
```
```
Cluster Size CPU cores Memory Subscribers Publishers
```
```
1 1 32 MiB 1 10
1 1 32 MiB 1 100
1 1 64 MiB 1 100
1 1 64 MiB 1 1000
3 1 32 MiB 1 100
3 1 64 MiB 1 100
3 1 64 MiB 1 1000
3 1 256 MiB 1 1000
```
### With JetStream


The simplest way to just get the binary of a release of nats-server for your machine is
to use the following shell command.

For example to get the binary for version 2.10.14 you would use:

To get the current very latest version (which may be ahead of the current last ocially
released version!) use @latest, you can also use a tag or a specic branch after the @
.

With Docker you can install the server easily without scattering binaries and other
artifacts on your system. The only pre-requisite is to install docker.

To run NATS on Docker:

More information on containerized NATS is available here.

```
curl -sf https://binaries.nats.dev/nats-io/nats-server/v2@v2.10.14 | sh
```
```
docker pull nats:latest
```
```
docker run -p 4222 :4222 -ti nats:latest
```
```
[1] 2019/05/24 15:42:58.228063 [INF] Starting nats-server version #.#.#
[1] 2019/05/24 15:42:58.228115 [INF] Git commit [#######]
[1] 2019/05/24 15:42:58.228201 [INF] Starting http monitor on 0.0.0.0:
[1] 2019/05/24 15:42:58.228740 [INF] Listening for client connections on
[1] 2019/05/24 15:42:58.228765 [INF] Server is ready
[1] 2019/05/24 15:42:58.229003 [INF] Listening for route connections on 0
```
## Getting the binary from the command line

## Installing via Docker


On Windows:

On Mac OS:

To test your installation (provided the executable is visible to your shell):

Typing nats-server should output something like

On Linux:

You can nd the latest release of nats-server on the nats-io/nats-server GitHub releases
page.

From the releases page, copy the link to the release archive le of your choice and
download it using curl -L.

For example, assuming version X.Y.Z of the server and a Linux AMD64:

```
choco install nats-server
```
```
brew install nats-server
```
```
[41634] 2019/05/13 09:42:11.745919 [INF] Starting nats-server version 2.*
[41634] 2019/05/13 09:42:11.746240 [INF] Listening for client connections
...
[41634] 2019/05/13 09:42:11.746249 [INF] Server id is NBNYNR4ZNTH4N2UQKSAA
[41634] 2019/05/13 09:42:11.746252 [INF] Server is ready
```
```
yay nats-server
```
## Installing via a Package Manager

## Downloading a Release Build


and nally:

If you have Go installed, installing the binary is easy:

This mechanism will install a build of the main branch, which almost certainly will not be
a released version. If you are a developer and want to play with the latest, this is the
easiest way.

To test your installation (provided $GOPATH/bin is in your path) by typing nats-server
which should output something like

```
curl -L https://github.com/nats-io/nats-server/releases/download/vX.Y.Z/n
```
```
unzip nats-server.zip -d nats-server
```
```
Archive: nats-server.zip
creating: nats-server-vX.Y.Z-linux-amd64/
...
```
```
sudo cp nats-server/nats-server-vX.Y.Z-linux-amd64/nats-server /usr/bin
```
```
go install github.com/nats-io/nats-server/v2@latest
```
```
[2397474] 2023/09/27 10:32:02.709019 [INF] Starting nats-server
[2397474] 2023/09/27 10:32:02.709165 [INF] Version: 2.10.
[2397474] 2023/09/27 10:32:02.709182 [INF] Git: [not set]
[2397474] 2023/09/27 10:32:02.709185 [INF] Name: NDQU7SGA4ECW4PHL4K
[2397474] 2023/09/27 10:32:02.709187 [INF] ID: NDQU7SGA4ECW4PHL4K
[2397474] 2023/09/27 10:32:02.709795 [INF] Listening for client connectio
[2397474] 2023/09/27 10:32:02.710173 [INF] Server is ready
```
## Installing From the Source

## NATS v2 and Go Modules


If you are having issues when using the recent versions of NATS and Go modules such
as:

To x it:

1. Update your go.mod using the latest tags, for example for both NATS and NATS
    Streaming clients:

Or if you want to import the NATS Server v2 to embed it, notice the /v2 after the nats-
server module name. If that is not present, then go modules will not fetch it and would
accidentally end up with 1.4.1 version of the server.

If embedding both NATS Streaming and NATS Servers:

2. Next, update the imports within the repo:

```
go: github.com/nats-io/go-nats@v1.8.1: parsing go.mod: unexpected module
go: github.com/nats-io/go-nats-streaming@v0.5.0: parsing go.mod: unexpect
```
```
module github.com/wallyqs/hello-nats-go-mod
go 1.
require (
github.com/nats-io/nats.go v1.8.
github.com/nats-io/stan.go v0.5.
)
```
```
require (
github.com/nats-io/nats-server/v2 v2.0.
github.com/nats-io/nats.go v1.8.
)
```
```
require (
github.com/nats-io/nats-server/v2 v2.0.0 // indirect
github.com/nats-io/nats-streaming-server v0.15.
)
```

3. (Recommended) Run Go fmt as the rename will affect the proper ordering of the
    imports

When using go get to fetch the client, include an extra slash at the end of the repo. For
example:

When trying to fetch the latest version of the server with go get, you have to add v2 at
the end:

Otherwise, go get will fetch the v1.4.1 version of the server, which is also named (
gnatsd), the previous name for nats-server.

In order to use an older tag, you will have to use the previous name (gnatsd) otherwise it
will result in go mod parsing errors.

```
find ./ -type f -name "*.go" -exec sed -i -e 's/github.com\/nats-io\/go-n
find ./ -type f -name "*.go" -exec sed -i -e 's/github.com\/nats-io\/go-n
find ./ -type f -name "*.go" -exec sed -i -e 's/github.com\/nats-io\/gnat
find ./ -type f -name "*.go" -exec sed -i -e 's/github.com\/nats-io\/nats
```
```
GO111MODULE=on go get github.com/nats-io/nats.go/@latest
GO111MODULE=on go get github.com/nats-io/nats.go/@v1.8.
```
```
GO111MODULE=on go get github.com/nats-io/nats-server/v2@latest
```
```
GO111MODULE=on go get github.com/nats-io/nats-server@latest
go: finding github.com/nats-io/gnatsd/server latest
go: finding golang.org/x/crypto/bcrypt latest
go: finding golang.org/x/crypto latest
```
### Gotchas when using go get


For more information you can review the original issue in GitHub.

```
# OK
GO111MODULE=on go get github.com/nats-io/go-nats/@v1.7.
# Not OK
GO111MODULE=on go get github.com/nats-io/nats.go/@v1.7.
go: finding github.com/nats-io/nats.go v1.7.
go: downloading github.com/nats-io/nats.go v1.7.
go: extracting github.com/nats-io/nats.go v1.7.
go: finding github.com/nats-io/go-nats/encoders/builtin latest
go: finding github.com/nats-io/go-nats/util latest
go: finding github.com/nats-io/go-nats/encoders latest
go: finding github.com/nats-io/go-nats v1.8.
go: downloading github.com/nats-io/go-nats v1.8.
go: extracting github.com/nats-io/go-nats v1.8.
go: github.com/nats-io/go-nats@v1.8.1: parsing go.mod: unexpected module
go: error loading module requirements
```

# Running and deploying a NATS Server

The nats-server has many command line options. To get started, you don't have to specify
anything. In the absence of any ags, the NATS server will start listening for NATS client
connections on port 4222. By default, security is disabled.

When the server starts it will print some information including where the server is
listening for client connections:

If you are running your NATS server in a docker container:

```
nats-server
```
```
[61052] 2021/10/28 16:53:38.003205 [INF] Starting nats-server
[61052] 2021/10/28 16:53:38.003329 [INF] Version: 2.6.
[61052] 2021/10/28 16:53:38.003333 [INF] Git: [not set]
[61052] 2021/10/28 16:53:38.003339 [INF] Name: NDUP6JO4T5LRUEXZUHWX
[61052] 2021/10/28 16:53:38.003342 [INF] ID: NDUP6JO4T5LRUEXZUHWX
[61052] 2021/10/28 16:53:38.004046 [INF] Listening for client connections
[61052] 2021/10/28 16:53:38.004683 [INF] Server is ready
...
```
```
docker run -p 4222 :4222 -ti nats:latest
```
## Standalone

## Docker


You can easily and quickly use systemd to start (and restart if needed) the
nats-server process.

Please see the example les located in the util directory of the nats-server repo that
you can use to generate your own /etc/systemd/system/nats.service le.

Remember that in order to enable JetStream and all the functionalities that use it you
need to enable it on at least one of your servers

Enable JetStream by specifying the -js ag when starting the NATS server.

```
$ nats-server -js
```
```
[1] 2021/10/28 23:51:52.705376 [INF] Starting nats-server
[1] 2021/10/28 23:51:52.705428 [INF] Version: 2.6.
[1] 2021/10/28 23:51:52.705432 [INF] Git: [c91f0fe]
[1] 2021/10/28 23:51:52.705439 [INF] Name: NB32AP7VSM3FTKTVEGPQ3OZW
[1] 2021/10/28 23:51:52.705446 [INF] ID: NB32AP7VSM3FTKTVEGPQ3OZW
[1] 2021/10/28 23:51:52.705448 [INF] Using configuration file: nats-serve
[1] 2021/10/28 23:51:52.709505 [INF] Starting http monitor on 0.0.0.0:
[1] 2021/10/28 23:51:52.709590 [INF] Listening for client connections on
[1] 2021/10/28 23:51:52.709882 [INF] Server is ready
[1] 2021/10/28 23:51:52.710394 [INF] Cluster name is 3tlKqFWx91wnnAekR76U
[1] 2021/10/28 23:51:52.710419 [WRN] Cluster name was dynamically generat
[1] 2021/10/28 23:51:52.710446 [INF] Listening for route connections on 0
...
```
## Running nats-server as a systemd service on

## Linux

## JetStream

### Command Line


You can also enable JetStream through a conguration le. By default, the JetStream
subsytem will store data in the /tmp directory. Here's a minimal le that will store data in
a local "nats" directory, suitable for development and local testing.

Normally JetStream will be run in clustered mode and will replicate data, so the best
place to store JetStream data would be locally on a fast SSD. One should specically
avoid NAS or NFS storage for JetStream. More information on containerized NATS is
available here.

```
nats-server -c js.conf
```
```
# js.conf
jetstream {
store_dir=nats
}
```
### Configuration File


# Windows Service

The NATS server supports running as a Windows service. In fact, this is the
recommended way of running NATS on Windows. There is currently no installer; users
should use sc.exe to install the service:

The above will create and start a nats-server service. Note the nats-server ags
should be provided when creating the service. This allows for the running multiple NATS
server congurations on a single Windows server by using a 1:1 service instance per
installed NATS server service. Once the service is running, it can be controlled using
sc.exe or nats-server.exe --signal:

The above commands will default to controlling the nats-server service. If the service
is another name, it can be specied:

For a complete list of signals, see process signaling.

The default user in the above example will be System, which has local administrator
permissions and write access to almost all les on disk.

```
sc.exe create nats-server binPath= "%NATS_PATH%\nats-server.exe [nats-ser
sc.exe start nats-server
```
```
REM Reload server configuration
nats-server.exe --signal reload
REM Reopen log file for log rotation
nats-server.exe --signal reopen
REM Stop the server
nats-server.exe --signal stop
```
```
nats-server.exe --signal stop=<service name>
```
## Permissions


If you change the service user, e.g. to the more restricted NetworkService, make sure
permissions have been set. The server at a minimum will need read access to the cong
le and when using Jetstream, write access to the JetStream store directory.

Nats-server will write log entries to the default console when no log le is congured.
Console logging is not permitted for all users (e.g. not for NetworkService).

```
To ease debugging, it is recommended to run a NATS service with an explicit log le and
carefully check write permissions for the congured user.
```
The Windows service system requires communication with programs that run as
Windows services. One important signal from the program is the initial "ready" signal,
where the program informs Windows that it is running as expected.

By default nats-server allows itself 10 seconds to send this signal. If the server is not
ready after this time, the server will signal a failure to start. This delay can be adjusted by
setting the NATS_STARTUP_DELAY environment variable to a suitable duration (e.g. "20s"
for 20 seconds, "1m" for one minute). This adjustment can be necessary in cases where
NATS is correctly running from command line, but the service fails to start in this
timeframe.

```
sc config "nats-server" obj= "NT AUTHORITY\NetworkService" password= ""
```
```
sc.exe create nats-server binPath= "%NATS_PATH%\nats-server.exe --log C:\
```
## Windows Service Specific Settings

### NATS_STARTUP_DELAY environment variable


# Flags

The NATS server has many ags to customize its behavior without having to write a
conguration le.

The conguration ags revolve around:

```
Server Options
Logging
Authorization
TLS Security
Clustering
Information
```
```
Flag Description
-a, --addr, --net Host address to bind to (default: 0.0.0.0 - all interfaces).
-p, --port NATS client port (default: 4222).
-n, --name
, --server_name Server name (default auto).
-P, --pid File to store the process ID (PID).
```
```
-m, --http_port HTTP por(exclusive of t for monit--https_portoring dashboar). d^
```
```
-ms, --https_port HTTPS pordashboard (ext monitclusivoring for monite of --http_portoring^ ).
```
```
-c, --config Path to NATS server conguration le.
-sl, --signal Send a signal to nats-server process. See process signaling.
```
## Server Options


The following options control straightforward authentication:

See token authentication, and username/password for more information.

The following ags are available on the server to congure logging:

```
Flag Description
--client_advertise Client HostPort to advertise to other servers.
```
```
-t Test conguration and exit
```
```
`--ports_le_dir Creates a pordirectory (<exts le in the speciedecutable_name>_.por^ ts).
```
```
Flag Description
-js, --jetstream Enable JetStream functionality.
-sd, --store_dir Set the storage directory.
```
```
Flag Description
--user Required username for connections (exclusive of --auth).
--pass Required password for connections (exclusive of --auth).
```
```
--auth Requir(exclusived e of authorization t--user and oken--password for connections).^
```
## JetStream Options

## Authentication Options

## Logging Options


You can read more about logging conguration here.

```
Flag Description
-l, --log File to redirect log output
-T, --logtime Specify -T=false to disable timestamping log entries
-s, --syslog Log to syslog or windows event log
-r, --remote_syslog The syslog server address, like udp://localhost:
-D, --debug Enable debugging output
-V, --trace Enable protocol trace log messages
-VV Verbose trace (traces system account as well)
-DV Enable both debug and protocol trace messages
-DVV Debug and verbose trace (traces system account as well)
--max_traced_msg_len Maximum printable length for traced messages. 0 for unlimited
`--max_traced_msg_len Maximum printable length for traced messages (default: unlimited)
```
```
Flag Description
--tls Enable TLS, do not verify clients
--tlscert Server certicate le
--tlskey Private key for server certicate
--tlsverify Enable client TLS certicate verication
--tlscacert Client certicate CA for verication
```
## TLS Options


You can read more about tls conguration here.

The following ags are available on the server to congure clustering:

You can read more about clustering conguration here.

```
Flag Description
--routes Comma-separated list of cluster URLs to solicit and connect
--cluster Cluster URL for clustering requests
--no_advertise Do not advertise known cluster information to clients
--cluster_advertise Cluster URL to advertise to other servers
--connect_retries For implicit routes, number of connect retries
--cluster_listen Cluster url from which members can solicit routes
```
```
Flag Description
-h, --help Show this message
-v, --version Show version
--help_tls TLS help
```
## Cluster Options

## Common Options


