# Configuring NATS Server

The NATS server offers extensive configuration options through a configuration file, which supports a simple format that combines elements of traditional formats with modern styles like JSON and YAML. This flexibility makes it easy to customize the server settings for various deployment scenarios.

## Configuration File Syntax

- The configuration file is parsed with UTF-8 encoding.
- It is recommended to use only ASCII for names and values, limiting the use of Unicode to comments.

**Configuration file can include:**
- Comments using `#` or `//`.
- Values assigned to properties using `=`, `:`, or whitespace.
- Arrays enclosed in brackets `["a", "b", "c"]`.
- Maps enclosed in braces `{foo: 2}`.

**Example of configuration file usage:**

```plaintext
nats-server -config my-server.conf
```

### Basic Structure

**Example of a simple configuration:**

```plaintext
host: 0.0.0.0
port: 4222

authorization {
  users = [
    {user: "admin", password: "admin"}
  ]
}
```

### String and Number Handling

- String values starting with a digit should be quoted to avoid parsing errors.

**Example:**

```plaintext
authorization: {
  token: "3secret"  # Correct way
}
```

### Variables

Variables can be defined and referenced within the configuration file.

**Example:**

```plaintext
TOKEN: "secret"

authorization {
  token: $TOKEN
}
```

### Include Directive

You can split the configuration into multiple files using the `include` directive.

**Example:**

```plaintext
server.conf:
  listen: 127.0.0.1:4222
  include ./auth.conf

auth.conf:
  authorization: {
    token: "f0oBar"
  }
```

### Configuration Reloading

NATS server can reload most configuration changes without restarting by sending a reload signal:

**Example:**

```plaintext
nats-server --signal reload
```

## Configuration Properties

### General Settings

```plaintext
host: 0.0.0.0
port: 4222
```

### JetStream Configuration

JetStream can be enabled and configured for persistence, limits, and other settings.

**Example:**

```plaintext
jetstream {
  store_dir: "/data/jetstream"
  max_mem: 1G
  max_file: 100G
}
```

### Authentication and Authorization

Defines user authentication and authorization mechanisms.

**Example:**

```plaintext
authorization {
  users = [
    {user: "admin", password: "admin"}
  ]
}
```

### Clustering

NATS supports clustering for high availability and load balancing.

**Example:**

```plaintext
cluster {
  name: "example"
  listen: 127.0.0.1:4248
  routes = [
    nats://route_user:pwd@127.0.0.1:4245,
    nats://route_user:pwd@127.0.0.1:4246
  ]
}
```

## Configuring JetStream

JetStream configuration can be specified at the server level or for specific accounts.

**Example:**

```plaintext
jetstream {
  store_dir: /data/jetstream
  max_mem: 1G
  max_file: 100G
}

accounts {
  HR: {
    jetstream {
      max_mem: 512M
      max_file: 1G
      max_streams: 10
      max_consumers: 100
    }
  }
}
```

## Configuration Management Tools

Several tools can assist in managing the configuration:

### NATS Admin CLI

The `nats` CLI tool can be used to manage Streams and Consumers.

**Example:**

```plaintext
nats str add ORDERS --config orders.json
nats str edit ORDERS --config orders.json
nats con add ORDERS NEW --config orders_new.json
```

### Terraform

Terraform can be used to manage NATS configurations through the `terraform-provider-jetstream`.

**Example:**

```plaintext
provider "jetstream" {
  servers = "connect.ngs.global"
  credentials = "ngs_jetstream_admin.creds"
}

resource "jetstream_stream" "ORDERS" {
  name = "ORDERS"
  subjects = ["ORDERS.*"]
  storage = "file"
  max_age = 60 * 60 * 24 * 365
}
```

### GitHub Actions

GitHub Actions can automate managing a running JetStream server.

**Example:**

```plaintext
jobs:
  clean_orders:
    runs-on: ubuntu-latest
    steps:
      - name: Delete ORDERS stream
        uses: nats-io/jetstream-gh-action/delete/stream@master
        with:
          stream: ORDERS
          server: js.example.net

  create_orders:
    runs-on: ubuntu-latest
    needs: clean_orders
    steps:
      - name: Create ORDERS stream
        uses: nats-io/jetstream-gh-action/create/stream@master
        with:
          config: ORDERS.json
          server: js.example.net
```

### Kubernetes Controller

Manage NATS JetStream Streams and Consumers via K8S CRDs.

**Example:**

```yaml
apiVersion: jetstream.nats.io/v1beta1
kind: Stream
metadata:
  name: mystream
spec:
  subjects: ["orders.*"]
  storage: memory
  maxAge: 1h
---
apiVersion: jetstream.nats.io/v1beta1
kind: Consumer
metadata:
  name: my-push-consumer
spec:
  streamName: mystream
  durableName: my-push-consumer
  deliverSubject: my-push-consumer.orders
  deliverPolicy: last
  ackPolicy: none
```

## Clustering

NATS supports running each server in clustered mode for high volume messaging systems, resiliency, and high availability.

**Example:**

```plaintext
cluster {
  name: "example"
  listen: "localhost:4248"
  routes = [
    "nats://route_user:pwd@127.0.0.1:4245",
    "nats://route_user:pwd@127.0.0.1:4246"
  ]
}
```

