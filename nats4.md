# NATS Server Clients

A NATS client is an application making a connection to one of the NATS servers specified by its connection URL, using a credential file to authenticate and indicate its authorization to the server and the entire NATS infrastructure.

## Tools and Utilities

### NATS CLI Tool

The NATS server doesn't come bundled with any clients, but its companion is the `nats` CLI tool, which is essential for testing, monitoring, managing, and interacting with a NATS infrastructure. This tool is useful whether the infrastructure is an isolated local server, a leaf node server, a cluster, or even a global super-cluster.

To install the `nats` CLI tool:

```bash
curl -sf https://binaries.nats.dev/nats-io/natscli/nats | sh
```

You can also get a specific version by adding `@` followed by the version's tag, e.g.,

```bash
curl -sf https://binaries.nats.dev/nats-io/natscli/nats@v0.1.4 | sh
```

For macOS:

```bash
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
```

For Arch Linux:

```bash
yay -S natscli
```

Binaries are also available as [GitHub Releases](https://github.com/nats-io/natscli/releases).

### Other Tools

- **nsc CLI tool**: Manages account attributes and user JWT tokens.
- **nk tool (and library)**: Manages Nkeys.

Most client libraries come with sample programs to publish, subscribe, send requests, and reply to messages.

## Embedding NATS

If your application is in Go, you can even embed a NATS server inside your application if it fits your use case and deployment scenarios. Refer to the [Embedding NATS in Go](https://github.com/nats-io/nats.go) documentation for more details.

## Installing and Running the NATS Server

Open a terminal and start a `nats-server`:

```bash
nats-server
```

Example output:

```text
[45695] 2021/09/29 02:22:53.570667 [INF] Starting nats-server
[45695] 2021/09/29 02:22:53.570796 [INF] Version: 2.6.
[45695] 2021/09/29 02:22:53.570799 [INF] Git: [not set]
[45695] 2021/09/29 02:22:53.570804 [INF] Name: NAAACXGWSD6ZW5KVHOTS
[45695] 2021/09/29 02:22:53.570807 [INF] ID: NAAACXGWSD6ZW5KVHOTS
[45695] 2021/09/29 02:22:53.571747 [INF] Listening for client connections
[45695] 2021/09/29 02:22:53.572051 [INF] Server is ready
```

## Testing Your Setup

On another terminal session, first check the connection to the server:

```bash
nats server check connection -s nats://127.0.0.1:4222
```

Example output:

```text
OK Connection OK: connected to nats://127.0.0.1:4222 in 790.28µs
OK: rtt time 120.98µs
```

Start a subscriber using the `nats` CLI tool:

```bash
nats subscribe ">" -s nats://127.0.0.1:4222
```

### Verbose Logging

To make the server output more lively, you can enable logging of server protocol tracing messages with the `-V` flag. Restart the server with the `-V` flag:

```bash
nats-server -V
```

Example output with verbose logging:

```text
[45703] 2021/09/29 02:23:05.189377 [INF] Starting nats-server
[45703] 2021/09/29 02:23:05.189489 [INF] Version: 2.6.
[45703] 2021/09/29 02:23:05.189493 [INF] Git: [not set]
[45703] 2021/09/29 02:23:05.189497 [INF] Name: NAIBOVQLOZSDIUFQYZOQ
[45703] 2021/09/29 02:23:05.189500 [INF] ID: NAIBOVQLOZSDIUFQYZOQ
[45703] 2021/09/29 02:23:05.190236 [INF] Listening for client connections
[45703] 2021/09/29 02:23:05.190504 [INF] Server is ready
[45703] 2021/09/29 02:23:07.111053 [TRC] 127.0.0.1:51653 - cid:4 - <<- [CONNECT {"verbose":false,"pedantic":false,"tls_required":false,"name":"","lang":"go","version":"1.12.1","protocol":1,"echo":true}]
[45703] 2021/09/29 02:23:07.111282 [TRC] 127.0.0.1:51653 - cid:4 - "v1.12.1:go:windows"
[45703] 2021/09/29 02:23:07.111301 [TRC] 127.0.0.1:51653 - cid:4 - <<- [PING]
[45703] 2021/09/29 02:23:07.111632 [TRC] 127.0.0.1:51653 - cid:4 - ->> [PONG]
```

### Publishing Messages

On a third terminal, publish your first message:

```bash
nats pub hello world -s nats://127.0.0.1:4222
```

On the subscriber window, you should see:

```text
[#1] Received on "hello"
world
```

## Testing Against a Remote Server

If the NATS server is running on a different machine or port, specify that to the client using the `-s` flag. NATS URLs take the form of `nats://<server>:<port>` and `tls://<server>:<port>` for secured TLS connections.

For example:

```bash
nats sub -s nats://demo.nats.io ">"
```

To try on a remote server, use the NATS team's demo server at `demo.nats.io`.

### Example

Subscribe to all subjects on the demo server:

```bash
nats sub -s nats://demo.nats.io ">"
```

## Summary

Using the `nats` CLI tool simplifies testing, monitoring, and managing your NATS infrastructure. Whether you are running a local server or connecting to a remote server, the CLI tool provides a straightforward way to interact with NATS. By embedding NATS in Go applications, you can create seamless integrations tailored to your specific use cases.