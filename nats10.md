# Developing With NATS

Developing with NATS involves a blend of distributed application techniques, common NATS features, and library-specific syntax. Besides this guide, most libraries provide auto-generated API documentation, along with language and platform-specific examples, guides, and other resources.

Not all libraries have their own documentation, depending on the language community, but be sure to check out the client libraries' README for more information.

There are many other NATS client libraries and examples contributed and maintained by the community and available on GitHub, such as:

- Kotlin
- Dart
- Tcl
- Crystal
- PHP
- Pascal
- and many more...

## Anatomy of a NATS Application

You can use NATS to exchange information with and make requests to other applications. You can also use NATS to make your application into a distributed peer-to-peer application.

At a high level, your application can use NATS to:

1. **Send (Publish) information** to other applications or instances of your application.
2. **Receive (Subscribe) information** (in real-time or whenever your application runs) from other applications or instances of your application.
3. **Make a request** to a server or the service provided by another application.
4. **Store shared (consistent) state/data** between other applications or instances of your application.
5. **Be informed in real-time** about any new data being sent, or any changes to the shared state/data.

Using NATS means that, as an application developer you never have to worry about:

- Who sends the information that you want to receive.
- Who is interested in the information you are sending to others, or how many are interested or where they are.
- Where the service you are sending a request to is located, or how many currently active instances of that service there are.
- How many partitions or servers there are in the cluster.
- Security (just identify yourself).
- Whether your application is up and running at the time the information is sent or not (using JetStream).
- Flow control (using JetStream).
- Higher qualities of services such as exactly-once (using JetStream).
- Fault-tolerance, and which servers are up or down at any given time.

## What Do You Use NATS For?

A NATS Client Application will use the NATS Client Library in the following way:

1. At initialization time it will first connect (securely if needed) to a NATS Service Infrastructure (i.e. one of the NATS servers).
2. Once successfully connected, the application will:
   - Create messages and publish them to subjects or streams.
   - Subscribe to subject(s) or stream consumers to receive messages from other processes.
   - Publish request messages to a service and receive a reply message.
   - Receive request messages and send back replies or acknowledgements.
   - Associate and retrieve messages associated with keys in KV buckets.
   - Store and retrieve arbitrary large blobs with keys in the object store.
3. Finally, when the application terminates it should disconnect from the NATS Service Infrastructure.

The first thing any application needs to do is connect to NATS. Depending on the way the NATS Service Infrastructure being used is configured, the connection may need to be secured, and therefore the application also needs to be able to specify security credentials at connection time. An application can create as many NATS connections as needed (each connection being completely independent, it could for example connect twice as two different users), although typically most applications only make a single NATS connection.

## Connecting and Disconnecting

Once you have obtained a valid connection, you can use that connection in your application to use all of the Core NATS functionalities such as subscribing to subjects, publishing messages, making requests (and getting a JetStream context).

It is recommended that the application use connection event listeners in order to be alerted and log whenever connections, reconnections, or disconnections happen. Note that in case of a disconnection from the NATS server process, the client library will automatically attempt to reconnect to one of the other NATS servers in the cluster. You can also always check the current connection status.

The recommended way to disconnect is to use `Drain()` which will wait for any ongoing processing to conclude and clean everything properly, but if you need to close the connection immediately you can use `close()` from your connection object.

## Working with Messages

Messages store the data that applications exchange with each other. A message has a subject, a data payload (byte array), and may also have reply-to and header fields.

You get messages returned or passed to your callbacks from subscribing or making requests. The publish (and request) operations typically just take a subject and a byte array data payload and create the message for you, but you can also create a message yourself (if you want to set some headers).

Some messages can be 'acknowledged' (for example, message received from JetStream pull consumers), and there are multiple forms of acknowledgements (including negative acknowledgements, and acknowledgements indicating that your application has properly received the message but needs more time to process it).

Some libraries allow you to easily send and receive structured data.

Once your application has successfully connected to the NATS Server infrastructure, you can then start using the returned connection object to interact with NATS.

You can directly publish on a connection some data addressed by a subject (or publish pre-created messages with headers).

Because of caching, if your application is highly sensitive to latency, you may want to flush after publishing.

Many of the client libraries use the PING/PONG interaction built into the NATS protocol to ensure that flush pushed all of the buffered messages to the server. When an application calls flush, most libraries will put a PING on the outgoing queue of messages and wait for the server to respond with a PONG before saying that the flush was successful.

Even though the client may use PING/PONG for flush, pings sent this way do not count towards max outgoing pings.

### Core NATS Publishing

The process of subscribing involves having the client library tell NATS that an application is interested in a particular subject. When an application is done with a subscription, it unsubscribes telling the server to stop sending messages.

Receiving messages with NATS can be library dependent. Some languages, like Go or Java, provide synchronous and asynchronous APIs, while others may only support one type of subscription. In general, applications can receive messages asynchronously or synchronously.