### Advanced Clustering Options

- **Connection pooling**: Introduced in v2.10.0 to improve performance.
- **Account pinning**: Allows dedicating routes for specific accounts.
- **Compression**: Can be enabled to reduce bandwidth usage.

**Example:**

```plaintext
cluster {
  pool_size: 3
  accounts: [acc1, acc2]
  compression: {
    mode: accept
  }
}
```

By following these guidelines and examples, you can effectively configure and manage a NATS server to suit various deployment scenarios, ensuring high performance, security, and scalability.
### Monitoring

When monitoring is enabled, the route connection's information will have a new field called `compression` that displays the current compression mode. The possible values are `off` or any of the compression modes described below. Note that `s2_auto` is not displayed directly; instead, the current mode (e.g., `s2_best`, `s2_fast`) will be shown. If connected to an older server that doesn't support compression, the `compression` field will display `not supported`.

### Disabling Connection Pooling

To disable connection pooling, set the `pool_size` configuration parameter to `-1`. This will cause the server to behave as it did pre-v2.10.0, creating a single route connection between its peers. Note that in this mode, no accounts list can be defined.

### Connecting to an Older Server

A v2.10.0+ server with a compression mode configured can connect to an older server that does not support compression. In this case, the connection will simply not use compression.

## Compression

Compression can help reduce the amount of data transmitted over network connections. NATS supports several compression modes, which do not need to be the same between routed servers.

### Compression Modes

- **off**: Explicitly disables compression for any route between the server and a peer.
- **accept (default)**: Does not initiate compression but will accept the compression mode of the peer it connects to.
- **s2_fast**: Applies compression optimized for speed over compression ratio.
- **s2_better**: Applies compression with a balance of speed and compression ratio.
- **s2_best**: Applies compression optimized for compression ratio over speed.
- **s2_auto**: Chooses the appropriate `s2_*` mode based on the round-trip time (RTT) measured between the server and the peer.

### Round-trip Time Thresholds

When using `s2_auto` compression, the `rtt_thresholds` option defines three latency thresholds that dictate increasing or decreasing the compression mode. The default `rtt_thresholds` value is `[10ms, 50ms, 100ms]`.

**Example:**

```plaintext
cluster {
  compression: {
    mode: s2_auto
    rtt_thresholds: [10ms, 50ms, 100ms]
  }
}
```

### Configuration Reload

Compression configuration can be changed through configuration reload. If the value is changed from `off` to another mode, connections will be closed and recreated. For all other compression modes, the mode is changed dynamically without closing route connections.

## JetStream Clustering

JetStream clustering uses the RAFT algorithm for high availability and scalability. Each server participating in a cluster requires a unique `server_name` within the same domain.

### RAFT Groups

- **Meta Group**: Manages the JetStream API and server placement.
- **Stream Group**: Synchronizes state and data between its members. If there's no leader, the stream will not accept messages.
- **Consumer Group**: Synchronizes consumer state between its members.

### Quorum

A quorum is the minimum number of nodes required to ensure data consistency across restarts. It is calculated as `½ cluster size + 1`.

### Cluster Size Recommendations

Generally, a cluster of 3 or 5 JetStream enabled servers is recommended. This balances scalability with failure tolerance. For instance, in a 5-server cluster, you can lose one server without affecting the cluster's operation.

### Mixing JetStream Enabled Servers with Standard NATS Servers

You can mix JetStream enabled servers with standard NATS servers to optimize for different workloads. JetStream servers handle persistent data, while standard NATS servers handle non-persistent data.

### Configuration

Below are examples of configurations for a three-node JetStream cluster across three machines.

### Server 1 (host_a)

```plaintext
server_name = "n1-c1"
listen = 4222

accounts {
  $SYS {
    users = [
      { user: "admin", pass: "$2a$11$DRh4C0KNbNnD8K/hb/buWe1zPxEHrLEiDmuq1Mi0rRJiH/W25Qi" }
    ]
  }
}

jetstream {
  store_dir = "/nats/storage"
}

cluster {
  name = "C1"
  listen = "0.0.0.0:6222"
  routes = [
    "nats://host_b:6222",
    "nats://host_c:6222"
  ]
}
```

### Server 2 (host_b)

```plaintext
server_name = "n2-c1"
listen = 4222

accounts {
  $SYS {
    users = [
      { user: "admin", pass: "$2a$11$DRh4C0KNbNnD8K/hb/buWe1zPxEHrLEiDmuq1Mi0rRJiH/W25Qi" }
    ]
  }
}

jetstream {
  store_dir = "/nats/storage"
}

cluster {
  name = "C1"
  listen = "0.0.0.0:6222"
  routes = [
    "nats://host_a:6222",
    "nats://host_c:6222"
  ]
}
```

### Server 3 (host_c)

```plaintext
server_name = "n3-c1"
listen = 4222

accounts {
  $SYS {
    users = [
      { user: "admin", pass: "$2a$11$DRh4C0KNbNnD8K/hb/buWe1zPxEHrLEiDmuq1Mi0rRJiH/W25Qi" }
    ]
  }
}

jetstream {
  store_dir = "/nats/storage"
}

cluster {
  name = "C1"
  listen = "0.0.0.0:6222"
  routes = [
    "nats://host_a:6222",
    "nats://host_b:6222"
  ]
}
```

