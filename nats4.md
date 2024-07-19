# NATS Server Clients

A NATS client is an application making a connection to one of the nats servers pointed to
by its connection URL, and uses a credential le to authenticate and indicate its
authorization to the server and the whole NATS infrastructure.
The nats-server doesn't come bundled with any clients, but its companion is the nats
CLI tool that you should install (even if you don't intend to run your own servers) as it is
the best tool to use to test, monitor, manage and generally interact with a NATS
infrastructure (regardless of that infrastructure being an isolated local server, a leaf node
server, a cluster or even a global super-cluster).
Other NATS client tools to know about are the nsc CLI tool (to manage accounts
attributes and user JWT tokens) and the 'nk' tool (and library) to manage Nkeys.
Also, most client libraries come with sample programs that allow you to publish,
subscribe, send requests and reply messages.

If your application is in Go, and if it ts your use case and deployment scenarios, you can
even embed a NATS server inside your application.
Embedding NATS in Go

From the shell (all platforms), to get the binary for the latest (top of the main branch)
version of nats you can simply do:

```
curl -sf https://binaries.nats.dev/nats-io/natscli/nats | sh
```
## Embedding NATS

## Installing the nats CLI Tool


You can also get a specic version by adding @ followed by the version's tag, e.g.
curl -sf https://binaries.nats.dev/nats-io/natscli/nats@v0.1.4 | sh
For macOS:

For Arch Linux:

Binaries are also available as GitHub Releases.

Open a terminal and start a nats-server:

On another terminal session rst check the connection to the server

Next, start a subscriber using the nats CLI tool:

```
brewbrew tapinstall nats-io/nats-tools nats-io/nats-tools/nats
```
```
yay natscli
```
```
nats-server
[45695] 2021/09/29 02:22:53.570667 [INF] Starting nats-server[45695] 2021/09/29 02:22:53.570796 [INF] Version: 2.6.
[45695] 2021/09/29 02:22:53.570799 [INF] Git: [not set][45695] 2021/09/29 02:22:53.570804 [INF] Name: NAAACXGWSD6ZW5KVHOTS
[45695] 2021/09/29 02:22:53.570807 [INF] ID: NAAACXGWSD6ZW5KVHOTS[45695] 2021/09/29 02:22:53.571747 [INF] Listening for client connections
[45695] 2021/09/29 02:22:53.572051 [INF] Server is ready
```
```
nats server check connection -s 0.0.0.0:
OK Connection OK:connected to nats://127.0.0.1:4222 in 790.28μs OK:rtt ti
```
## Testing your setup


Note that when the client connected, the server didn't log anything interesting because
server output is relatively quiet unless something interesting happens.
To make the server output more lively, you can specify the -V ag to enable logging of
server protocol tracing messages. Go ahead and <ctrl>+c the process running the
server, and restart the server with the -V ag:

If you had created a subscriber, you should notice output on the subscriber telling you
that it disconnected, and reconnected. The server output above is more interesting. You
can see the subscriber send a CONNECT protocol message and a PING which was
responded to by the server with a PONG.

```
You can learn more about the NATS protocol here, but more interesting than the
protocol description is an interactive demo.
```
On a third terminal, publish your rst message:

On the subscriber window you should see:

```
nats subscribe ">" -s 0.0.0.0:
```
```
nats-server -V
[45703] 2021/09/29 02:23:05.189377 [INF] Starting nats-server[45703] 2021/09/29 02:23:05.189489 [INF] Version: 2.6.
[45703] 2021/09/29 02:23:05.189493 [INF] Git: [not set][45703] 2021/09/29 02:23:05.189497 [INF] Name: NAIBOVQLOZSDIUFQYZOQ
[45703] 2021/09/29 02:23:05.189500 [INF] ID: NAIBOVQLOZSDIUFQYZOQ[45703] 2021/09/29 02:23:05.190236 [INF] Listening for client connections
[45703] 2021/09/29 02:23:05.190504 [INF] Server is ready[45703] 2021/09/29 02:23:07.111053 [TRC] 127.0.0.1:51653 - cid:4 - <<- [C
[45703] 2021/09/29 02:23:07.111282 [TRC] 127.0.0.1:51653 - cid:4 - "v1.12[45703] 2021/09/29 02:23:07.111301 [TRC] 127.0.0.1:51653 - cid:4 - "v1.
[45703] 2021/09/29 02:23:07.111632 [TRC] 127.0.0.1:51653 - cid:4 - "v1.12[45703] 2021/09/29 02:23:07.111679 [TRC] 127.0.0.1:51653 - cid:4 - "v1.
[45703] 2021/09/29 02:23:07.111689 [TRC] 127.0.0.1:51653 - cid:4 - "v1.
```
```
nats pub hello world -s 0.0.0.0:
```

If the NATS server were running in a different machine or a different port, you'd have to
specify that to the client by specifying a NATS URL (either in a nats context or using
the -s ag).

NATS URLs take the form of: nats://<server>:<port> and tls://<server>:<port>.
URLs with a tls protocol sport a secured TLS connection.
If you are connecting to a cluster you can specify more than one URL (comma separated).
e.g. nats://localhost:4222,nats://localhost:5222,nats://localhost:6222 if you
are running a test cluster of 3 nats servers on your local machine, listening at ports 4222,
5222, and 6222 respectively.

If you want to try on a remote server, the NATS team maintains a demo server you can
reach at demo.nats.io.

```
[#1] Received on "hello"world
```
```
nats sub -s nats://server:port ">"
```
```
nats sub -s nats://demo.nats.io ">"
```
## Testing Against a Remote Server

### NATS URLs

### Example


