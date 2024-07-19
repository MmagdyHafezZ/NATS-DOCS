# Installing, Running, and Deploying a NATS Server

The NATS Server is highly optimized and its binary is very compact (less than 20 MB), making it suitable for running on any machine from a Raspberry Pi to the largest of servers, in the cloud, on-premise, or at the edge, on bare metal, VMs, or in containers.

NATS Servers can cluster together to provide fault-tolerance and scalability. They can run as Leaf Nodes connecting to a cluster, transparently proxying and expanding NATS and JetStream services to any location, such as a VPC, a VLAN, a remote office, a single machine at home, or even an edge location with partial connectivity.

NATS clusters in different regions or cloud providers can be connected together by gateways that filter unneeded traffic and allow for disaster recovery and queued distributed processing with geo-location affinity.

Even better, if you build your application with NATS, you don't need to run your own server, cluster, or super-cluster and can simply leverage Synadia's NGS, the global NATS super-cluster service provided by Synadia.

This section of the documentation explains how to install, deploy, run, configure, orchestrate, test, and manage NATS servers.

## Installing a NATS Server

NATS philosophy is simplicity. Installation is just decompressing a zip file and copying the binary to an appropriate directory; you can also use your favorite package manager. Here's a list of different ways you can install or run NATS:

- Command Line
- Docker
- Kubernetes
- Package Manager
- Release Zip
- Development Build

### Supported Operating Systems and Architectures

The following table indicates the current supported NATS server build combinations for operating systems and architectures.

| Operating System | Architectures                                     | Status       |
|------------------|---------------------------------------------------|--------------|
| Darwin (macOS)   | amd64, arm64                                      | Stable       |
| Linux            | amd64, 386, arm6, arm7, arm64, mips64le, s390x    | Stable       |
| Windows          | amd64, 386, arm6, arm7, arm64                     | Stable       |
| FreeBSD          | amd64                                             | Stable       |
| NetBSD           | -                                                 | Experimental |
| IBM z/OS         | -                                                 | Experimental |

Note that not all installation methods below have distributions for all OS and architecture combinations.

## Hardware Requirements

The NATS server itself has minimal hardware requirements to support small edge devices but can take advantage of more resources if available.

### Core NATS

The table below notes the minimum number of cores and memory for different combinations of publishers, subscribers, and message rates where the single server or cluster remained stable. These were tested inside containers with GOMEMLIMIT set to 90% of the memory allocation and with a 2021-era CPU and SSD for JetStream storage. All message rates are per second.

| Cluster Size | CPU Cores | Memory | Subscribers | Publishers |
|--------------|-----------|--------|-------------|------------|
| 1            | 1         | 32 MiB | 1           | 100        |
| 1            | 1         | 64 MiB | 1           | 1000       |
| 3            | 1         | 32 MiB | 1           | 1000       |
| 3            | 1         | 64 MiB | 1           | 1000       |

### With JetStream

This table follows the same pattern as above, with the published messages being received by a stream using file storage with one replica or three (for a cluster size of three). The subscriber is relying on a "pull consumer" for fetching messages.

| Cluster Size | CPU Cores | Memory   | Subscribers | Publishers |
|--------------|-----------|----------|-------------|------------|
| 1            | 1         | 32 MiB   | 1           | 10         |
| 1            | 1         | 32 MiB   | 1           | 100        |
| 1            | 1         | 64 MiB   | 1           | 100        |
| 1            | 1         | 64 MiB   | 1           | 1000       |
| 3            | 1         | 32 MiB   | 1           | 100        |
| 3            | 1         | 64 MiB   | 1           | 100        |
| 3            | 1         | 64 MiB   | 1           | 1000       |
| 3            | 1         | 256 MiB  | 1           | 1000       |

## Getting the Binary from the Command Line

The simplest way to get the binary of a release of `nats-server` for your machine is to use the following shell command. For example, to get the binary for version 2.10.14, use:

```sh
curl -sf https://binaries.nats.dev/nats-io/nats-server/v2@v2.10.14 | sh
```

To get the current latest version (which may be ahead of the current last officially released version), use `@latest`.

## Installing via Docker

With Docker, you can install the server easily without scattering binaries and other artifacts on your system. The only prerequisite is to install Docker.