### Administration

Once a JetStream cluster is operating, interactions with the CLI and the `nats` CLI are the same as before. When adding a stream using the `nats` CLI, you will be asked for the number of replicas.

**Example:**

```plaintext
nats str add ORDERS --replicas 3
```

**Output:**

```plaintext
Information for Stream ORDERS created 2021-02-05T12:07:34+01:00
Configuration:
  Replicas: 3
Cluster Information:
  Name: C1
  Leader: n1-c1
  Replica: n4-c1, current, seen 0.07s ago
  Replica: n3-c1, current, seen 0.07s ago
```

### Viewing Stream Placement and Stats

```plaintext
nats stream report
```

### Forcing Stream and Consumer Leader Election

```plaintext
nats stream cluster step-down ORDERS
```

### Viewing the Cluster State

```plaintext
nats server report jetstream --user admin --password s3cr3t!
```

### Forcing Meta Group Leader Election

```plaintext
nats server raft step-down --user admin --password s3cr3t!
```

### Evicting a Peer

```plaintext
nats stream cluster peer-remove ORDERS
```

## Troubleshooting

Diagnosing problems in NATS JetStream clusters requires knowledge of JetStream concepts and the NATS CLI. 

### Basic Commands

- **nats account info**: Verify JetStream is enabled on an account.
- **nats server ls**: List known servers.
- **nats server ping**: Ping all servers.
- **nats server info**: Show information about a single server.
- **nats server check**: Health check for NATS servers.

### Reporting Commands

- **nats server report connections**: Report on connections.
- **nats server report accounts**: Report on account activity.
- **nats server report jetstream**: Report on JetStream activity.

### Request Commands

- **nats server request jetstream**: Show JetStream details.
- **nats server request subscriptions**: Show subscription information.
- **nats server request variables**: Show runtime variables.
- **nats server request connections**: Show connection details.
- **nats server request routes**: Show route details.
- **nats server request gateways**: Show gateway details.
- **nats server request leafnodes**: Show leafnode details.
- **nats server request accounts**: Show account details.

### Cluster Commands

- **nats server cluster step-down**: Force a new leader election by standing down the current meta leader.
- **nats server cluster peer-remove**: Remove a server from a JetStream cluster.

## Super-cluster with Gateways

Gateways connect multiple clusters into a super-cluster. Unlike clusters, gateways form a full mesh between clusters and use uni-directional connections.

### Gateway Configuration

**Example:**

```plaintext
gateway {
  name: "A"
  listen: "localhost:7222"
  authorization {
    user: "gwu"
    password: "gwp"
  }
  gateways: [
    {name: "A", url: "nats://gwu:gwp@localhost:7222"},
    {name: "B", url: "nats://gwu:gwp@localhost:7333"},
    {name: "C", url: "nats://gwu:gwp@localhost:7444"}
  ]
}
```

## Leaf Nodes

Leaf Nodes extend an existing NATS system, optionally bridging both operator and security domains. Leaf nodes

 route messages as needed from local clients to one or more remote NATS systems and vice versa.

### Leaf Node Configuration

**Example:**

**Main Server:**

```plaintext
leafnodes {
  port: 7422
}
authorization {
  token: "s3cr3t"
}
```

**Leaf Node Server:**

```plaintext
leafnodes {
  remotes = [
    {
      url: "nats://s3cr3t@localhost:7422"
    }
  ]
}
listen: 4111
```

```plaintext
nats-server -c /tmp/server.conf
```

```plaintext
nats reply -s nats://s3cr3t@localhost q 42
```

This guide provides an overview of configuring and managing NATS and JetStream clusters, including handling compression, clustering, monitoring, and troubleshooting.
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

### Service

The `nsc` tool is used to manage NATS configurations. Let's create a user for our example using `nsc`:

1. **Create a User**:

    ```sh
    nsc add user leaftestuser
    ```

    This command generates and stores the user key and credentials file, and adds the user `leaftestuser` to the account `leaftest`.

2. **Configure the Leaf Node**:

    ```plaintext
    leafnodes {
        remotes = [
            {
                url: "tls://connect.ngs.global"
                credentials: "/Users/alberto/.nkeys/creds/synadia/leaftest/leaftestuser.creds"
            }
        ]
    }
    ```

    Save this configuration to a file, e.g., `ngs_leaf.conf`.

3. **Start the Leaf Server**:

    ```sh
    nats-server -c /tmp/ngs_leaf.conf
    ```

    The server will listen for client connections and connect the leaf node to `connect.ngs.global`.

4. **Connect a Replier**:

    Connect a replier to Synadia's NGS using the credentials file.

5. **Make a Request**:

    From the local host, make a request to verify the setup.

### Leaf Authorization

Leaf node connections follow a model where, when a TCP connection is created to the server, the server immediately sends an INFO protocol message in clear text. This INFO protocol provides metadata, including whether the server requires a secure connection.

### TLS-first Handshake

In some environments, it is preferable to avoid having any traffic sent in clear text. To enforce this, configure both the accepting and remote servers to perform a TLS handshake before sending the INFO protocol message.

**Accepting Side Configuration**:

```plaintext
leafnodes {
    port: 7422
    tls {
        handshake_first: true
        cert_file: "./server-cert.pem"
        key_file: "./server-key.pem"
    }
}
```

