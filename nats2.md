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


#### Operator Mode

### Connecting Accounts


#### As shown in what are accounts, they can be connected via exports and imports. While in

#### conguration les this is straight forward, this becomes a bit more complicated when

#### using JWTs. In part this is due to the addition of new concepts such as

#### public/private/activation tokens that do not make sense in a cong based context.

#### Add an export with:

#### nsc add export --name <export name> --subject <export subject> This will

#### export a public stream that can be imported by any account. To alter the export to be a

#### service add --service.

#### To have more control over which account is allowed to import provide the option

#### --private. When doing so only accounts for which you generate tokens can add the

#### matching import. A token can be generated and stored in a le as follows:

```
nsc generate activation --account <account name> --subject <export
subject> --output-file <token file> --target-account <account identity
public NKEY>
```
#### The resulting le can then be exchanged with the importer.

#### To add an import for a public export use

```
nsc add import --account <account name> --src-account <account identity
public NKEY> --remote-subject <subject of export>
```
#### . To import a service provide the option --service.

#### To add an import for a private export use

#### nsc add import --account <account name> --token <token file or url> If your

#### nsc environment contains operator and account signing NKEYs, nsc add import -i

#### will generate token to embed on the y

#### Between export/import/activation tokens there are many subjects in use. Their

#### relationship is as follows:

#### Exports

#### Imports

#### Import Subjects


#### In order to be independent of subject names chosen by the exporter, importing allows to

#### remap the imported subject. To do so provide the option

#### --remote-subject <subject name> to the import command.

#### This example will change the subject name the importing account uses locally from the

#### exporter picked subject foo to bar.

#### NSC can generate diagrams of inter account relationships using:

#### nsc generate diagram component --output-file test.uml The generated le

#### contains a plantuml component diagram of all accounts connected through their

#### exports/imports. To turn the le into a .png execute: plantuml -tpng test.uml If the

#### diagram is cut off, increase available memory and image size limit with these options:

```
-Xmx2048m -DPLANTUML_LIMIT_SIZE=16384
```
#### Identity keys are extremely important, so you may want to keep them safe and instead

#### hand out more easily replaceable signing keys to operators. Key importance generally

#### follows the chain of trust with operator keys being more important than account keys.

#### Furthermore, identity keys are more important than signing keys.

#### Import subject is identical to or a subset of the exported subject.

#### An activation token's subject is identical to or a subset of the exported subject.

#### An activation token's subject is also identical to or a subset of the import subject of

#### the account it is embedded in.

```
nsc add import --account test --src-account ACJ6G45BE7LLOFCVAZSZR3RY4XELX
```
```
[ OK ] added stream import "blo"
```
#### Import Remapping

#### Visualizing Export/Import Relationships

### Managing Keys


#### There are instances where regenerating a completely new identity key of either type is not

#### a feasible option. For example, you might have an extremely large deployment (IoT)

#### where there is simply too much institutional overhead. In this case, we suggest you

#### securely backup identity keys oine and use exchangeable signing keys instead.

#### Depending on which key was compromised, you may have to exchange signing keys and

#### re-sign all JWTs signed with the compromised key. The compromised key may also have

#### to be revoked.

#### Whether you simply plan to regenerate new NKEY/JWT or exchange signing NKEYs and

#### re-sign JWTs, in either case, you need to prepare and try this out beforehand and not wait

#### until disaster strikes.

#### Usage of signing keys for Operator and Account has been shown in the nsc section.

#### This shows how to take an identity key oine. Identity NKEY of the operator/account is

#### the only one allowed to modify the corresponding JWT and thus add/remove signing

#### keys. Thus, initial signing keys are best created and assigned prior to removing the

#### private identity NKEY.

#### Basic strategy: take them oine & delete in nsc NKEY directory.

#### Use nsc env to determine your NKEY directory. (Assuming ~/.nkeys for this example)

#### nsc list keys --all lists all keys under your operator and indicates if they are