You can always subscribe to more than one subject at a time using wildcards.

A client will receive a message for each matching subscription, so if a connection has multiple subscriptions using identical or overlapping subjects (say `foo` and `foo.*`) the same message will be sent to the client multiple times.

You can also subscribe as part of a distributed queue group. All the subscribers with the same queue group name form the distributed queue. The NATS Servers automatically distribute the messages published on the matching subject(s) between the members of the queue group.

On a given subject, there can be more than one queue group created by subscribing applications, each queue group being an independent queue and distributing its own copy of the messages between the queue group members.

One thing to keep in mind when making Core NATS subscriptions to subjects is that your application must be able to keep up with the flow of messages published on the subject(s) or it will otherwise become a slow consumer.

When you no longer want to receive the messages on a particular subject, you must call unsubscribe, or you can automatically unsubscribe after receiving a specific number of messages.

## Using JetStream

JetStream is an extension of NATS that provides higher-level functionalities such as streaming, key-value storage, and object storage. It allows for temporal decoupling and queuing of messages, enabling applications to consume messages on demand or use streams as distributed work queues.

### Defining Streams

Before you can use a stream to replay or consume messages published on a subject, it must be defined. The stream definition attributes specify:

- What is being stored (i.e., which subject(s) the stream monitors).
- How it is being stored (e.g., file or memory storage, number of replicas).
- How long the messages are stored (e.g., depending on limits, or on interest, or as a work queue): the retention policy.

Streams can be (and often are) administratively defined ahead of time (for example, using the NATS CLI Tool). The application can also manage streams (and consumers) programmatically.

### Publishing to Streams