**Remote Side Configuration**:

```plaintext
leafnodes {
    remotes = [
        {
            urls: ["tls://example:7422"]
            tls {
                handshake_first: true
                cert_file: "./client-cert.pem"
                key_file: "./client-key.pem"
            }
        }
    ]
}
```

### LeafNode Configuration Block

The leaf node configuration block is used to configure incoming and outgoing leaf node connections.

**Properties**:

- `host`: Interface where the server will listen for incoming leafnode connections.
- `port`: Port for incoming leafnode connections (default is 7422).
- `listen`: Combines host and port as `<host>:<port>`.
- `tls`: TLS configuration block.
- `advertise`: Hostport to advertise to leaf nodes.
- `no_advertise`: If true, the server shouldn't be advertised to leaf nodes.
- `authorization`: Authorization block.
- `remotes`: List of remote entries specifying servers where leafnode client connections can be made.
- `reconnect`: Interval in seconds at which reconnect attempts to a remote server are made.
- `compression`: Configuration for compression of leafnode connections (defaults to `s2_auto`).

**Example**:

```plaintext
leafnodes {
    port: 7422
    tls {
        handshake_first: true
        cert_file: "./server-cert.pem"
        key_file: "./server-key.pem"
    }
    authorization {
        users = [
            { user: "leaf1", password: "secret", account: "account1" },
            { user: "leaf2", password: "secret", account: "account2" }
        ]
    }
}
```

**Remote Configuration Example**:

```plaintext
leafnodes {
    remotes = [
        {
            urls: ["tls://example:7422"]
            tls {
                handshake_first: true
                cert_file: "./client-cert.pem"
                key_file: "./client-key.pem"
            }
        }
    ]
}
```

### JetStream on Leaf Nodes

To support disconnected use cases with JetStream, independent JetStream islands are supported and available through the same NATS network. Each server in a cluster or super cluster needs to have the same domain name, which can change between servers connected via a leaf node connection.

**Example Configuration**:

**Accounts Configuration**:

```plaintext
accounts {
    SYS: {
        users: [{ user: "admin", password: "admin" }]
    },
    ACC: {
        users: [{ user: "acc", password: "acc" }]
        jetstream: enabled
    }
}
system_account: SYS
```

**Hub Configuration**:

```plaintext
port: 4222
server_name: hub-server
jetstream {
    store_dir: "./store_server"
    domain: "hub"
}
leafnodes {
    port: 7422
}
include "./accounts.conf"
```

**Leaf Configuration**:

```plaintext
port: 4111
server_name: leaf-server
jetstream {
    store_dir: "./store_leaf"
    domain: "leaf"
}
leafnodes {
    remotes = [
        {
            urls: ["nats://admin:admin@0.0.0.0:7422"]
            account: "SYS"
        },
        {
            urls: ["nats://acc:acc@0.0.0.0:7422"]
            account: "ACC"
        }
    ]
}
include "./accounts.conf"
```

**Start the Servers**:

```sh
nats-server -c hub.conf
nats-server -c leaf.conf
```

**Create and Verify Streams**:

1. **Create a Stream in Hub Domain**:

    ```sh
    nats --server nats://admin:admin@localhost:4222 stream add
    ```

2. **Create a Stream in Leaf Domain**:

    ```sh
    nats --server nats://acc:acc@localhost:4222 stream add --js-domain leaf
    ```

3. **Publish a Message**:

    ```sh
    nats --server nats://acc:acc@localhost:4222 pub test "hello world"
    ```

4. **Verify Stream Reports**:

    ```sh
    nats --server nats://acc:acc@localhost:4222 stream report --js-domain leaf
    nats --server nats://acc:acc@localhost:4222 stream report --js-domain hub
    ```

**Copy Streams Across Domains**:

1. **Create a Mirror Stream**:

    ```sh
    nats --server nats://acc:acc@localhost:4222 stream add --js-domain hub --mirror
    ```

2. **Create a Source Stream**:

    ```sh
    nats --server nats://acc:acc@localhost:4222 stream add --js-domain hub --source
    ```

### Cross Account and Domain Import

To share domain access across accounts, modify the `accounts.conf` to export and import the necessary subjects.

```plaintext
accounts {
    SYS: {
        users: [{ user: "admin", password: "admin" }]
    },
    ACC: {
        users: [{ user: "acc", password: "acc" }]
        jetstream: enabled
        exports: [
            { service: "$JS.hub.API.CONSUMER.CREATE.*", response_type: "stream" },
            { stream: "deliver.acc.hub.>" },
            { service: "$JS.hub.API.CONSUMER.MSG.NEXT.aggregate-test-leaf.dur.>" },
            { service: "$JS.ACK.aggregate-test-leaf.dur.>" },
            { service: "$JS.FC.aggregate-test-leaf.dur.>" }
        ]
    },
    IMPORT_MIRROR: {
        users: [{ user: "import_mirror", password: "import_mirror" }]
        jetstream: enabled
        imports: [
            { service: { account: "ACC", subject: "$JS.hub.API.CONSUMER.CREATE.*" } },
            { service: { account: "ACC", subject: "$JS.FC.aggregate-test-leaf.dur.>" } },
            { stream: { account: "ACC", subject: "deliver.acc.hub.import_mirror.>" } }
        ]
    },
    IMPORT_CLIENT: {
        users: [{ user: "import_client", password: "import_client" }]
        jetstream: enabled
        imports: [
            { service: { account: "ACC", subject: "$JS.hub.API.CONSUMER.MSG.NEXT.aggregate-test-leaf.dur.>" } },
            { service: { account: "ACC", subject: "$JS.ACK.aggregate-test-leaf.dur.>" } }
        ]
    }
}
system_account: SYS
```