#### present and if they are signing keys.

#### Keys for your Operator/Account can be found under

#### <nkyesdir>/keys/O/../<public-nkey>.nk or

#### <nkyesdir>/keys/A/../<public-nkey>.nk. The operator identity NKEY

#### ODMFND7EIJ2MBHNPO2JHCKOZIAY6NAK7OT4V2ZT2C5O6LEB3DPKYV3QL would

#### reside under

```
~/.nkeys/keys/O/DM/ODMFND7EIJ2MBHNPO2JHCKOZIAY6NAK7OT4V2ZT2C5O6LEB3DPKYV3Q
L.nk
```
#### .

#### Please note that key storage is sharded by the 2nd and 3rd letter in the key

#### Protect Identity NKEYs


#### Once these les are backed up and deleted nsc list keys --all will show them as

#### not stored. You can continue as normal, nsc will pick up signing keys instead.

#### Since you typically distribute user keys or creds les to your applications, there is no need

#### for nsc to hold on to them in the rst place. Credentials les are a concatenated user

#### JWT and the corresponding private key, so don't forget to delete that as well.

#### Key and creds can be found under <nkyesdir>/keys/U/../<public-nkey>.nk and

```
<nkyesdir>/creds/<operator-name>/<account-name>/<user-name>.creds
```
#### If you can easily re-deploy all necessary keys and JWTs, simply by re-generating a new

#### account/user (possibly operator) this will be the simplest solution. The steps necessary

#### are identical to the initial setup, which is why it would be preferred. In fact, for user NKEYs

#### and JWT, generating and distributing new ones to affected applications is the best option.

#### Even if regeneration of an account or operator is not your rst choice, it may be your

#### method of last resort. Below sections outline the steps this would entail.

#### In order to reissue an operator identity NKEY use nsc reissue operator. It will

#### generate a new identity NKEY and use it to sign the operator. nsc will also re-sign all

#### accounts signed by the original identity NKEY. Accounts signed by operator signing keys

#### will remain untouched.

#### The altered operator JWT will have to be deployed to all affected nats-server (one

#### server at a time). Once all nats-server have been restarted with the new operator, push

#### the altered accounts. Depending on your deployment mode you may have to distribute

#### the operator JWT and altered account JWT to all other nsc environments.

#### This process will be a lot easier when operator signing keys were used throughout and no

#### account will be re-signed because of this. If they were not, you can convert the old

#### identity NKEY into a signing key using

#### nsc reissue operator --convert-to-signing-key. On your own time - you can then

#### Reissue Identity NKEYs

#### Operator


#### remove the then signing NKEY using nsc edit operator --rm-sk O.. and redeploy

#### the operator JWT to all nats-server.

#### Unlike with the operator, account identity NKEYs can not be changed as easily. User JWT

#### explicitly reference the account identity NKEY such that the nats-server can download

#### them via a resolver. This complicates reissuing these kind of NKEYs, which is why we

#### strongly suggest sticking to signing keys.

#### The basic approach is to:

#### 1. generate a new account with similar settings - including signing NKEYs,

#### 2. re-sign all users that used to be signed by the old identity NKEY,

#### 3. push the account and,

#### 4. deploy the new user JWT to all programs running inside the account.

#### When signing keys were used, the account identity NKEY would only be needed to self-

#### sign the account JWT exchange with an administrators/operators nsc environment.

#### JWTs for user, activations and accounts can be explicitly revoked. Furthermore, signing

#### keys can be removed, thus invalidating all JWTs signed by the removed NKEY.

#### To revoke all JWTs for a user in a account issue

#### nsc revocations add-user --account <account name> --name <user name>.

#### With the argument --at you can specify a time different than now. Use

#### nsc revocations list-users --account <account name> to inspect the result or

#### nsc revocations delete-user --account <account name> --name <user name> to

#### remove the revocation.

```
nsc revocations add-user --account SYS --name sys
```
#### Account

#### Revocations

#### User