To run NATS on Docker:

```sh
docker pull nats:latest
docker run -p 4222:4222 -ti nats:latest
```

This will print log messages indicating that the server is ready to accept connections.

## Installing via a Package Manager

### Windows

```sh
choco install nats-server
```

### Mac OS

```sh
brew install nats-server
```

### Linux

```sh
yay -S nats-server
```

## Downloading a Release Build

You can find the latest release of `nats-server` on the [nats-io/nats-server GitHub releases page](https://github.com/nats-io/nats-server/releases). From the releases page, copy the link to the release archive file of your choice and download it using `curl -L`.

For example, assuming version X.Y.Z of the server and a Linux AMD64:

```sh
curl -L https://github.com/nats-io/nats-server/releases/download/vX.Y.Z/nats-server-vX.Y.Z-linux-amd64.zip -o nats-server.zip
unzip nats-server.zip -d nats-server
sudo cp nats-server/nats-server-vX.Y.Z-linux-amd64/nats-server /usr/bin
```

## Installing from Source

If you have Go installed, installing the binary is easy:

```sh
go install github.com/nats-io/nats-server/v2@latest
```

This mechanism will install a build of the main branch, which almost certainly will not be a released version. If you are a developer and want to play with the latest, this is the easiest way.

## NATS v2 and Go Modules

If you are having issues when using recent versions of NATS and Go modules:

```sh
go: github.com/nats-io/go-nats@v1.8.1: parsing go.mod: unexpected module
go: github.com/nats-io/go-nats-streaming@v0.5.0: parsing go.mod: unexpect
```

### Fixing Go Module Issues

1. Update your `go.mod` using the latest tags:

```go
module github.com/yourusername/yourmodule
go 1.18
require (
    github.com/nats-io/nats.go v1.8.1
    github.com/nats-io/stan.go v0.5.0
)
```

Or if you want to import the NATS Server v2 to embed it, notice the `/v2` after the `nats-server` module name:

```go
require (
    github.com/nats-io/nats-server/v2 v2.0.0
    github.com/nats-io/nats.go v1.8.1
)
```

2. Next, update the imports within the repo:

```sh
find ./ -type f -name "*.go" -exec sed -i -e 's/github.com\/nats-io\/go-nats/github.com\/nats-io\/nats.go/g' {} \;
find ./ -type f -name "*.go" -exec sed -i -e 's/github.com\/nats-io\/gnatsd/github.com\/nats-io\/nats-server/g' {} \;
```

3. (Recommended) Run `go fmt` as the rename will affect the proper ordering of the imports.

When using `go get` to fetch the client, include an extra slash at the end of the repo:

```sh
GO111MODULE=on go get github.com/nats-io/nats.go/@latest
```

When trying to fetch the latest version of the server with `go get`, you have to add `v2` at the end:

```sh
GO111MODULE=on go get github.com/nats-io/nats-server/v2@latest
```

Otherwise, `go get` will fetch the v1.4.1 version of the server (previously named `gnatsd`).

For more information, you can review the [original issue on GitHub](https://github.com/nats-io/nats-server/issues/).

# Running and Deploying a NATS Server

The `nats-server` has many command line options. To get started, you don't have to specify anything. In the absence of any flags, the NATS server

 will start listening for NATS client connections on port 4222. By default, security is disabled.

When the server starts, it will print some information, including where the server is listening for client connections.

## Standalone

To run the server:

```sh
nats-server
```

The output will show:

```
[61052] 2021/10/28 16:53:38.003205 [INF] Starting nats-server
[61052] 2021/10/28 16:53:38.003329 [INF] Version: 2.6.2
[61052] 2021/10/28 16:53:38.003333 [INF] Git: [not set]
[61052] 2021/10/28 16:53:38.003339 [INF] Name: NDUP6JO4T5LRUEXZUHWX
[61052] 2021/10/28 16:53:38.003342 [INF] ID: NDUP6JO4T5LRUEXZUHWX
[61052] 2021/10/28 16:53:38.004046 [INF] Listening for client connections on port 4222
[61052] 2021/10/28 16:53:38.004683 [INF] Server is ready
```

## Docker

To run the NATS server in a Docker container:

```sh
docker run -p 4222:4222 -ti nats:latest
```

## Running nats-server as a systemd Service on Linux

You can use systemd to start (and restart if needed) the `nats-server` process.

Please see the example files located in the util directory of the nats-server repo that you can use to generate your own `/etc/systemd/system/nats.service` file.

### Enabling JetStream

Remember that to enable JetStream and all the functionalities that use it, you need to enable it on at least one of your servers. Enable JetStream by specifying the `-js` flag when starting the NATS server.

```sh
nats-server -js
```

```text
[1] 2021/10/28 23:51:52.705376 [INF] Starting
[1] 2021/10/28 23:51:52.705428 [INF] Version: 2.6.2
[1] 2021/10/28 23:51:52.705432 [INF] Git: [c91f0fe]
[1] 2021/10/28 23:51:52.705439 [INF] Name: NB32AP7VSM3FTKTVEGPQ3OZW
[1] 2021/10/28 23:51:52.705446 [INF] ID: NB32AP7VSM3FTKTVEGPQ3OZW
[1] 2021/10/28 23:51:52.705448 [INF] Using configuration file: nats-server.conf
[1] 2021/10/28 23:51:52.709505 [INF] Starting http monitor on 0.0.0.0:8222
[1] 2021/10/28 23:51:52.709590 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2021/10/28 23:51:52.709882 [INF] Server is ready
[1] 2021/10/28 23:51:52.710394 [INF] Cluster name is 3tlKqFWx91wnnAekR76U
[1] 2021/10/28 23:51:52.710419 [WRN] Cluster name was dynamically generated
[1] 2021/10/28 23:51:52.710446 [INF] Listening for route connections on 0.0.0.0:6222
```

### Configuration File

You can also enable JetStream through a configuration file. By default, the JetStream subsystem will store data in the `/tmp` directory. Here's a minimal file that will store data in a local "nats" directory, suitable for development and local testing.

```conf
# js.conf
jetstream {
    store_dir=nats
}
```

To start the server with this configuration:

```sh
nats-server -c js.conf
```

## Windows Service

The NATS server supports running as a Windows service. This is the recommended way of running NATS on Windows. There is currently no installer; users should use `sc.exe` to install the service:

```sh
sc.exe create nats-server binPath= "%NATS_PATH%\nats-server.exe [nats-server-flags]"
sc.exe start nats-server
```

The above will create and start a `nats-server` service. Note the `nats-server` flags should be provided when creating the service. This allows for running multiple NATS server configurations on a single Windows server by using a 1:1 service instance per installed NATS server service. Once the service is running, it can be controlled using `sc.exe` or `nats-server.exe --signal`:

```sh
REM Reload server configuration
nats-server.exe --signal reload

REM Reopen log file for log rotation
nats-server.exe --signal reopen

REM Stop the server
nats-server.exe --signal stop
```

The above commands will default to controlling the `nats-server` service. If the service is another name, it can be specified:

```sh
nats-server.exe --signal stop=<service name>
```

For a complete list of signals, see [process signaling](https://docs.nats.io/nats-server/configuration/signals).

### Permissions

If you change the service user, e.g., to the more restricted `NetworkService`, make sure permissions have been set. The server at a minimum will need read access to the configuration file and, when using JetStream, write access to the JetStream store directory.

Nats-server will write log entries to the default console when no log file is configured. Console logging is not permitted for all users (e.g., not for `NetworkService`).

To ease debugging, it is recommended to run a NATS service with an explicit log file and carefully check write permissions for the configured user.

```sh
sc.exe create nats-server binPath= "%NATS_PATH%\nats-server.exe --log C:\path\to\logfile.log"
```

### NATS_STARTUP_DELAY Environment Variable

The Windows service system requires communication with programs that run as Windows services. One important signal from the program is the initial "ready" signal, where the program informs Windows that it is running as expected.

By default, `nats-server` allows itself 10 seconds to send this signal. If the server is not ready after this time, the server will signal a failure to start. This delay can be adjusted by setting the `NATS_STARTUP_DELAY` environment variable to a suitable duration (e.g., "20s" for 20 seconds, "1m" for one minute). This adjustment can be necessary in cases where NATS is correctly running from the command line, but the service fails to start in this timeframe.

```sh
set NATS_STARTUP_DELAY=20s
```

# Flags

The NATS server has many flags to customize its behavior without having to write a configuration file. The configuration flags revolve around:

- Server Options
- Logging
- Authorization
- TLS Security
- Clustering
- Information

## Server Options

| Flag              | Description                                            |
|-------------------|--------------------------------------------------------|
| -a, --addr, --net | Host address to bind to (default: 0.0.0.0 - all interfaces). |
| -p, --port        | NATS client port (default: 4222).                      |
| -n, --name        | Server name (default auto).                            |
| -P, --pid         | File to store the process ID (PID).                    |
| -m, --http_port   | HTTP port for monitoring dashboard (exclusive of --https_port). |
| -ms, --https_port | HTTPS port for monitoring dashboard (exclusive of --http_port). |
| -c, --config      | Path to NATS server configuration file.                |
| -sl, --signal     | Send a signal to `nats-server` process. See [process signaling](https://docs.nats.io/nats-server/configuration/signals). |
| --client_advertise | Client HostPort to advertise to other servers.         |
| -t                | Test configuration and exit                            |
| --ports_file_dir  | Creates a ports file in the specified directory        |

## JetStream Options

| Flag        | Description                                      |
|-------------|--------------------------------------------------|
| -js, --jetstream | Enable JetStream functionality.                 |
| -sd, --store_dir | Set the storage directory for JetStream data.   |

## Authentication Options

The following options control straightforward authentication:

| Flag       | Description                                       |
|------------|---------------------------------------------------|
| --user     | Required username for connections (exclusive of --auth). |
| --pass     | Required password for connections (exclusive of --auth). |
| --auth     | Required authorization token for connections.     |

See [token authentication](https://docs.nats.io/nats-server/configuration/securing_nats/token) and [username/password](https://docs.nats.io/nats-server/configuration/securing_nats/user_pwd) for more information.

## Logging Options

| Flag               | Description                                             |
|--------------------|---------------------------------------------------------|
| -l, --log          | File to redirect log output                             |
| -T, --logtime      | Specify `-T=false` to disable timestamping log entries  |
| -s, --syslog       | Log to syslog or Windows event log                      |
| -r, --remote_syslog| The syslog server address, like `udp://localhost:514`   |
| -D, --debug        | Enable debugging output                                 |
| -V, --trace        | Enable protocol trace log messages                      |
| -VV                | Verbose trace (traces system account as well)           |
| -DV                | Enable both debug and protocol trace messages           |
| -DVV               | Debug and verbose trace (traces system account as well) |
| --max_traced_msg_len | Maximum printable length for traced messages (default: unlimited) |

You can read more about logging configuration [here](https://docs.nats.io/nats-server/configuration/logging).

## TLS Options

| Flag         | Description                                            |
|--------------|--------------------------------------------------------|
| --tls        | Enable TLS, do not verify clients                      |
| --tlscert    | Server certificate file                                |
| --tlskey     | Private key for server certificate                     |
| --tlsverify  | Enable client TLS certificate verification             |
| --tlscacert  | Client certificate CA for verification                 |

You can read more about TLS configuration [here](https://docs.nats.io/nats-server/configuration/securing_nats/tls).

## Cluster Options

| Flag              | Description                                              |
|-------------------|----------------------------------------------------------|
| --routes          | Comma-separated list of cluster URLs to solicit and connect |
| --cluster         | Cluster URL for clustering requests                      |
| --no_advertise    | Do not advertise known cluster information to clients    |
| --cluster_advertise | Cluster URL to advertise to other servers               |
| --connect_retries | For implicit routes, number of connect retries           |
| --cluster_listen  | Cluster URL from which members can solicit routes        |

You can read more about clustering configuration [here](https://docs.nats.io/nats-server/configuration/clustering).

## Common Options

|

 Flag        | Description                                      |
|-------------|--------------------------------------------------|
| -h, --help  | Show this message                                |
| -v, --version | Show version                                   |
| --help_tls  | TLS help                                         |

This concludes the refined and expanded documentation on installing, running, and deploying a NATS server. For more detailed information, please refer to the [official NATS documentation](https://docs.nats.io).