### Securing NATS

NATS server provides security through TLS encryption, client authentication, and authorization for subjects they publish or subscribe to.

**Enabling TLS**:

```plaintext
tls {
    cert_file: "./server-cert.pem"
    key_file: "./server-key.pem"
    ca_file: "./ca-cert.pem"
}
```

**Start Server with TLS**:

```sh
nats-server --tls --tlscert=./server-cert.pem --tlskey=./server-key.pem
```

**Self-Signed Certificates for Testing**:

Self-signed certificates are useful for testing but should not be used in production.

```plaintext
tls {
    cert_file: "./server-cert.pem"
    key_file: "./server-key.pem"
    ca_file: "./ca-cert.pem"
}
```

**Generate Self-Signed Certificates**:

```sh
openssl req -new -x509 -days 365 -nodes -out server-cert.pem -keyout server-key.pem
```



Ensure that the certificate contains the appropriate Subject Alternative Name (SAN) entries to match the IP or DNS names used for connections.

This guide provides an overview of configuring and managing NATS and JetStream clusters, including handling leaf nodes, TLS encryption, and cross-domain stream management.
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
### Creating Self-Signed Certificates for Testing

To generate self-signed certificates for testing purposes, follow these steps:

1. **Install mkcert**:
    ```sh
    mkcert -install
    ```

2. **Generate Certificates**:
    ```sh
    mkcert -cert-file server-cert.pem -key-file server-key.pem localhost
    mkcert -client -cert-file client-cert.pem -key-file client-key.pem localhost
    ```

    Note: The `client` in `mkcert -client` refers to the connecting process, not necessarily a NATS client. It will generate a certificate with key usage suitable for client and server authentication.

3. **Verify Certificates**:
    ```sh
    openssl x509 -noout -text -in server-cert.pem
    openssl x509 -noout -text -in client-cert.pem
    ```

4. **Check mkcert CA location**:
    ```sh
    mkcert -CAROOT
    ```

5. **Remove CA from System Trust Store (after testing)**:
    ```sh
    mkcert -uninstall
    ```

**TLS-Terminating Reverse Proxies**

Using a TLS-terminating reverse proxy with NATS is not supported due to the nature of the TLS upgrade mechanism NATS uses. However, workarounds can be used in client libraries to make it work, such as providing a CustomDialer.