#### Please note that the revocation created only applies to JWTs issued before the time

#### listed. Users created or updated after revocation will be valid as they are outside of the

#### revocation time. Also, please be aware that adding a revocation will modify the account

#### and therefore has to be pushed in order to publicize the revocation.

#### To revoke all activations of the export, identied by --account and --subject (

#### --stream if the export is a stream), issued for a given Account identity NKEY use:

#### Use nsc revocations list-activations --account SYS to inspect the result or:

#### to remove the revocation.

```
[ OK ] revoked user "UCL5YXXUKCEO4HDTTYUOHDMHP4JJ6MGE3SVQBDWFZUGJUMUKE24D
```
```
nsc revocations list-users
```
###### +------------------------------------------------------------------------

```
| Revoked Users for test5
+----------------------------------------------------------+-------------
| Public Key | Revoke Crede
+----------------------------------------------------------+-------------
| UAX7KQJJNL5NIRTSQSANKE3DNBHLLFUYKRXCD5QRKI75XBEHQOA4ZZGV | Wed, 10 Feb
+----------------------------------------------------------+-------------
```
```
nsc revocations add-activation --account <account name> --subject <export
--target-account <account identity public NKEY>
```
```
nsc revocations delete_activation --account <account name> \
--subject <export name> --target-account <account identity public NKEY>
```
```
nsc revocations add-activation --account SYS --subject foo \
--target-account AAUDEW26FB4TOJAQN3DYMDLCVXZMNIJWP2EMOAM5HGKLF6RGMO2PV7W
```
```
[ OK ] revoked activation "foo" for account AAUDEW26FB4TOJAQN3DYMDLCVXZMN
```
#### Activations


#### Please note the revocation created only applies to JWTs issued before the time listed.

#### Activations created or edited after, will be valid as they are outside of the revocation time.

#### Also be aware that adding a revocation will modify the account and therefore has to be

#### pushed in order to publicize the revocation.

#### Account identity NKEYS can not be revoked like user or activations. Instead lock out all

#### users by setting the connection count to 0 using

#### nsc edit account --name <account name> --conns 0 and pushing the change using

#### nsc push --all.

#### Alternatively you can also remove the account using nsc delete account --name and

#### keep it from found by the account resolver. How to do this depends on your resolver type:

```
nsc revocations list-activations --account SYS
```
###### +------------------------------------------------------------------------

```
| Revoked Accounts for stream foo
+----------------------------------------------------------+-------------
| Public Key | Revoke Crede
+----------------------------------------------------------+-------------
| AAUDEW26FB4TOJAQN3DYMDLCVXZMNIJWP2EMOAM5HGKLF6RGMO2PV7WP | Wed, 10 Feb
+----------------------------------------------------------+-------------
```
#### mem-resolver:

#### Remove the JWT from the conguration eld resolver_preload and restart all

```
nats-server
```
#### url-resolver:

#### Manually delete the JWT from the nats-account-server store directory.

#### nats-resolver: Prune removed accounts using: nsc push --all --prune.

#### For this to work, the resolver has to have deletion enabled (allow_delete: true)

#### and you need to be in possession of an operator signing key.

#### Accounts

#### Signing keys


#### Accounts, Activations, and Users can be revoked in bulk by removing the respective

#### signing key.

#### Remove an operator signing key: nsc edit operator --rm-sk <signing key> As a

#### modication of the operator, in order to take effect, all dependent nsc installations as

#### well as nats-server will need this new version of the operator JWT.

#### Remove an account signing key:

#### nsc edit account --name <account name> --rm-sk <signing key>. In order to take

#### effect, a modication of an account needs to be pushed: nsc push --all.


# Upgrading a Cluster

#### The basic strategy for upgrading a cluster revolves around the server's ability to gossip

#### cluster conguration to clients and other servers. When cluster conguration changes,

#### clients become aware of new servers automatically. In the case of a disconnect, a client