Any message published on a subject monitored by a stream gets stored in the stream. If your application publishes a message using the Core NATS publish call (from the connection object) on a stream's subject, the message will get stored in the stream. The Core NATS publishers do not know or care whether there is a stream for that subject or not. However, if you know that there is going to be a stream defined for that subject, you will get a higher quality of service by publishing using the JetStream Context's publish call (rather than the connection's publish call). This is because JetStream publications will receive an acknowledgement (or not) from the NATS Servers when the message has been positively received and stored in the stream, while Core NATS publications are not acknowledged by the NATS Servers. This difference is also the reason why there are both synchronous and asynchronous versions of the JetStream publish operation.

### Stream Consumers

Stream consumers are how applications get messages from streams. To make another analogy to database concepts, a consumer can be seen as a kind of 'view' (on a stream):

- Consumers can have a subject filter to select messages from the stream according to their subject names.
- Consumers have an ack policy which defines whether applications must acknowledge the reception and processing of the messages being sent to them by the consumers (note that explicit acknowledgements are required for some types of streams and consumers to work properly), as well as how long to wait for acknowledgements and how many times the consumer should try to re-deliver unacknowledged

 messages.
- Consumers have a deliver policy specifying where in the stream the consumer should start delivering messages from.
- Consumers have a replay policy to specify the speed at which messages are being replayed by the consumer.

#### Ephemeral Consumers

Ephemeral consumers are, as the name suggests, not meant to last and are automatically cleaned up by the NATS Servers when the application instance that created them shuts down. Ephemeral consumers are created on-demand by individual application instances and are used only by the application instance that created them.

Applications typically use ephemeral ordered push consumers to get their own copy of the messages stored in a stream whenever they want.

#### Durable Consumers

Durable consumers are, as the name suggests, meant to be 'always on' and used (shared) by multiple instances of the client application or by applications that get stopped and restarted multiple times and need to maintain state from one run of the application to another.

Durable consumers can be managed administratively using the NATS CLI Tool or programmatically by the application itself. A consumer is created as a durable consumer simply by specifying a durable name at creation time.

Applications typically use durable pull consumers to distribute and scale horizontally the processing (or consumption) of the messages in a stream.

#### Consumer Acknowledgements

Some types of consumers (e.g., pull consumers) require the application receiving messages from the consumer to explicitly acknowledge the reception and processing of those messages. The application can invoke one of the following acknowledgement functions on the message received from the consumer:

- `ack()`: To positively acknowledge the reception and processing of the message.
- `term()`: To indicate that the message could not be and will never be able to be processed and should not be sent again later. Use `term` when the request is invalid.
- `nack()`: To negatively acknowledge the processing of the message, indicating that the message should be sent again. Use `nack` when the request is valid but you are unable to process it. If this inability to process happens because of a temporary condition, you should also close your subscription temporarily until you are able to process it again.
- `inProgress()`: To indicate that the processing of the message is still ongoing and more time is needed before the message is considered for being sent again.

### Higher Qualities of Service

JetStream enables higher qualities of service compared to Core NATS. Defining a stream on a subject and using consumers brings the quality of service up to at least once, meaning that you are guaranteed to get the message (even if your application is down at publication time). However, there are some corner case failure scenarios in which you could result in message duplication due to double publication of the message, or double processing of a message due to acknowledgement loss or crashing after processing but before acknowledging. You can enable and use message de-duplication and double-acking to protect against those failure scenarios and get exactly once quality of service.

### Key-Value Store

The Key-Value Store functionality is implemented on top of JetStream but offers a different interface in the form of keys and values rather than subject names and messages. You can use a bucket to put (including compare and set), get, and delete a value (a byte array like a message payload) associated with a key (a string, like a subject). It also allows you to 'watch' for changes to the bucket as they happen and maintain a history of the values associated with a key over time, as well as get a specific revision of the value.

### Object Store

The Object Store is similar to the Key-Value Store but meant to be used where the values can be of any arbitrarily large size, as opposed to being limited to the maximum size of a NATS message, as is the case with the Key-Value Store.

## Connecting

In order for a NATS client application to connect to the NATS service, and then subscribe or publish messages to subjects, it needs to be configured with the details of how to connect to the NATS service infrastructure and how to authenticate with it.

### NATS URL

A 'NATS URL' is a string (in a URL format) that specifies the IP address and port where the NATS server(s) can be reached, and what kind of connection to establish:

- TLS encrypted only TCP connection (i.e., NATS URLs starting with tls://...).
- TLS encrypted if the server is configured for it or plain unencrypted TCP connection otherwise (i.e., NATS URLs starting with nats://...).
- WebSocket connection (i.e., NATS URLs starting with ws://...).

Note that when connecting to a NATS service infrastructure with clusters there is more than one URL, and the application should allow for more than one URL to be specified in its NATS connect call (typically you pass a comma-separated list of URLs as the URL, e.g., "nats://server1:port1,nats://server2:port2").

### Connecting to Clusters

When connecting to a cluster, it is best to provide the complete set of 'seed' URLs for the cluster. After a client connects to the server, the server may provide a list of URLs for additional known servers. This allows a client to connect to one server and still have other servers available during reconnect. To ensure the initial connection, your code should include a list of reasonable front-line or seed servers. Those servers may know about other members of the cluster and may tell the client about those members, but you don't have to configure the client to pass every valid member of the cluster in the connect method.

### Authentication Details

If required, authentication details for the application to identify itself with the NATS server(s) must be provided. NATS supports multiple authentication schemes:

- Username/Password credentials (which can be passed as part of the NATS URL).
- Decentralized JWT Authentication/Authorization (where the application is configured with the location of 'credentials file' containing the JWT and private Nkey).
- Token Authentication (where the application is configured with a Token string).
- TLS Certificate (where the client is configured to use a client TLS certificate and the servers are configured to map the TLS client certificates to users defined in the server configuration).
- NKEY with Challenge (where the client is configured with a Seed and User NKeys).

Your application should expose a way to be configured at runtime with the NATS URL(s) to use. If you want to use a secure infrastructure, the application must provide for the definition of either the credentials file (.creds) to use, or the means to encode the token, or Nkey, in the URL(s).

### Connection Options

Besides the connectivity and security details, there are numerous options for a NATS connection ranging from timeouts to reconnect settings to setting asynchronous error and connection event callback handlers in your application.

### WebSocket and NATS

NATS also supports WebSocket connections, allowing for integration with web applications and frameworks such as React.

## Connecting to the Default Server

Some libraries also provide a special way to connect to a default URL, which is generally `nats://localhost:4222`:

- Go
- Java
- JavaScript
- Python
- Ruby
- C

## Connecting to a Specific Server

The NATS client libraries can take a full URL, such as `nats://demo.nats.io:4222`, to specify a specific server host and port to connect to.

Libraries are removing the requirement for an explicit protocol and may allow `demo.nats.io:4222` or just `demo.nats.io`. In the latter example, the default port 4222 will be used. Check with your specific client library's documentation to see what URL formats are supported.

For example, to connect to the demo server with a URL you can use:

- Go
- Java
- JavaScript
- Python
- C#
- Ruby
- C

## Connecting to a Cluster

When connecting to a cluster, there are a few things to think about. By providing the ability to pass multiple connect options, NATS can handle the possibility of a machine going down or being unavailable to a client. By adding the ability of the server to feed clients a list of known servers as part of the client-server protocol, the mesh created by a cluster can grow and change organically while the clients are running.

Note, failure behavior is library dependent, so please check the documentation for your client library on information about what happens if the connection fails.

### Connection Algorithm

- Passing a URL for each cluster member (semi-optional).
- The connection algorithm.
- The reconnect algorithm (discussed later).
- Server provided URLs.

## Summary

NATS is a powerful and flexible messaging system that allows for a wide range of use cases, from simple publish/subscribe models to complex distributed systems with fault-tolerance and high availability. Understanding the core concepts and best practices for connecting, publishing, subscribing, and managing streams and consumers will enable you to build robust and scalable applications using NATS.