Refer to the following links for details:
- [nats.js CustomDialer Example](https://github.com/nats-io/nats.js/issues/369)
- [nats.rs CustomDialer Example](https://github.com/nats-io/nats.rs/blob/main/async-nats/src/connector.rs)
- [nats.net CustomDialer Example](https://github.com/nats-io/nats.net/tree/main/src/Samples/TLSReverseProxyExample)

## Authentication

NATS provides various ways of authenticating clients, including token authentication, username/password, TLS certificates, NKEY with challenge, and decentralized JWT authentication/authorization.

### Authentication Methods

1. **Token Authentication**:
    - Simplest form of authentication using a predefined token.
    - Can be used with bcrypt for added security.

    ```sh
    nats-server --auth s3cr3t
    ```

    Client connection:
    ```sh
    nats sub -s nats://s3cr3t@localhost:4222 ">"
    ```

    Generating bcrypt tokens:
    ```sh
    nats server passwd
    ```

2. **Username/Password Authentication**:
    - Supports multiple users with plain text or bcrypt passwords.

    Single user configuration:
    ```plaintext
    authorization: {
        user: "a",
        password: "b"
    }
    ```

    Multiple users configuration:
    ```plaintext
    authorization: {
        users: [
            { user: "a", password: "b" },
            { user: "b", password: "a" }
        ]
    }
    ```

    Reloading configuration:
    ```sh
    nats-server --signal reload
    ```

3. **TLS Authentication**:
    - Requires client certificates and can map certificate attributes to user identity.

    Example:
    ```plaintext
    tls {
        cert_file: "server-cert.pem"
        key_file: "server-key.pem"
        ca_file: "rootCA.pem"
        verify_and_map: true
    }
    ```

    Authorizing user:
    ```plaintext
    authorization: {
        users: [
            { user: "email@localhost" }
        ]
    }
    ```

### NKeys

NKeys provide a highly secure public-key signature system based on Ed25519.

Generating User NKEY:
```sh
nk -gen user -pubout
```

Server configuration:
```plaintext
authorization: {
    users: [
        { nkey: "UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4" }
    ]
}
```

Client configuration (node.js example):
```javascript
const NATS = require('nats');
const nkeys = require('ts-nkeys');

const nkey_seed = 'SUACSSL3UAHUDXKFSNVUZRF5UHPMWZ6BFDTJ7M6USDXIEDNPPQYYYCU3VYUDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4';

const nc = NATS.connect({
    port: 4222,
    nkey: 'UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4',
    sigCB: function(nonce) {
        const sk = nkeys.fromSeed(Buffer.from(nkey_seed));
        return sk.sign(nonce);
    }
});
```

### Authentication Timeout

Specify a timeout value for authentication to prevent abuse:
```plaintext
authorization: {
    timeout: 3,
    users: [
        { user: "a", password: "b" },
        { user: "b", password: "a" }
    ]
}
```

## Decentralized JWT Authentication/Authorization

JWT authentication leverages JSON Web Tokens (JWT) for a distributed configuration paradigm. JWTs describe accounts and users, improving account management by eliminating the need for server configuration updates.

NKeys roles:
- **Operators**
- **Accounts**
- **Users**

JWT authentication configuration is typically managed using the `nsc` tool. JWTs are signed using the Ed25519 algorithm and verified by the server. The server uses the operator JWT to trust specific operators and obtain account JWTs via resolvers.

### Managing JWT Authentication

1. **NATS Based Resolver**:
    - Preferred method for account lookup.
    - Full resolver stores all JWTs and synchronizes them in an eventually consistent way.
    - Cache resolver stores a subset of JWTs and evicts others based on an LRU scheme.

    Example full resolver configuration:
    ```plaintext
    resolver: {
        type: "full",
        dir: "./jwt",
        allow_delete: false,
        interval: "2m",
        limit: 1000
    }
    ```

    Example cache resolver configuration:
    ```plaintext
    resolver: {
        type: "cache",
        dir: "./",
        limit: 1000,
        ttl: "2m"
    }
    ```

2. **Memory Resolver**:
    - Used for small, static configurations.
    - Preloads JWTs in the server's configuration file.

    Example:
    ```plaintext
    resolver_preload: {
        ACSU3Q6LTLBVLGAQUONAGXJHVNWGSKKAUA7IY5TB4Z7PLEKSR5O6JTGR: "eyJ0eXAiOiJqd3Q..."
    }
    ```

3. **URL Resolver**:
    - Specifies a URL for the server to retrieve account JWTs.

    Example:
    ```plaintext
    resolver: "http://localhost:9090/jwt/v1/accounts/"
    ```

Use the `nsc` tool to manage account data and synchronize it with the NATS servers. The NATS-based resolver is preferred for its built-in functionality and ease of use.

This guide provides an overview of configuring and managing NATS authentication and security, including self-signed certificates, TLS, NKeys, and JWT-based authentication/authorization.
## URL Resolver

The URL resolver allows the server to fetch account JWTs from a specified URL. This is useful when using an external account resolver service.

Note: If you are not using a nats-account-server, the URL can be anything, as long as appending the public key for an account returns the requested JWT.

Example URL resolver configuration:
```plaintext
resolver: URL(http://localhost:9090/jwt/v1/accounts/)
```

To further restrict TLS for the resolver, you can specify `resolver_tls`:
```plaintext
resolver_tls: {
  cert_file: "path/to/client-cert.pem",
  key_file: "path/to/client-key.pem",
  ca_file: "path/to/ca-cert.pem"
}
```

## Memory Resolver Tutorial

The MEMORY resolver is built into the server and is suitable for scenarios with a small number of accounts that do not change frequently.

### Basic Configuration

To set up the MEMORY resolver:
1. **Add an Operator**:
    ```sh
    nsc add operator -n memory
    ```

2. **Add an Account**:
    ```sh
    nsc add account --name A
    ```

3. **Describe the Account**:
    ```sh
    nsc describe account -W
    ```

4. **Create a User**:
    ```sh
    nsc add user --name TA
    ```

5. **Generate Server Config**:
    ```sh
    nsc generate config --mem-resolver --config-file /tmp/server.conf
    ```

6. **Manual Server Config**:
    Reference the operator and account JWTs in the server configuration:
    ```plaintext
    operator: /Users/synadia/.nsc/nats/memory/memory.jwt
    resolver: MEMORY
    resolver_preload: {
      ACSU3Q6LTLBVLGAQUONAGXJHVNWGSKKAUA7IY5TB4Z7PLEKSR5O6JTGR: "eyJ0eXAiOiJqd3Q..."
    }
    ```

7. **Start the Server**:
    ```sh
    nats-server -c /tmp/server.conf
    ```

8. **Test the Configuration**:
    ```sh
    nats pub --creds ~/.nkeys/creds/memory/A/TA.creds hello world
    ```

### Mixed Authentication/Authorization Setup

Mixing NKEYS static configuration and decentralized JWT authentication/authorization requires preparation:
1. **Prepare a Trusted Operator Setup**:
    ```sh
    nsc add accountuser --name SYS
    nsc add accountuser -a --name A test
    nsc add accountuser -a --name B test
    ```

2. **Generate Memory-Based Resolver Config**:
    ```sh
    nsc generate config --mem-resolver --sys-account SYS
    ```

3. **Example Node Configuration**:
    ```plaintext
    port = 4222
    cluster {
      port = 6222
      routes = [ nats://127.0.0.1:6223 ]
    }
    system_account = ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO
    accounts {
      ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2 {
        nkey: ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2
        users = [ {nkey: "UAPOK2P7EN3UFBL7SBJPQK3M3JMLALYRYKX5XWSVMVYK63ZMBHTOHVJR" } ]
      }
    }
    ```

4. **Second Node Configuration**:
    ```plaintext
    port = 4223
    cluster {
      port = 6223
      routes = [ nats://127.0.0.1:6222 ]
    }
    system_account = ABKOWIYVTHNEK5HELPWLAT2CF2CUPELIK4SZH2VCJHLFU22B5U2IIZUO
    resolver = MEMORY
    resolver_preload = {
      ADFB2JXYTXOJEL6LNAXDREUGRX35BOLZI3B4PFFAC7IRPR3OA4QNKBN2: "eyJ0eXAiOiJqd3Q...",
      ACWOMQA7PZTKJSBTR7BF6TBK3D776734PWHWDKO7HFMQOM5BIOYPSYZZ: "eyJ0eXAiOiJqd3Q..."
    }
    ```

**Multi-Tenancy using Accounts**

Accounts provide isolated communication contexts, enabling multi-tenancy in NATS deployment. Each account can define users, exports, and imports of streams and services.

Example Account Configuration:
```plaintext
accounts: {
  A: {
    users: [ {user: a, password: a} ]
    exports: [ {stream: "puba.>"} ]
  },
  B: {
    users: [ {user: b, password: b} ]
    imports: [ {stream: {account: A, subject: "puba.>"}, prefix: "from_a"} ]
  }
}
```

### Exporting and Importing

Exports configuration example:
```plaintext
accounts: {
  A: {
    users: [ {user: a, password: a} ]
    exports: [
      {stream: "puba.>"},
      {service: "pubq.>"},
      {stream: "b.>", accounts: ["B"]},
      {service: "q.b", accounts: ["B"]}
    ]
  }
}
```

Imports configuration example:
```plaintext
accounts: {
  B: {
    users: [ {user: b, password: b} ]
    imports: [
      {stream: {account: A, subject: "b.>"}},
      {service: {account: A, subject: "q.b"}}
    ]
  }
}
```

### No Auth User

Clients connecting without authentication can be associated with a particular user within an account.

Example Configuration:
```plaintext
accounts: {
  A: {
    users: [ {user: a, password: a} ]
  }
}
no_auth_user: a
```

## See Also

For more details on advanced configuration and OCSP Stapling, refer to the documentation.

### Logging

NATS server supports various logging options:
```sh
nats-server -DV -m 8222 -user foo -pass bar
nats-server -DV -m 8222 -l nats.log
```

Configuration file example:
```plaintext
debug: false
trace: true
logtime: false
logfile_size_limit: 1GB
log_file: "/tmp/nats-server.log"
```

Use `logrotate` for log rotation:
```sh
nats-server -r udp://localhost:514
nats-server -r syslog://logs.papertrailapp.com:26900
```
## Available Events and Services

## System Account

In addition to system account privileges, other tools can initiate requests using system account endpoints. The monitoring endpoints listed below are accessible as system services using the following subject patterns:

### System Events

#### Server-Initiated Events
- `$SYS.ACCOUNT.<id>.CONNECT` (client connects)
- `$SYS.ACCOUNT.<id>.DISCONNECT` (client disconnects)
- `$SYS.ACCOUNT.<id>.SERVER.CONNS` (connections for an account changed)
- `$SYS.SERVER.<id>.CLIENT.AUTH.ERR` (authentication error)
- `$SYS.ACCOUNT.<id>.LEAFNODE.CONNECT` (leaf node connects)
- `$SYS.ACCOUNT.<id>.LEAFNODE.DISCONNECT` (leaf node disconnects)
- `$SYS.SERVER.<id>.STATSZ` (stats summary)

### Monitoring Endpoints
- `$SYS.REQ.SERVER.<id>.<endpoint-name>` (request server monitoring endpoint corresponding to endpoint name)
- `$SYS.REQ.SERVER.PING.<endpoint-name>` (request monitoring endpoint from all servers, will return multiple messages)

#### Endpoint Names
- `VARZ` - General Server Information
- `CONNZ` - Connections
- `ROUTEZ` - Routing
- `GATEWAYZ` - Gateways
- `LEAFZ` - Leaf Nodes
- `SUBSZ` - Subscription Routing
- `JSZ` - JetStream
- `ACCOUNTZ` - Accounts

Servers like nats-account-server publish system account messages when a claim is updated. The nats-server listens for these messages and updates its account information accordingly.

With these few messages, you can build useful monitoring tools:

### Example Configuration for System Events
To use system events with accounts, your configuration can look like this:
```plaintext
accounts: { 
  USERS: {
    users: [ {user: a, password: a} ]
  },
  SYS: { 
    users: [ {user: admin, password: changeit} ]
  },
}
system_account: SYS
```

### Subscribing to System Events
Subscribe to all system events:
```sh
nats sub -s nats://admin:changeit@localhost:4222 ">"
```
Example command to observe events:
```sh
nats pub -s "nats://a:a@localhost:4222" foo bar
```

## Enabling System Events with Decentralized Authentication/Authorization

### Create an Operator, Account, and User
1. Create an operator:
   ```sh
   nsc add operator -n SAOP
   ```
2. Add the system account:
   ```sh
   nsc add account -n SYS
   ```
3. Add a system account user:
   ```sh
   nsc add user -n SYSU
   ```

### Running a NATS Account Server
Start a nats-account-server to serve the JWT credentials:
```sh
nats-account-server -nsc ~/.nsc/nats/SAOP
```

### NATS Server Configuration
Create a server configuration file (`server.conf`) with the following contents:
```plaintext
operator: /Users/synadia/.nsc/nats/SAOP/SAOP.jwt
system_account: ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF
resolver: URL(http://localhost:9090/jwt/v1/accounts/)
```

Start the NATS server:
```sh
nats-server -c server.conf
```

### Subscribing to Events
Add a subscriber for all events published by the system account:
```sh
nats sub --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds ">"
```

Inspect server events by publishing a message:
```sh
nats pub --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds foo bar
```

### Inspecting Server Events
Example output for account connect and disconnect events:
```json
{
  "server": {
    "host": "0.0.0.0",
    "id": "NBTGVY3OKDKEAJPUXRHZLKBCRH3LWCKZ6ZXTAJRS2RMYN3PMDRMUZWPR",
    "ver": "2.0.0-RC5",
    "seq": 32
  },
  "time": "2019-05-03T14:53:15.455266-05:00",
  "acc": "ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF",
  "conns": 1,
  "total_conns": 1
}
```

### User Services
#### Request Connected User Information
```sh
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds '$SYS.REQ.USER.INFO' ''
```
Example response:
```json
{
  "user": "UACPEXCAZEYWZK4O52MEGWGK4BH3OSGYM3P3C3F3LF2NGNZUS24IVG36",
  "account": "ADWJVSUSEVC2GHL5GRATN2LOEOQOY2E6Z2VXNU3JEIK6BDGPWNIW3AXF"
}
```

#### Discovering Servers
```sh
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds '$SYS.REQ.SERVER.PING.IDZ' ''
```
Example response:
```json
{
  "host": "0.0.0.0",
  "id": "NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMCKQTGE5PUL",
  "name": "n1"
}
```

### Requesting Server Stats
```sh
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds '$SYS.REQ.SERVER.<id>.STATSZ' ''
```
Example response:
```json
{
  "server": {
    "host": "0.0.0.0",
    "id": "NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMCKQTGE5PUL",
    "ver": "2.0.0-RC5",
    "seq": 25,
    "time": "2019-05-03T14:34:02.066077-05:00"
  },
  "statsz": {
    "start": "2019-05-03T14:32:19.969037-05:00",
    "mem": 11874304,
    "cores": 20,
    "cpu": 0,
    "connections": 2,
    "total_connections": 4,
    "active_accounts": 1,
    "subscriptions": 10,
    "sent": {
      "msgs": 26,
      "bytes": 9096
    },
    "received": {
      "msgs": 2,
      "bytes": 0
    },
    "slow_consumers": 0
  }
}
```

### Request Profiling Information
```sh
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds '$SYS.REQ.SERVER.<id>.PROFILEZ' '{"name": "heap", "debug": 1}'
```
Example response:
```json
{
  "profile": "<base64-encoded profile output>"
}
```

### Hot Reload Configuration
```sh
nats request --creds ~/.nkeys/SAOP/accounts/SYS/users/SYSU.creds '$SYS.REQ.SERVER.<id>.RELOAD' ''
```
Example response:
```json
{
  "server": {
    "host": "0.0.0.0",
    "id": "NC7AKPQRC6CIZGWRJOTVFIGVSL7VW7WXTQCTUJFNG7HTCMCKQTGE5PUL",
    "ver": "2.10.0-RC5",
    "seq": 25,
    "time": "2023-09-19T14:34:02.066077-04:00"
  }
}
```

# WebSocket

Supported since NATS Server version 2.2, WebSocket support can be enabled in the server and used alongside traditional TCP socket connections.

### Configuration

Example WebSocket configuration block:
```plaintext
websocket {
  port: 443
  tls {
    cert_file: "/path/to/cert.pem"
    key_file: "/path/to/key.pem"
  }
  no_tls: true
  same_origin: true
  allowed_origins: [
    "http://www.example.com",
    "https://www.other-example.com"
  ]
  compression: true
  handshake_timeout: "2s"
  jwt_cookie: "my_jwt_cookie_name"
  no_auth_user: "my_username_for_apps_not_providing_credentials"
  authorization {
    username: "my_user_name"
    password: "my_password"
    token: "my_token"
    timeout: 2.0
  }
}
```

### Authorization of WebSocket Users

NATS supports different forms of authentication for clients connecting over WebSocket, including username/password, token, NKEYS, client certificates, and JWTs.

### Restricting Connection Types



You can restrict connection types for users:
```plaintext
authorization { 
  users [
    {user: foo, password: foopwd, permission: {...}},
    {user: bar, password: barpwd, permission: {...}, allowed_connection_types: ["STANDARD", "WEBSOCKET"]}
  ]
}
```

## Docker Configuration for WebSocket

When running on Docker, create a configuration file with minimal entries:
```plaintext
websocket {
  port: 8080
  no_tls: true
}
```
Start Docker with the configuration file:
```sh
docker run -it --rm -v /tmp:/container -p 8080:8080 nats -c /container/nats.conf
```

This configuration and commands provide a comprehensive guide to setting up and managing NATS system events, monitoring, MQTT, and WebSocket features, ensuring secure and efficient operations.