#### has a list of servers that joined the cluster in addition to the ones it knew about from its

#### connection settings.

#### Note that since each server stores it's own permission and authentication conguration,

#### new servers added to a cluster should provide the same users and authorization to

#### prevent clients from getting rejected or gaining unexpected privileges.

#### For purposes of describing the scenario, let's get some ngers on keyboards, and go

#### through the motions. Let's consider a cluster of two servers: 'A' and 'B', and yes - clusters

#### should be three to v e servers, but for purposes of describing the behavior and cluster

#### upgrade process, a cluster of two servers will suce.

#### Let's build this cluster:

#### The command above is starting nats-server with debug output enabled, listening for

#### clients on port 4222, and accepting cluster connections on port 6222. The -routes

#### option species a list of nats URLs where the server will attempt to connect to other

#### servers. These URLs dene the cluster ports enabled on the cluster peers.

#### Keen readers will notice a self-route. The NATS server will ignore the self-route, but it

#### makes for a single consistent conguration for all servers.

#### You will see the server started, we notice it emits some warnings because it cannot

#### connect to 'localhost:6333'. The message more accurately reads:

#### Let's x that, by starting the second server:

```
nats-server -D -p 4222 -cluster nats://localhost:6222 -routes nats://loca
```
```
Error trying to connect to route: dial tcp localhost:6333: connect: conn
```

#### The second server was started on port 4333 with its cluster port on 6333. Otherwise the

#### same as 'A'.

#### Let's get one client, so we can observe it moving between servers as servers get

#### removed:

#### After starting the subscriber you should see a message on 'A' that a new client

#### connected.

#### We have two servers and a client. Time to simulate our rolling upgrade. But wait, before

#### we upgrade 'A', let's introduce a new server 'C'. Server 'C' will join the existing cluster while

#### we perform the upgrade. Its sole purpose is to provide an additional place where clients

#### can go other than 'A' and ensure we don't end up with a single server serving all the clients

#### after the upgrade procedure. Clients will randomly select a server when connecting

#### unless a special option is provided that disables that functionality (usually called

#### 'DontRandomize' or 'noRandomize'). You can read more about "Avoiding the Thundering

#### Herd". Suce it to say that clients redistribute themselves about evenly between all

#### servers in the cluster. In our case 1/2 of the clients on 'A' will jump over to 'B' and the

#### remaining half to 'C'.

#### Let's start our temporary server:

#### After an instant or so, clients on 'A' learn of the new cluster member that joined. On our

#### hands-on tutorial, nats sub is now aware of 3 possible servers, 'A' (specied when we

#### started the tool) and 'B' and 'C' learned from the cluster gossip.

#### We invoke our admin powers and turn off 'A' by issuing a CTRL+C to the terminal on 'A'

#### and observe that either 'B' or 'C' reports that a new client connected. That is our

#### nats sub client.

```
nats-server -D -p 4333 -cluster nats://localhost:6333 -routes nats://loca
```
```
nats sub -s nats://localhost:4222 ">"
```
```
nats server -D -p 4444 -cluster nats://localhost:6444 -routes nats://loca
```

#### We perform the upgrade process, update the binary for 'A', and restart 'A':

#### We move on to upgrade 'B'. Notice that clients from 'B' reconnect to 'A' and 'C'. We

#### upgrade and restart 'B':

#### If we had more servers, we would continue the stop, update, restart rotation as we did for

#### 'A' and 'B'. After restarting the last server, we can go ahead and turn off 'C.' Any clients on

#### 'C' will redistribute to our permanent cluster members.

#### In the examples above we started nats-server specifying two clustering routes. It is

#### possible to allow the server gossip protocol drive it and reduce the amount of

#### conguration. You could for example start A, B and C as follows:

```
nats-server -D -p 4222 -cluster nats://localhost:6222 -routes nats://loca
```
```
nats-server -D -p 4333 -cluster nats://localhost:6333 -routes nats://loca
```
```
nats-server -D -p 4222 -cluster nats://localhost:6222
```
```
nats-server -D -p 4333 -cluster nats://localhost:6333 -routes nats://loca
```
```
nats-server -D -p 4444 -cluster nats://localhost:6444 -routes nats://loca
```
## Seed Servers

### A - Seed Server

### B

### C


#### Once they connect to the 'seed server', they will learn about all the other servers and

#### connect to each other forming the full mesh.

#### Although the NATS server goes through rigorous testing for each release, there may be a

#### need to revert to the previous version if you observe a performance regression for your

#### workload. The support policy for the server is the current release as well as one patch

#### version release prior. For example, if the latest is 2.8.4, a downgrade to 2.8.3 is

#### supported. Downgrades to earlier versions may work, but is not recommended.

#### Fortunately, the downgrade path is the same as the upgrade path as noted above. Swap

#### the binary and do a rolling restart.

## Downgrading


# Slow Consumers

#### To support resiliency and high availability, NATS provides built-in mechanisms to

#### automatically prune the registered listener interest graph that is used to keep track of

#### subscribers, including slow consumers and lazy listeners. NATS automatically handles a

#### slow consumer. If a client is not processing messages quick enough, the NATS server

#### cuts it off. To support scaling, NATS provides for auto-pruning of client connections. If a

#### subscriber does not respond to ping requests from the server within the ping-pong

#### interval, the client is cut off (disconnected). The client will need to have reconnect logic to

#### reconnect with the server.

#### In core NATS, consumers that cannot keep up are handled differently from many other

#### messaging systems: NATS favors the approach of protecting the system as a whole over

#### accommodating a particular consumer to ensure message delivery.

#### What is a slow consumer?

#### A slow consumer is a subscriber that cannot keep up with the message ow delivered

#### from the NATS server. This is a common case in distributed systems because it is often

#### easier to generate data than it is to process it. When consumers cannot process data fast

#### enough, back pressure is applied to the rest of the system. NATS has mechanisms to

#### reduce this back pressure.

#### NATS identies slow consumers in the client or the server, providing notication through

#### registered callbacks, log messages, and statistics in the server's monitoring endpoints.

#### What happens to slow consumers?

#### When detected at the client, the application is notied and messages are dropped to

#### allow the consumer to continue and reduce potential back pressure. When detected in the

#### server, the server will disconnect the connection with the slow consumer to protect itself

#### and the integrity of the messaging system.

## Slow consumers identified in the client


#### A client can detect it is a slow consumer on a local connection and notify the application

#### through use of the asynchronous error callback. It is better to catch a slow consumer

#### locally in the client rather than to allow the server to detect this condition. This example

#### demonstrates how to dene and register an asynchronous error handler that will handle

#### slow consumer errors.

#### With this example code and default settings, a slow consumer error would generate

#### output something like this:

#### Note that if you are using a synchronous subscriber,

#### Subscription.NextMsg(timeout time.Duration) will also return an error indicating

#### there was a slow consumer and messages have been dropped.

#### When a client does not process messages fast enough, the server will buffer messages in

#### the outbound connection to the client. When this happens and the server cannot write

```
func natsErrHandler(nc *nats.Conn, sub *nats.Subscription, natsErr error)
fmt.Printf("error: %v\n", natsErr)
if natsErr == nats.ErrSlowConsumer {
pendingMsgs, _, err := sub.Pending()
if err != nil {
fmt.Printf("couldn't get pending messages: %v", err)
return
}
fmt.Printf("Falling behind with %d pending messages on subject %q
pendingMsgs, sub.Subject)
// Log error, notify operations...
}
// check for other errors
}
```
```
// Set the error handler when creating a connection.
nc, err := nats.Connect("nats://localhost:4222",
nats.ErrorHandler(natsErrHandler))
```
```
error: nats: slow consumer, messages dropped
Falling behind with 65536 pending messages on subject "foo".
```
## Slow consumers identified by the server


