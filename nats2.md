# Managing and Monitoring Your NATS Server Infrastructure

Managing a NATS Server is simple, typical lifecycle operations include:

- Using the `nats` CLI tool to check server cluster connectivity and latencies, get account information, and manage and interact with streams (and other NATS applications). 

### Common `nats` CLI Commands

```sh
nats cheat
nats cheat server
```

- Use `nats stream --help` to monitor, manage, and interact with streams.
- Use `nats consumer --help` to monitor and manage stream consumers.
- Use `nats context --help` to switch between servers, clusters, or user credentials.
- Use the `nsc` CLI tool for JWT-based authentication and authorization to create and revoke operators, accounts, and user JWTs and keys.
- Sending signals to a server to reload a configuration or rotate log files.
- Upgrading a server (or cluster).
- Understanding slow consumers.
- Monitoring the server via:
  - The monitoring endpoint and tools like `nats-top`.
  - By subscribing to system events.
  - Gracefully shut down a server with Lame Duck Mode.

# Monitoring

To monitor the NATS messaging system, `nats-server` provides a lightweight HTTP server on a dedicated monitoring port. The monitoring server provides several endpoints, returning JSON objects with statistics and other information about the server state.

### Enabling Monitoring

Monitoring can be enabled in the server configuration or as a server command-line option. The conventional port is 8222.

#### Server Configuration
```plaintext
http_port: 8222
```

#### Command-Line Option
```sh
nats-server -m 8222
```