#### data fast enough to the client, in order to protect itself, it will designate a subscriber as a

#### "slow consumer" and may drop the associated connection.

#### When the server initiates a slow consumer error, you'll see the following in the server

#### output:

#### The server will also keep count of the number of slow consumer errors encountered,

#### available through the monitoring varz endpoint in the slow_consumers eld.

#### Apart from using NATS streaming or optimizing your consuming application, there are a

#### few options available: scale, meter, or tune NATS to your environment.

#### Scaling with queue subscribers

#### This is ideal if you do not rely on message order. Ensure your NATS subscription belongs

#### to a queue group, then scale as required by creating more instances of your service or

#### application. This is a great approach for microservices - each instance of your

#### microservice will receive a portion of the messages to process, and simply add more

#### instances of your service to scale. No code changes, conguration changes, or downtime

#### whatsoever.

#### Create a subject namespace that can scale

#### You can distribute work further through the subject namespace, with some forethought in

#### design. This approach is useful if you need to preserve message order. The general idea

#### is to publish to a deep subject namespace, and consume with wildcard subscriptions

#### while giving yourself room to expand and distribute work in the future.

#### For a simple example, if you have a service that receives telemetry data from IoT devices

#### located throughout a city, you can publish to a subject namespace like Sensors.North,

#### Sensors.South, Sensors.East and Sensors.West. Initially, you'll subscribe to

```
[54083] 2017/09/28 14:45:18.001357 [INF] ::1:63283 - cid:7 - Slow Consume
```
## Handling slow consumers


#### Sensors.> to process everything in one consumer. As your enterprise grows and data

#### rates exceed what one consumer can handle, you can replace your single consumer with

#### four consuming applications to subscribe to each subject representing a smaller

#### segment of your data. Note that your publishing applications remain untouched.

#### Meter the publisher

#### A less favorable option may be to meter the publisher. There are several ways to do this

#### varying from simply slowing down your publisher to a more complex approach

#### periodically issuing a blocking request-reply to match subscriber rates.

#### Tune NATS through conguration

#### The NATS server can be tuned to determine how much data can be buffered before a

#### consumer is considered slow, and some ocially supported clients allow buffer sizes to

#### be adjusted. Decreasing buffer sizes will let you identify slow consumers more quickly.

#### Increasing buffer sizes is not typically recommended unless you are handling temporary

#### bursts of data. Often, increasing buffer capacity will only postpone slow consumer

#### problems.

#### The NATS server has a write deadline it uses to write to a connection. When this write

#### deadline is exceeded, a client is considered to have a slow consumer. If you are

#### encountering slow consumer errors in the server, you can increase the write deadline to

#### buffer more data.

#### The write_deadline conguration option in the NATS server conguration le will tune

#### this:

#### Tuning this parameter is ideal when you have bursts of data to accommodate. Be sure

#### you are not just postponing a slow consumer error.

```
write_deadline: 2s
```
### Server Configuration


#### Most ocially supported clients have an internal buffer of pending messages and will

#### notify your application through an asynchronous error callback if a local subscription is

#### not catching up. Receiving an error locally does not necessarily mean that the server will

#### have identied a subscription as a slow consumer.

#### This buffer can be congured through setting the pending limits after a subscription has

#### been created:

#### The default subscriber pending message limit is 65536 , and the default subscriber

#### pending byte limit is 65536*1024

#### If the client reaches this internal limit, it will drop messages and continue to process new

#### messages. This is aligned with NATS at most once delivery. It is up to your application to

#### detect the missing messages and recover from this condition.

```
if err := sub.SetPendingLimits( 1024 * 500 , 1024 * 5000 ); err != nil {
log.Fatalf("Unable to set pending limits: %v", err)
}
```
### Client Configuration


# Signals

#### On Unix systems, the NATS server responds to the following signals:

#### The nats-server binary can be used to send these signals to running NATS servers

#### using the -sl ag:

```
Signal Result
```
```
SIGKILL Kills the process immediately
```
```
SIGQUIT Kills the process immediately and performs a core dump
```
```
SIGINT Stops the server gracefully
```
```
SIGTERM Stops the server gracefully
```
```
SIGUSR1 Reopens the log le for log rotation
```
```
SIGHUP Reloads server conguration le
```
```
SIGUSR2 Stops the server after evicting all clients (lame duck mode)
```
```
nats-server --signal quit
```
```
nats-server --signal stop
```
## Command Line

## Quit the server

## Stop the server


#### If there are multiple nats-server processes running, or if pgrep isn't available, you

#### must either specify a PID or the absolute path to a PID le:

#### As of NATS v2.10.0, a glob expression can be used to match one or more process IDs,

#### such as:

#### See the Windows Service section for information on signaling the NATS server on

#### Windows.

```
nats-server --signal ldm
```
```
nats-server --signal reopen
```
```
nats-server --signal reload
```
```
nats-server --signal stop=<pid>
```
```
nats-server --signal stop=/path/to/pidfile
```
```
nats-server --signal ldm= 12 *
```
### Lame duck mode the server

### Reopen log file for log rotation

### Reload server configuration

### Multiple processes

## Windows


# Lame Duck Mode

#### In production we recommend that a server is shut down with lame duck mode as a

#### graceful way to slowly evict clients. With large deployments this mitigates the "thundering

#### herd" situation that will place CPU pressure on servers as TLS enabled clients reconnect.

#### Lame duck mode is initiated by signaling the server:

#### After entering lame duck mode, the server will stop accepting new connections, wait for a

#### 10 second grace period, then begin to evict clients over a period of time congurable by

#### the lame_duck_duration conguration option. This period defaults to 2 minutes.

#### When entering lame duck mode, the server will send a message to clients. Some

#### maintainer supported clients will invoke an optional callback indicating that a server is

#### entering lame duck mode. This is used for cases where an application can benet from

#### preparing for the short outage between the time it is evicted and automatically

#### reconnected to another server.

```
nats-server --signal ldm
```
## Server

## Clients


# Profiling

#### When investigating and debugging a performance issue with the NATS Server (i.e.

#### unexpectedly high CPU or RAM utilisation), it may be necessary for you to collect and

#### provide proles fr om your deployment for troublshooting. These proles are crucial to

#### understand where CPU time and memory are being spent.

#### Note that proling is an advanced operation for development purposes only. Server

#### operators should use the monitoring port instead for monitoring day-to-day runtime

#### statistics.

```
nats-server does not have authentication/authorization for the proling endpoint. When
you plan to open your nats-server to the internet make sure to not expose the proling
port as well. By default, proling binds to every interface 0.0.0.0 so consider setting
proling t o localhost or have appropriate rewall rules.
```
#### The NATS Server has pprof proling support built-in, although it must be enabled by

#### setting the prof_port in your NATS Server conguration le. For example, to enable the

#### proling port on TCP/65432:

#### Once the proling port has been enabled, you can download proles as per the following

#### sections.

#### These proles can be inspected using go tool pprof, as per the Go documentation.

```
http://localhost:65432/debug/pprof/allocs
```
#### This endpoint will return instantly.

```
prof_port = 65432
```
## Enabling profiling

## Memory profile


#### For example, to download an allocation prole fr om NATS running on the same machine:

#### The prole will be saved into mem.prof.

```
http://localhost:65432/debug/pprof/profile?seconds=30
```
#### This endpoint will block for the specied duration and then return. You can specify a

#### different duration by adjusting ?seconds= in the URL if you want to sample a shorter or

#### longer period of time.

#### For example, to download a CPU prole fr om NATS running on the same machine with a

#### 30 second window:

#### The prole will be saved into cpu.prof.

```
curl -o mem.prof http://localhost:65432/debug/pprof/allocs
```
```
curl -o cpu.prof http://localhost:65432/debug/pprof/profile?seconds= 30
```
### CPU profile