Once the server is running, go to [http://localhost:8222](http://localhost:8222) to browse the available endpoints.

### Monitoring Endpoints

- General Server Information
- Connections
- Routing
- Leaf Nodes
- Subscription Routing
- Account Information
- Account Stats
- JetStream Information
- Health

#### Example Response for /varz Endpoint
```json
{
  "server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW",
  "version": "2.0.0",
  "proto": 1,
  "go": "go1.12",
  "host": "0.0.0.0",
  "port": 4222,
  "max_connections": 65536,
  "ping_interval": 120000000000,
  "ping_max": 2,
  "http_host": "0.0.0.0",
  "http_port": 8222,
  "https_port": 0,
  "auth_timeout": 1,
  "max_control_line": 4096,
  "max_payload": 1048576,
  "max_pending": 67108864,
  "cluster": {},
  "gateway": {},
  "leaf": {},
  "tls_timeout": 0.5,
  "write_deadline": 2000000000,
  "start": "2019-06-24T14:24:43.928582-07:00",
  "now": "2019-06-24T14:24:46.894852-07:00",
  "uptime": "2s",
  "mem": 9617408,
  "cores": 4,
  "gomaxprocs": 4,
  "cpu": 0,
  "connections": 0,
  "total_connections": 0,
  "routes": 0,
  "remotes": 0,
  "leafnodes": 0,
  "in_msgs": 0,
  "out_msgs": 0,
  "in_bytes": 0,
  "out_bytes": 0,
  "slow_consumers": 2,
  "subscriptions": 0,
  "http_req_stats": {
    "/": 0,
    "/connz": 0,
    "/gatewayz": 0,
    "/routez": 0,
    "/subsz": 0,
    "/varz": 1
  },
  "config_load_time": "2019-06-24T14:24:43.928582-07:00",
  "slow_consumer_stats": {
    "clients": 1,
    "routes": 1,
    "gateways": 0,
    "leafs": 0
  }
}
```

## NATS Monitoring with Prometheus and Grafana

In addition to writing custom monitoring tools, you can monitor `nats-server` with Prometheus. The Prometheus NATS Exporter allows you to configure the metrics you want to observe and store in Prometheus. Grafana dashboards are available for visualizing server metrics.

### Example: JQuery Implementation
```js
$.getJSON("https://demo.nats.io:8222/connz?callback=?", function(data) {
  console.log(data);
});
```

## Monitoring JetStream

JetStream has a `/jsz` HTTP endpoint and advisories available. JetStream publishes advisories to NATS subjects below `$JS.EVENT.ADVISORY.>`.

### Example Advisory Subjects
- API interactions: `$JS.EVENT.ADVISORY.API`
- Stream CRUD operations: `$JS.EVENT.ADVISORY.STREAM.CREATED.<STREAM>`
- Consumer CRUD operations: `$JS.EVENT.ADVISORY.CONSUMER.CREATED.<STREAM>.<CONSUMER>`

### Viewing Events
```sh
nats event --js-advisory
```

## Managing JetStream

### Installing the Management Tool
```sh
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
nats --help
nats cheat
```

### Checking Account Information
```sh
nats account info
```

### Example Account Information Response
```plaintext
Connection Information:
Client ID: 8
Client IP: 127.0.0.1
RTT: 178.545μs
Headers Supported: true
Maximum Payload: 1.0 MiB
Connected URL: nats://localhost:4222
Connected Address: 127.0.0.1:4222
Connected Server ID: NCCOHA6ONXJOGAEZP4WPU4UJ3IQP2VVXEPRKTQCGBCW4IL4Y
JetStream Account Information:
Memory: 0 B of 5.7 GiB
Storage: 0 B of 11 GiB
Streams: 0 of Unlimited
Max Consumers: unlimited
```

## Creating and Managing Streams

### Adding a Stream
```sh
nats str add ORDERS
```

### Interactive Prompt
```plaintext
? Subjects to consume ORDERS.*
? Storage backend file
? Retention Policy Limits
? Discard Policy Old
? Message count limit -1
? Message size limit -1
? Maximum message age limit 1y
? Maximum individual message size -1
Stream ORDERS was created
```

### Confirm Stream Creation
```sh
nats str ls
nats str info ORDERS
```

### Example Stream Configuration
```plaintext
Information for Stream ORDERS created 2021-02-27T16:49:36-07:00

Configuration:
Subjects: ORDERS.*
Acknowledgements: true
Retention: File - Limits
Replicas: 1
Discard Policy: Old
Duplicate Window: 2m0s
Maximum Messages: unlimited
Maximum Bytes: unlimited
Maximum Age: 1y0d0h0m0s
Maximum Message Size: unlimited
Maximum Consumers: unlimited

State:
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```

### Publishing Into a Stream
```sh
nats pub ORDERS.scratch hello
nats req ORDERS.scratch hello
```

### Deleting All Data in a Stream
```sh
nats str purge ORDERS -f
nats str rmm ORDERS 1 -f
```

### Deleting a Stream
```sh
nats str rm ORDERS -f
nats str add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes
```

## Managing Consumers

### Creating Pull-Based Consumers
```sh
nats con add ORDERS NEW --filter ORDERS.received --ack explicit --max-deliver 20
```

### Creating Push-Based Consumers
```sh
nats con add ORDERS MONITOR --config monitor.json
```

### Querying Consumer Information
```sh
nats con ls ORDERS
nats con info ORDERS NEW
```

This comprehensive guide covers the essential operations for managing and monitoring NATS servers, including configuration, monitoring, and managing JetStream streams and consumers.
# Managing and Monitoring Your NATS Server Infrastructure

## Managing a NATS Server

Typical lifecycle operations include:

- Using the `nats` CLI tool to check server cluster connectivity and latencies, get account information, and manage and interact with streams (and other NATS applications). 
- Using the `nsc` CLI tool when using JWT-based authentication and authorization to create and revoke operators, accounts, and user JWTs and keys.
- Sending signals to a server to reload a configuration or rotate log files.
- Upgrading a server (or cluster).
- Understanding slow consumers.
- Monitoring the server via:
  - The monitoring endpoint and tools like `nats-top`.
  - Subscribing to system events.
- Gracefully shutting down a server with Lame Duck Mode.

### Common `nats` CLI Commands

```sh
nats cheat
nats cheat server
```

- Use `nats stream --help` to monitor, manage, and interact with streams.
- Use `nats consumer --help` to monitor and manage stream consumers.
- Use `nats context --help` if you need to switch between servers, clusters, or user credentials.

## Monitoring

To monitor the NATS messaging system, `nats-server` provides a lightweight HTTP server on a dedicated monitoring port. The monitoring server provides several endpoints, returning JSON objects with statistics and other information about the server state.

### Enabling Monitoring

Monitoring can be enabled in the server configuration or as a server command-line option. The conventional port is 8222.

#### Server Configuration

```plaintext
http_port: 8222
```

#### Command-Line Option

```sh
nats-server -m 8222
```

Once the server is running, go to [http://localhost:8222](http://localhost:8222) to browse the available endpoints.

### Monitoring Endpoints

- General Server Information
- Connections
- Routing
- Leaf Nodes
- Subscription Routing
- Account Information
- Account Stats
- JetStream Information
- Health

#### Example Response for /varz Endpoint

```json
{
  "server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW",
  "version": "2.0.0",
  "proto": 1,
  "go": "go1.12",
  "host": "0.0.0.0",
  "port": 4222,
  "max_connections": 65536,
  "ping_interval": 120000000000,
  "ping_max": 2,
  "http_host": "0.0.0.0",
  "http_port": 8222,
  "https_port": 0,
  "auth_timeout": 1,
  "max_control_line": 4096,
  "max_payload": 1048576,
  "max_pending": 67108864,
  "cluster": {},
  "gateway": {},
  "leaf": {},
  "tls_timeout": 0.5,
  "write_deadline": 2000000000,
  "start": "2019-06-24T14:24:43.928582-07:00",
  "now": "2019-06-24T14:24:46.894852-07:00",
  "uptime": "2s",
  "mem": 9617408,
  "cores": 4,
  "gomaxprocs": 4,
  "cpu": 0,
  "connections": 0,
  "total_connections": 0,
  "routes": 0,
  "remotes": 0,
  "leafnodes": 0,
  "in_msgs": 0,
  "out_msgs": 0,
  "in_bytes": 0,
  "out_bytes": 0,
  "slow_consumers": 2,
  "subscriptions": 0,
  "http_req_stats": {
    "/": 0,
    "/connz": 0,
    "/gatewayz": 0,
    "/routez": 0,
    "/subsz": 0,
    "/varz": 1
  },
  "config_load_time": "2019-06-24T14:24:43.928582-07:00",
  "slow_consumer_stats": {
    "clients": 1,
    "routes": 1,
    "gateways": 0,
    "leafs": 0
  }
}
```

## NATS Monitoring with Prometheus and Grafana

In addition to writing custom monitoring tools, you can monitor `nats-server` with Prometheus. The Prometheus NATS Exporter allows you to configure the metrics you want to observe and store in Prometheus. Grafana dashboards are available for visualizing server metrics.

### Example: JQuery Implementation

```js
$.getJSON("https://demo.nats.io:8222/connz?callback=?", function(data) {
  console.log(data);
});
```

## Monitoring JetStream

JetStream has a `/jsz` HTTP endpoint and advisories available. JetStream publishes advisories to NATS subjects below `$JS.EVENT.ADVISORY.>`.

### Example Advisory Subjects

- API interactions: `$JS.EVENT.ADVISORY.API`
- Stream CRUD operations: `$JS.EVENT.ADVISORY.STREAM.CREATED.<STREAM>`
- Consumer CRUD operations: `$JS.EVENT.ADVISORY.CONSUMER.CREATED.<STREAM>.<CONSUMER>`

### Viewing Events

```sh
nats event --js-advisory
```

## Managing JetStream

### Installing the Management Tool

```sh
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
nats --help
nats cheat
```

### Checking Account Information

```sh
nats account info
```

### Example Account Information Response

```plaintext
Connection Information:
Client ID: 8
Client IP: 127.0.0.1
RTT: 178.545μs
Headers Supported: true
Maximum Payload: 1.0 MiB
Connected URL: nats://localhost:4222
Connected Address: 127.0.0.1:4222
Connected Server ID: NCCOHA6ONXJOGAEZP4WPU4UJ3IQP2VVXEPRKTQCGBCW4IL4Y
JetStream Account Information:
Memory: 0 B of 5.7 GiB
Storage: 0 B of 11 GiB
Streams: 0 of Unlimited
Max Consumers: unlimited
```

## Creating and Managing Streams

### Adding a Stream

```sh
nats str add ORDERS
```

### Interactive Prompt

```plaintext
? Subjects to consume ORDERS.*
? Storage backend file
? Retention Policy Limits
? Discard Policy Old
? Message count limit -1
? Message size limit -1
? Maximum message age limit 1y
? Maximum individual message size -1
Stream ORDERS was created
```

### Confirm Stream Creation

```sh
nats str ls
nats str info ORDERS
```

### Example Stream Configuration

```plaintext
Information for Stream ORDERS created 2021-02-27T16:49:36-07:00

Configuration:
Subjects: ORDERS.*
Acknowledgements: true
Retention: File - Limits
Replicas: 1
Discard Policy: Old
Duplicate Window: 2m0s
Maximum Messages: unlimited
Maximum Bytes: unlimited
Maximum Age: 1y0d0h0m0s
Maximum Message Size: unlimited
Maximum Consumers: unlimited

State:
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```

### Publishing Into a Stream

```sh
nats pub ORDERS.scratch hello
nats req ORDERS.scratch hello
```

### Deleting All Data in a Stream

```sh
nats str purge ORDERS -f
nats str rmm ORDERS 1 -f
```

### Deleting a Stream

```sh
nats str rm ORDERS -f
nats str add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes
```

## Managing Consumers

### Creating Pull-Based Consumers

```sh
nats con add ORDERS NEW --filter ORDERS.received --ack explicit --max-deliver 20
```

### Creating Push-Based Consumers

```sh
nats con add ORDERS MONITOR --config monitor.json
```

### Listing Consumers

```sh
nats con ls ORDERS
```

```plaintext
Consumers for Stream ORDERS:
DISPATCH
MONITOR
NEW
```

### Querying Consumer Information

```sh
nats con info ORDERS DISPATCH
```

#### Example Output for Pull-Based Consumer

```plaintext
Information for Consumer ORDERS > DISPATCH

Configuration:
Durable Name: DISPATCH
Pull Mode: true
Subject: ORDERS.processed
Deliver All: true
Deliver Last: false
Ack Policy: explicit
Ack Wait: 30s
Replay Policy: instant
Sampling Rate: 100

State:
Last Delivered Message: Consumer sequence: 1 Stream sequence: 1
Acknowledgment floor: Consumer sequence: 0 Stream sequence: 0
Pending Messages: 0
Redelivered Messages: 0
```

### Consuming Pull-Based Consumers

#### Publishing Messages

```sh
nats pub ORDERS.processed "order 1"
nats pub ORDERS.processed "order

 2"
nats pub ORDERS.processed "order 3"
```

#### Consuming Messages

```sh
nats con next ORDERS DISPATCH
```

```plaintext
--- received on ORDERS.processed
order 1
Acknowledged message
```

```sh
nats con next ORDERS DISPATCH
```

```plaintext
--- received on ORDERS.processed
order 2
Acknowledged message
```

#### Consuming via Code

```sh
nats req '$JS.API.CONSUMER.MSG.NEXT.ORDERS.DISPATCH' ''
```

```plaintext
Published [$JS.API.CONSUMER.MSG.NEXT.ORDERS.DISPATCH] : ''
Received [ORDERS.processed] : 'order 3'
```

### Consuming Push-Based Consumers

#### Subscribing to Consumer Subject

```sh
nats sub monitor.ORDERS
```

```plaintext
Listening on [monitor.ORDERS]
[#3] Received on [ORDERS.processed]: 'order 3'
[#4] Received on [ORDERS.processed]: 'order 4'
```

## Data Replication

Replication allows you to move data between streams in either a 1:1 mirror style or by multiplexing multiple source streams into a new stream.

### Creating Streams

#### Adding the ARCHIVE Stream

```sh
nats s add ARCHIVE --source ORDERS --source RETURNS
```

#### Interactive Prompt

```plaintext
? Storage backend file
? Retention Policy Limits
? Discard Policy Old
? Stream Messages Limit -1
? Message size limit -1
? Maximum message age limit -1
? Maximum individual message size -1
? Duplicate tracking time window 2m0s
? Allow message Roll-ups No
? Allow message deletion Yes
? Allow purging subjects or the entire stream Yes
? Replicas 1
? Adjust source "ORDERS" start Yes
? ORDERS Source Start Sequence 0
? ORDERS Source UTC Time Stamp (YYYY:MM:DD HH:MM:SS)
? ORDERS Source Filter source by subject
? Import "ORDERS" from a different JetStream domain No
? Import "ORDERS" from a different account No
? Adjust source "RETURNS" start No
? Import "RETURNS" from a different JetStream domain No
? Import "RETURNS" from a different account No
Stream ARCHIVE was created
```

#### Example Configuration for ARCHIVE Stream

```plaintext
Information for Stream ARCHIVE created 2022-01-21T11:49:52-08:00

Configuration:
Acknowledgements: true
Retention: File - Limits
Replicas: 1
Discard Policy: Old
Duplicate Window: 2m0s
Allows Msg Delete: true
Allows Purge: true
Allows Rollups: false
Maximum Messages: unlimited
Maximum Bytes: unlimited
Maximum Age: unlimited
Maximum Message Size: unlimited
Maximum Consumers: unlimited
Sources: ORDERS, RETURNS

State:
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```

#### Adding the REPORT Stream

```sh
nats s add REPORT --mirror ARCHIVE
```

#### Interactive Prompt

```plaintext
? Storage backend file
? Retention Policy Limits
? Discard Policy Old
? Stream Messages Limit -1
? Message size limit -1
? Maximum message age limit -1
? Maximum individual message size -1
? Allow message Roll-ups No
? Allow message deletion Yes
? Allow purging subjects or the entire stream Yes
? Replicas 1
? Adjust mirror start No
? Import mirror from a different JetStream domain No
? Import mirror from a different account No
Stream REPORT was created
```

### Verifying Configuration

```sh
nats stream info ARCHIVE
nats stream info REPORT
```

#### Example Output for ARCHIVE Stream

```plaintext
Source Information:
Stream Name: ORDERS
Lag: 0
Last Seen: 2m23s
Stream Name: RETURNS
Lag: 0
Last Seen: 2m15s
```

#### Example Output for REPORT Stream

```plaintext
Mirror Information:
Stream Name: ARCHIVE
Lag: 0
Last Seen: 2m35s
```

### Replication Report

```sh
nats s report
```

#### Example Output

```plaintext
Obtaining Stream stats

+---------+---------+-----------+----------+---------+------+---------+--
| Stream  | Storage | Consumers | Messages | Bytes   | Lost | Deleted | C
+---------+---------+-----------+----------+---------+------+---------+--
| ORDERS  | Memory  | 1         | 100      | 3.3 KiB | 0    | 0       | n
| RETURNS | Memory  | 1         | 100      | 3.5 KiB | 0    | 0       | n
| ARCHIVE | File    | 1         | 200      | 27 KiB  | 0    | 0       | n
| REPORT  | File    | 0         | 200      | 27 KiB  | 0    | 0       | n
+---------+---------+-----------+----------+---------+------+---------+--
```

## Disaster Recovery

In the event of unrecoverable JetStream message persistence on one (or more) server nodes, there are two recovery scenarios:

1. Automatic recovery from intact quorum nodes.
2. Manual recovery from existing stream snapshots (backups).

### Automatic Recovery

NATS will create replacement stream replicas automatically under the following conditions:

- Impacted stream is of replica configuration R3 (or greater).
- Remaining intact nodes (stream replicas) meet minimum RAFT quorum: floor(R/2) + 1.
- Available node(s) in the stream's cluster for new replica(s).
- Impacted node(s) removed from the stream's domain RAFT Meta group (e.g. `nats server raft peer-remove`).

### Manual Recovery

Snapshots (also known as backups) can proactively be made of any stream regardless of replication configuration. The backup includes:

- Stream configuration and state.
- Stream durable consumer configuration and state.
- All message payload data including metadata like timestamps and headers.

#### Creating a Snapshot

```sh
nats stream backup ORDERS '/data/js-backup/backup1'
```

#### Example Output

```plaintext
Starting backup of Stream "ORDERS" with 13 data blocks
2.4 MiB/s [==============================================================
Received 13 MiB bytes of compressed data in 3368 chunks for stream "ORDER
```

#### Restoring a Snapshot

```sh
nats stream restore '/data/js-backup/backup1'
```

#### Example Output

```plaintext
Starting restore of Stream "ORDERS" from file "/data/js-backup/backup1"
13 MiB/s [===============================================================
Restored stream "ORDERS" in 937.071149ms
```

#### Full Account Backup

```sh
nats account backup /data/js-backup
```

#### Example Output

```plaintext
15:56:11 Creating JetStream backup into /data/js-backup
15:56:11 Stream ORDERS to /data/js-backup/stream_ORDERS.json
15:56:11 Consumer ORDERS > NEW to /data/js-backup/stream_ORDERS_consumer_
15:56:11 Configuration backup complete
```

## Encryption at Rest

Supported since NATS server version 2.3.0. The NATS server can be configured to encrypt message blocks which include message headers and payloads. 

### Enabling Encryption

```plaintext
jetstream: {
  cipher: chachapoly,
  key: "6dYfBV0zzEkR3vxZCNjxmnVh/aIqgid1"
}
```

### Example with Environment Variable

```plaintext
jetstream: {
  cipher: chachapoly,
  key: $JS_KEY
}
```

```sh
JS_KEY="mykey" nats-server -c js.conf
```

### Changing Encryption Settings

- Enabling with existing data is supported, but existing unencrypted message blocks will not be re-encrypted.
- If encryption was enabled and the server is restarted with a different key or disabled, the server will fail to decrypt messages.
- Changing the cipher while using the same key is supported.

```plaintext
Error decrypting our stream metafile: chacha20poly1305: message authentic
```

## Managing JWT Security

If you are using the JWT model of authentication to secure your NATS infrastructure or implementing an Auth callout service, you can administer authentication and authorization without changing the servers' configuration files.

### Creating, Updating, and Managing JWTs Programmatically

You can use the `nsc` CLI tool to manage identities. Identities take the form of `nkeys`, which are a public-key signature system based on Ed25519 for the NATS ecosystem. 

### Example: Creating a User in Golang

```go
package main

import (
  "flag"
  "fmt"
  "log"

  "github.com/nats-io/jwt/v2"
  "github.com/nats-io/nkeys"
)

func main() {
  log.SetFlags(0)

  var (
    accountSeed  string
    operatorSeed string
    name         string
  )

  flag.StringVar(&operatorSeed, "operator", "", "Operator seed for creating account")
  flag.StringVar(&accountSeed, "account", "", "Account seed for creating user")
  flag.StringVar(&name, "name", "", "Account or user name to be created

")
  flag.Parse()

  if accountSeed != "" && operatorSeed != "" {
    log.Fatal("operator and account cannot both be provided")
  }

  var (
    jwt string
    err error
  )

  if operatorSeed != "" {
    jwt, err = createAccount(operatorSeed, name)
  } else if accountSeed != "" {
    jwt, err = createUser(accountSeed, name)
  } else {
    flag.PrintDefaults()
    return
  }
  if err != nil {
    log.Fatalf("error creating account JWT: %v", err)
  }

  fmt.Println(jwt)
}

func createAccount(operatorSeed, accountName string) (string, error) {
  akp, err := nkeys.CreateAccount()
  if err != nil {
    return "", fmt.Errorf("unable to create account using nkeys: %w", err)
  }

  apub, err := akp.PublicKey()
  if err != nil {
    return "", fmt.Errorf("unable to retrieve public key: %w", err)
  }

  ac := jwt.NewAccountClaims(apub)
  ac.Name = accountName

  okp, err := nkeys.FromSeed([]byte(operatorSeed))
  if err != nil {
    return "", fmt.Errorf("unable to create operator key pair from seed: %w", err)
  }

  ajwt, err := ac.Encode(okp)
  if err != nil {
    return "", fmt.Errorf("unable to sign the claims: %w", err)
  }
  return ajwt, nil
}

func createUser(accountSeed, userName string) (string, error) {
  ukp, err := nkeys.CreateUser()
  if err != nil {
    return "", fmt.Errorf("unable to create user using nkeys: %w", err)
  }

  upub, err := ukp.PublicKey()
  if err != nil {
    return "", fmt.Errorf("unable to retrieve public key: %w", err)
  }

  uc := jwt.NewUserClaims(upub)
  uc.Name = userName

  akp, err := nkeys.FromSeed([]byte(accountSeed))
  if err != nil {
    return "", fmt.Errorf("unable to create account key pair from seed: %w", err)
  }

  ujwt, err := uc.Encode(akp)
  if err != nil {
    return "", fmt.Errorf("unable to sign the claims: %w", err)
  }
  return ujwt, nil
}
```

### Deleting Accounts

Use the `$SYS.REQ.CLAIMS.DELETE` subject and enable JWT deletion in your `nats-server` resolver (`allow_delete: true` in the resolver stanza of the server configuration).

### Notes on NKEYs and JWTs

- NKEYs are a public-key signature system based on Ed25519.
- JWTs are digitally signed by the private key of an issuer forming a chain of trust.

## Installing NATS Tools

```sh
export GO111MODULE=on
go install github.com/nats-io/nats-server/v2@latest
go install github.com/nats-io/natscli/nats@latest
go install github.com/nats-io/nkeys/nk@latest
go install github.com/nats-io/nsc/v2@latest
```

### Starting the NATS Server

```sh
nats-server -c server.conf
```

### Reloading Configuration

```sh
nats-server --signal reload
```

### Example Configuration File

```plaintext
accounts: {
  A: {
    users: [{user: a, password: a}]
  },
  B: {
    users: [{user: b, password: b}]
  }
}
```

### Example Subscription and Publishing

```sh
nats -s nats://a:a@localhost:4222 sub ">"
nats -s nats://b:b@localhost:4222 pub "foo" "user b"
nats -s nats://a:a@localhost:4222 pub "foo" "user a"
```

### Modifying Configuration

Modify `server.conf` and run `nats-server --signal reload`.

### Advanced Configuration with Imports and Exports

```plaintext
accounts: {
  A: {
    users: [{user: a, password: a}],
    imports: [{stream: {account: B, subject: "foo"}}]
  },
  B: {
    users: [{user: b, password: b}],
    exports: [{stream: "foo"}]
  }
}
```

### Example Subscription with Import

```sh
nats -s nats://a:a@localhost:4222 sub ">"
nats -s nats://b:b@localhost:4222 pub "foo" "user b"
```

```plaintext
18:28:25 [#1] Received on "foo"
user b
```

## NKEYs Overview

- NKEYs are decorated, Base32 encoded, CRC16 check-summed, Ed25519 keys.
- Ed25519 is a public key signature system resistant to side channel attacks.

### Key Takeaways for NKEYs

- NKEYs provide secure identity management for NATS.
- NKEYs can sign and verify signatures securely.

---

This guide provides a comprehensive overview of managing and monitoring your NATS server infrastructure, including the use of the `nats` CLI tool, configuring monitoring endpoints, managing JetStream, handling disaster recovery, and securing your NATS infrastructure with JWTs and NKEYs.
#### Configuring NATS Server with NKEYs

NATS server can be configured with public NKEYs as user identities. When a client connects, the NATS server sends a challenge for the client to sign to prove it possesses the corresponding private key. The server verifies the signed challenge, ensuring the secret never leaves the client.

### Generating and Viewing NKEYs

To assist with identifying key types in configuration or logs, keys are decorated as follows:

- **Public Keys**: Have a one-byte prefix (O for operator, A for account, U for user).
- **Private Keys**: Have a two-byte prefix (SO, SA, SU) where S stands for seed and the second character matches the public key type.

#### Generate a User NKEY

```sh
nk -gen user -pubout > a.nk
```

View the generated key:

```sh
cat a.nk
```

Example output:

```plaintext
SUAAEZYNLTEA2MDTG7L5X7QODZXYHPOI2LT2KH5I4GD6YVP24SE766EGPA
UC435ZYS52HF72E2VMQF4GO6CUJOCHDUUPEBU7XDXW5AQLIC6JZ46PO5
```

Generate another key:

```sh
nk -gen user -pubout > b.nk
```

View the generated key:

```sh
cat b.nk
```

Example output:

```plaintext
SUANS4XLL5NWBTM57GSVHLN4TMFW55WGGWNI5YXXSIOYFJQYFVNHJK5GFY
UARZVI6JAV7YMJTPRANXANOOW4K3ZCD45NYP6S7C7XKCBHPVN2TFZ7ZC
```

### Configuring NATS Server with NKEYs

Replace the user/password with NKEY in account configuration:

```plaintext
accounts: {
  A: {
    users: [{nkey:UC435ZYS52HF72E2VMQF4GO6CUJOCHDUUPEBU7XDXW5AQLIC6JZ46PO5}],
    imports: [{stream: {account: B, subject: "foo"}}]
  },
  B: {
    users: [{nkey:UARZVI6JAV7YMJTPRANXANOOW4K3ZCD45NYP6S7C7XKCBHPVN2TFZ7ZC}],
    exports: [{stream: "foo"}]
  }
}
```

### Using NKEYs for Authentication

Subscribe with NATS CLI:

```sh
nats -s nats://localhost:4222 sub --nkey=a.nk ">"
```

Publish a message using another key:

```sh
nats -s nats://localhost:4222 pub --nkey=b.nk foo "nkey"
```

The subscriber should receive the message. If the NATS server is started with `-V` tracing, you can see the signature in the `CONNECT` message.

### Telnet Connection Example

On connect, clients are instantly sent the nonce to sign as part of the INFO message:

```sh
telnet localhost 4222
```

Example output:

```plaintext
INFO {"auth_required": true, ... "nonce": "-QPTE1Jsk8kI3rE", ...}
-ERR 'Authentication Timeout'
Connection closed by foreign host.
```

### Key Takeaways

- NKEYs provide secure client authentication.
- Private keys are never stored or accessed by the NATS server.
- Public keys need to be configured on the NATS server.

---

## JSON Web Tokens (JWT) for NATS

JWTs are used for authentication and authorization in NATS, enabling decentralized security management.

### NATS JWT Hierarchy

- **Operator**: Manages NATS servers in the same authentication domain.
- **Account**: Manages configuration for a single account.
- **User**: Manages individual user configurations.

### Decentralized Authentication/Authorization

JWTs carry configuration and are signed by the private key of an issuer in a hierarchical chain of trust.

#### Example JWT Structure

- **Operator JWT**: Contains server configuration.
- **Account JWT**: Contains account-specific configurations like exports, imports, and limits.
- **User JWT**: Contains user-specific configurations like permissions and limits.

### JWT Resolver Types

1. **Mem-resolver**: For few or static accounts. Requires server configuration changes for updates.
2. **URL-resolver**: For large volumes of accounts. Downloads accounts from a web server.
3. **NATS-resolver**: Uses NATS instead of HTTP. Recommended for production environments.

### Example Configuration for URL-resolver

```plaintext
operator: ./trustedOperator.jwt
resolver: URL(http://localhost:9090/jwt/v1/accounts/)
```

### JWT and Chain of Trust Verification

JWTs are hierarchically organized and verified top-down. Each JWT has a subject (public identity NKEY) and an issuer. JWTs are signed using the issuer's private key.

### Deployment Models

1. **Centralized Config**: One set of users manage all private operator and account NKEYs.
2. **Decentralized Config**: Teams manage their respective private account NKEYs.
3. **Self-service Config**: Teams manage their own accounts with operator signing keys.

### Example: Creating an Account with NSC

Create an operator and account:

```sh
nsc add operator -n <operator-name> --sys
nsc add account -n <account-name>
```

Generate a signing key for the account:

```sh
nsc generate nkey -a --store
nsc edit account --sk <account-signing-key>
```

Create a user:

```sh
nsc add user -a <account-name> -n <user-name>
```

Export the user credentials:

```sh
nsc generate creds -a <account-name> -n <user-name>
```

### Conclusion

NATS JWTs provide a secure and flexible way to manage authentication and authorization. By leveraging NKEYs and JWTs, NATS ensures secure, decentralized, and scalable security management.
#### Create/Edit Account - All Environments - All Deployment Modes

To create or edit an account using `nsc`, follow these steps:

1. **Create an account**:
   
   ```sh
   nsc add account -n <account name> -i
   ```

2. **Edit an account**:
   
   ```sh
   nsc edit account [flags]
   ```

#### Export Account - Non Operator/Administrator Environment - Decentralized Deployment Modes

If an account needs to be updated, repeat the steps and provide the `--force` option to overwrite the stored account JWT.

#### Push Account to NATS Network

To publicize an account, the method depends on the resolver type you are using:

- **mem-resolver**: The operator imports all accounts and generates a new configuration.
- **url-resolver**: Use `nsc push` to send an HTTP POST request to the hosting webserver.
- **nats-resolver**: Push account JWTs using `$SYS.REQ.CLAIMS.UPDATE` or `$SYS.REQ.ACCOUNT.*.CLAIMS.UPDATE`.

To push an account or all accounts:

```sh
nsc push -a TEST
nsc push --all
```

#### Example: NATS-based Resolver

1. **Add an operator**:

   ```sh
   nsc add operator -n DEMO --sys
   ```

   Output:

   ```plaintext
   [ OK ] generated and stored operator key "ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV7F7KUWADCM4K6"
   [ OK ] added operator "DEMO"
   [ OK ] created system_account: name:SYS id:AA6W5MRDIFIQWE6UE6D4YWQT5L4YZG7ZRHSKYCPF2VIEMUHRZH3VQZ27
   [ OK ] created system account user: name:sys id:UABM73CE5F3ZYFNC3ZDODAF7GIB62W2WXV5DOLMYLGEW4MEHYBC46PN4
   [ OK ] system account user creds file stored in `~/test/demo/env1/keys/cr`
   ```

2. **Edit the operator to set the account JWT server URL**:

   ```sh
   nsc edit operator --account-jwt-server-url nats://localhost:4222
   ```

   Output:

   ```plaintext
   [ OK ] set account jwt server url to "nats://localhost:4222"
   [ OK ] edited operator "DEMO"
   ```

3. **List keys**:

   ```sh
   nsc list keys --all
   ```

4. **Describe the operator**:

   ```sh
   nsc describe operator
   ```

   Output:

   ```plaintext
   +------------------------------------------------------------------------
   | Operator Details
   +--------------------+---------------------------------------------------
   | Name               | DEMO
   | Operator ID        | ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV7F7KUWA
   | Issuer ID          | ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV7F7KUWA
   | Issued             | 2020-11-04 19:25:25 UTC
   | Expires            |
   | Account JWT Server | nats://localhost:4222
   | System Account     | AA6W5MRDIFIQWE6UE6D4YWQT5L4YZG7ZRHSKYCPF2VIEMUHRZH
   +--------------------+---------------------------------------------------
   ```

5. **Generate configuration and start the server**:

   ```sh
   nsc generate config --nats-resolver > nats-res.cfg
   nats-server -c nats-res.cfg --addr localhost --port 4222 &
   ```

6. **Add an account and user for testing**:

   ```sh
   nsc add account -n TEST
   nsc add user -a TEST -n foo
   ```

7. **Push the account**:

   ```sh
   nsc push -a TEST
   nsc push --all
   ```

### Automated Sign-up Services - JWT and NKEY Libraries

1. **Generate User Key**:

   ```go
   func generateUserKey() (userPublicKey string, userSeed []byte, userKeyPair nkeys.KeyPair, err error) {
       kp, err := nkeys.CreateUser()
       if err != nil {
           return "", nil, nil, err
       }
       userSeed, err = kp.Seed()
       if err != nil {
           return "", nil, nil, err
       }
       userPublicKey, err = kp.PublicKey()
       if err != nil {
           return "", nil, nil, err
       }
       return userPublicKey, userSeed, kp, nil
   }
   ```

2. **Generate User JWT**:

   ```go
   func generateUserJWT(userPublicKey, accountPublicKey string, accountSigningKey nkeys.KeyPair) (string, error) {
       uc := jwt.NewUserClaims(userPublicKey)
       uc.Pub.Allow.Add("subject.foo") // Allow publishing to subject.foo
       uc.Expires = time.Now().Add(time.Hour).Unix() // Expires in an hour
       uc.IssuerAccount = accountPublicKey
       vr := jwt.ValidationResults{}
       uc.Validate(&vr)
       if vr.IsBlocking(true) {
           return "", errors.New("generated user claim is invalid")
       }
       userJWT, err := uc.Encode(accountSigningKey)
       if err != nil {
           return "", err
       }
       return userJWT, nil
   }
   ```

3. **Request User**:

   ```go
   func RequestUser() {
       // Setup! Obtain the account signing key!
       accountPublicKey := GetAccountPublicKey()
       accountSigningKey := GetAccountSigningKey()
       userPublicKey, userSeed, userKeyPair, _ := generateUserKey()
       userJWT := generateUserJWT(userPublicKey, accountPublicKey, accountSigningKey)
       // userJWT and userKeyPair can be used in conjunction with this nats.Option
       jwtAuthOption := nats.UserJWT(func() (string, error) {
           return userJWT, nil
       }, func(bytes []byte) ([]byte, error) {
           return userKeyPair.Sign(bytes)
       })
       // Alternatively, create a creds file
       credsContent, _ := jwt.FormatUserConfig(userJWT, userSeed)
       ioutil.WriteFile("my.creds", credsContent, 0644)
       jwtAuthOption = nats.UserCredentials("my.creds")
       nc, err := nats.Connect("nats://localhost:4222", jwtAuthOption)
       if err != nil {
           log.Fatal(err)
       }
       defer nc.Close()
       // Use the connection
   }
   ```

### System Account

The system account is used for administrative purposes and should not facilitate communication between applications. It is recommended to configure the system account using the public NKEY of the account you want to be the system account.

```plaintext
system_account: AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLKU7A5
```

#### Event Subjects

Events are published as they happen, but the order is not guaranteed. Some messages carry aggregate data and are periodically emitted.

Example subjects:

- `$SYS.SERVER.<server-id>.SHUTDOWN`
- `$SYS.SERVER.<server-id>.CLIENT.AUTH.ERR`
- `$SYS.SERVER.<server-id>.STATSZ`

#### Service Subjects

Service subjects are used to publish requests and receive responses.

Example subjects:

- `$SYS.REQ.SERVER.PING.STATZ`
- `$SYS.REQ.SERVER.<server-id>.VARZ`

Each subject can be used without input, but JSON with type-specific options can be sent.

### Operator Mode

#### Connecting Accounts

To connect accounts via exports and imports using JWTs, follow these steps:

1. **Add an export**:
   
   ```sh
   nsc add export --name <export name> --subject <export subject>
   ```

   - To export a service, add `--service`.
   - To make the export private, add `--private`. Generate a token for the private export:

   ```sh
   nsc generate activation --account <account name> --subject <export subject> --output-file <token file> --target-account <account identity public NKEY>
   ```

2. **Add an import**:
   
   - For a public export:
     
     ```sh
     nsc add import --account <account name> --src-account <account identity public NKEY> --remote-subject <subject of export>
     ```

   - For a service, add `--service`.
   - For a private export:

     ```sh
     nsc add import --account <account name> --token <token file or url>
     ```

#### Import Remapping

To remap an imported subject:

```sh
nsc add import --account <account name> --src-account <account identity public NKEY> --remote-subject <subject name> --local-subject <new subject name>
```

#### Visualizing Export/Import Relationships

Generate a diagram of inter-account relationships:

```sh
nsc generate diagram component --output-file test.uml
```

Convert the diagram to PNG:

```sh
plantuml -tpng test.uml -Xmx2048m -DPLANTUML_LIMIT_SIZE=16384
```

### Managing Keys

#### Protecting Identity NKEYs

1. **Backup identity NKEYs offline**.
2. **Use signing keys** for operators and accounts.
3. **Remove private identity NKEYs from the nsc directory**:
   
   ```sh
   nsc list keys --all
   nsc env
   ```

4. **Reissue Identity NKEYs**:
   
   - **Operator**:
     
     ```sh
     nsc reissue operator
     ```

   - **Convert old identity NKEY to a signing key**:
     
     ```sh
     nsc reissue operator --convert-to-signing-key
     ```

#### Revocations

1. **Revoke User JWTs**:
   
   ```sh
   nsc revocations add-user --account <account name> --name <user name>
   ```

   - List revoked users:
     
     ```sh
     nsc revocations list-users --account <account name>
     ```

2. **Revoke Activation Tokens**:
   
   ```sh
   nsc revocations add-activation --account <account name> --subject <export subject> --target-account <account identity public NKEY>
   ```

   - List revoked activations:
     
     ```sh
     nsc revocations list-activations --account <account name>
     ```

3. **Revoke Accounts**:
   
   ```sh
   nsc edit account --name <account name> --conns 0
   nsc push --all
   ```

   - Remove the account JWT depending on resolver type:
     
     - **mem-resolver**: Remove from configuration field `resolver_preload` and restart `nats-server`.
     - **url-resolver**: Manually delete the JWT from the `nats-account-server` store directory.
     - **nats-resolver**: Prune removed accounts:

       ```sh
       nsc push --all --prune
       ```

#### Removing Signing Keys

1. **Remove an operator signing key**:
   
   ```sh
   nsc edit operator --rm-sk <signing key>
   ```

2. **Remove an account signing key**:
   
   ```sh
   nsc edit account --name <account name> --rm-sk <signing key>
   nsc push --all
   ```

### Upgrading a Cluster

1. **Set up initial servers**:

   ```sh
   nats-server -D -p 4222 -cluster nats://localhost:6222 -routes nats://localhost:6222
   nats-server -D -p 4333 -cluster nats://localhost:6333 -routes nats://localhost:6222
   ```

2. **Add a new server for rolling upgrade**:

   ```sh
   nats-server -D -p 4444 -cluster nats://localhost:6444 -routes nats://localhost:6222
   ```

3. **Client to observe server movement**:

   ```sh
   nats sub -s nats://localhost:4222 ">"
   ```

4. **Upgrade process**:
   - Stop, update, and restart each server one at a time.
   - Monitor client reconnections to other servers.

5. **After upgrading**:
   - Turn off the temporary server `C`.

### Slow Consumers

#### Handling Slow Consumers in Clients

1. **Define and register an asynchronous error handler**:

   ```go
   func natsErrHandler(nc *nats.Conn, sub *nats.Subscription, natsErr error) {
       fmt.Printf("error: %v\n", natsErr)
       if natsErr == nats.ErrSlowConsumer {
           pendingMsgs, _, err := sub.Pending()
           if err != nil {
               fmt.Printf("couldn't get pending messages: %v", err)
               return
           }
           fmt.Printf("Falling behind with %d pending messages on subject %q\n", pendingMsgs, sub.Subject)
           // Log error, notify operations...
       }
       // check for other errors
   }

   nc, err := nats.Connect("nats://localhost:4222", nats.ErrorHandler(natsErrHandler))
   ```

2. **Set the pending limits**:

   ```go
   if err := sub.SetPendingLimits(1024 * 500, 1024 * 5000); err != nil {
       log.Fatalf("Unable to set pending limits: %v", err)
   }
   ```

#### Handling Slow Consumers in Servers

- **Server output**:
  
  ```plaintext
  [54083] 2017/09/28 14:45:18.001357 [INF] ::1:63283 - cid:7 - Slow Consumer
  ```

- **Server monitoring endpoint**:
  
  - `varz` endpoint for slow consumer count.

- **Tuning NATS Server**:

  ```sh
  write_deadline: 2s
  ```

### Signals

#### Sending Signals to NATS Server

- **Quit the server**:

  ```sh
  nats-server --signal quit
  ```

- **Stop the server**:

  ```sh
  nats-server --signal stop
  ```

- **Lame duck mode**:

  ```sh
  nats-server --signal ldm
  ```

- **Reopen log file for log rotation**:

  ```sh
  nats-server --signal reopen
  ```

- **Reload server configuration**:

  ```sh
  nats-server --signal reload
  ```

### Profiling

#### Enabling Profiling

1. **Set the prof_port**:

   ```sh
   prof_port = 65432
   ```

2. **Download memory profile**:

   ```sh
   curl -o mem.prof http://localhost:65432/debug/pprof/allocs
   ```

3. **Download CPU profile**:

   ```sh
   curl -o cpu.prof http://localhost:65432/debug/pprof/profile?seconds=30
   ```
