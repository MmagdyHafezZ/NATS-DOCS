# NATS and Docker

##### The NATS server is provided as a Docker image on Docker Hub that you can run using the

##### Docker daemon. The NATS server Docker image is extremely lightweight, coming in under

##### 10 MB in size.

##### Synadia actively maintains and supports the NATS server Docker image.

##### The nightly build container can be found here

##### To use the Docker container image, install Docker and pull the public image:

##### Run the NATS server image:

##### By default the NATS server exposes multiple ports:

##### The default ports may be customized by providing either a -p or -P option on the

##### docker run command line.

```
docker pull nats
```
```
docker run nats
```
##### 4222 is for clients.

##### 8222 is an HTTP management port for information reporting.

##### 6222 is a routing port for clustering.

## NATS Server Containerization

## Nightly

## Usage


##### The following steps illustrate how to run a server with the ports exposed on a

```
docker network.
```
##### First, create the docker network 'nats:'

##### Then start the server:

##### First, run a server with the ports exposed on the 'nats' docker network:

##### Next, start additional servers pointing them to the seed server to cause them to form a

##### cluster:

##### NOTE Since the Docker image protects routes using credentials we need to provide them

##### above. Extracted from Docker image conguration

```
docker network create nats
```
```
docker run --name nats --network nats --rm -p 4222 :4222 -p 8222 :8222 nats
```
```
docker run --name nats --network nats --rm -p 4222:4222 -p 8222:8222 nats
```
```
[1] 2021/09/28 09:21:56.554756 [INF] Starting nats-server
[1] 2021/09/28 09:21:56.554864 [INF] Version: 2.6.
[1] 2021/09/28 09:21:56.554878 [INF] Git: [c91f0fe]
[1] 2021/09/28 09:21:56.554894 [INF] Name: NDIQLLD2UGGPSAEYBKHW3S2J
[1] 2021/09/28 09:21:56.555001 [INF] ID: NDIQLLD2UGGPSAEYBKHW3S2J
[1] 2021/09/28 09:21:56.557658 [INF] Starting http monitor on 0.0.0.0:
[1] 2021/09/28 09:21:56.557967 [INF] Listening for client connections on
[1] 2021/09/28 09:21:56.559224 [INF] Server is ready
[1] 2021/09/28 09:21:56.559375 [INF] Cluster name is NATS
[1] 2021/09/28 09:21:56.559433 [INF] Listening for route connections on 0
```
```
docker run --name nats-1 --network nats --rm nats --cluster_name NATS --c
docker run --name nats-2 --network nats --rm nats --cluster_name NATS --c
```
### Creating a NATS Cluster


##### To verify the routes are connected, you can make a request to the monitoring endpoint on

##### /routez and conrm that there are now 2 routes:

```
curl http://127.0.0.1:8222/routez
```
###### {

```
"server_id": "NDIQLLD2UGGPSAEYBKHW3S2JB2DXIAFHMIWWRUBAX7FC4RTQX4ET2JNQ"
"now": "2021-09-28T09:22:15.8019785Z",
"num_routes": 2,
"routes": [
{
"rid": 5,
"remote_id": "NBRAUY3YSVFYU7BFWI2YF5VPQFGO2XCKKAHYZ7ETCMGB3SQY3FDFTY
"did_solicit": false,
"is_configured": false,
"ip": "172.18.0.3",
"port": 59092,
"pending_size": 0,
"rtt": "1.2505ms",
"in_msgs": 4,
"out_msgs": 3,
"in_bytes": 2714,
"out_bytes": 1943,
"subscriptions": 35
},
{
"rid": 6,
"remote_id": "NA5STTST5GYFCD22M2I3VDJ57LQKOU35ZVWKQY3O5QRFGOPC3RFDI
"did_solicit": false,
"is_configured": false,
"ip": "172.18.0.4",
"port": 47424,
"pending_size": 0,
"rtt": "1.2008ms",
"in_msgs": 4,
"out_msgs": 1,
"in_bytes": 2930,
"out_bytes": 833,
"subscriptions": 35
}
]
}
```

##### It is also straightforward to create a cluster using Docker Compose. Below is a simple

##### example that uses a network named 'nats' to create a full mesh cluster.

##### Now we use Docker Compose to create the cluster that will be using the 'nats' network:

```
version: "3.5"
services:
nats:
image: nats
ports:
```
- "8222:8222"
command: "--cluster_name NATS --cluster nats://0.0.0.0:6222 --http_po
networks: ["nats"]
nats-1:
image: nats
command: "--cluster_name NATS --cluster nats://0.0.0.0:6222 --routes=
networks: ["nats"]
depends_on: ["nats"]
nats-2:
image: nats
command: "--cluster_name NATS --cluster nats://0.0.0.0:6222 --routes=
networks: ["nats"]
depends_on: ["nats"]

```
networks:
nats:
name: nats
```
```
docker-compose -f nats-cluster.yaml up
```
### Creating a NATS Cluster with Docker Compose


##### Now, the following should work: make a subscription on one of the nodes and publish it

##### from another node. You should be able to receive the message without problems.

```
[+] Running 3/
⠿ Container xxx_nats_1 Created
⠿ Container xxx_nats-1_1 Created
⠿ Container xxx_nats-2_1 Created
Attaching to nats-1_1, nats-2_1, nats_
nats_1 | [1] 2021/09/28 10:42:36.742844 [INF] Starting nats-server
nats_1 | [1] 2021/09/28 10:42:36.742898 [INF] Version: 2.6.
nats_1 | [1] 2021/09/28 10:42:36.742913 [INF] Git: [c91f0fe]
nats_1 | [1] 2021/09/28 10:42:36.742929 [INF] Name: NCZIIQ6QT4KT
nats_1 | [1] 2021/09/28 10:42:36.742954 [INF] ID: NCZIIQ6QT4KT
nats_1 | [1] 2021/09/28 10:42:36.745289 [INF] Starting http monitor on
nats_1 | [1] 2021/09/28 10:42:36.745737 [INF] Listening for client con
nats_1 | [1] 2021/09/28 10:42:36.750381 [INF] Server is ready
nats_1 | [1] 2021/09/28 10:42:36.750669 [INF] Cluster name is NATS
nats_1 | [1] 2021/09/28 10:42:36.751444 [INF] Listening for route conn
nats-1_1 | [1] 2021/09/28 10:42:37.709888 [INF] Starting nats-server
nats-1_1 | [1] 2021/09/28 10:42:37.709977 [INF] Version: 2.6.
nats-1_1 | [1] 2021/09/28 10:42:37.709999 [INF] Git: [c91f0fe]
nats-1_1 | [1] 2021/09/28 10:42:37.710023 [INF] Name: NBHTXXY3HYZV
nats-1_1 | [1] 2021/09/28 10:42:37.710042 [INF] ID: NBHTXXY3HYZV
nats-1_1 | [1] 2021/09/28 10:42:37.711646 [INF] Listening for client con
nats-1_1 | [1] 2021/09/28 10:42:37.712197 [INF] Server is ready
nats-1_1 | [1] 2021/09/28 10:42:37.712376 [INF] Cluster name is NATS
nats-1_1 | [1] 2021/09/28 10:42:37.712469 [INF] Listening for route conn
nats_1 | [1] 2021/09/28 10:42:37.718918 [INF] 172.18.0.4:52950 - rid:
nats-1_1 | [1] 2021/09/28 10:42:37.719906 [INF] 172.18.0.3:6222 - rid:
nats-2_1 | [1] 2021/09/28 10:42:37.731357 [INF] Starting nats-server
nats-2_1 | [1] 2021/09/28 10:42:37.731518 [INF] Version: 2.6.
nats-2_1 | [1] 2021/09/28 10:42:37.731531 [INF] Git: [c91f0fe]
nats-2_1 | [1] 2021/09/28 10:42:37.731543 [INF] Name: NCG6UQ2N3IHE
nats-2_1 | [1] 2021/09/28 10:42:37.731554 [INF] ID: NCG6UQ2N3IHE
nats-2_1 | [1] 2021/09/28 10:42:37.732893 [INF] Listening for client con
nats-2_1 | [1] 2021/09/28 10:42:37.733431 [INF] Server is ready
nats-2_1 | [1] 2021/09/28 10:42:37.733491 [INF] Cluster name is NATS
nats-2_1 | [1] 2021/09/28 10:42:37.733835 [INF] Listening for route conn
nats_1 | [1] 2021/09/28 10:42:37.740860 [INF] 172.18.0.5:54616 - rid:
nats-2_1 | [1] 2021/09/28 10:42:37.741557 [INF] 172.18.0.3:6222 - rid:
nats-1_1 | [1] 2021/09/28 10:42:37.743981 [INF] 172.18.0.5:6222 - rid:
nats-2_1 | [1] 2021/09/28 10:42:37.744332 [INF] 172.18.0.4:40250 - rid:
```
### Testing the Clusters


##### Inside the container:

##### Stopping the seed node, which received the subscription, should trigger an automatic

##### failover to the other nodes:

##### Output extract:

##### Publishing again will continue to work after the reconnection:

##### See the NATS Docker tutorial for more instructions on using the NATS server Docker

##### image.

```
docker run --network nats --rm -it natsio/nats-box
```
```
nats sub -s nats://nats:4222 hello &
nats pub -s "nats://nats-1:4222" hello first
nats pub -s "nats://nats-2:4222" hello second
```
```
docker-compose -f nats-cluster.yaml stop nats
```
###### ...

```
16e55f1c4f3c:~# 10:47:28 Disconnected due to: EOF, will attempt reconnect
10:47:28 Disconnected due to: EOF, will attempt reconnect
10:47:28 Reconnected [nats://172.18.0.4:4222]
```
```
nats pub -s "nats://nats-1:4222" hello again
nats pub -s "nats://nats-2:4222" hello again
```
## Tutorial


# Tutorial

##### In this tutorial you run the NATS server Docker image. The Docker image provides an

##### instance of the NATS Server. Synadia actively maintains and supports the nats-server

##### Docker image. The NATS image is only 6 MB in size.

##### 1. Set up Docker.

##### See Get Started with Docker for guidance.

##### The easiest way to run Docker is to use the Docker Toolbox.

##### 2. Run the nats-server Docker image.

##### 3. Verify that the NATS server is running.

##### You should see the following:

##### Followed by this, indicating that the NATS server is running:

##### Notice how quickly the NATS server Docker image downloads. It is a mere 6 MB in size.

##### 4. Test the NATS server to verify it is running.

```
docker run -p 4222 :4222 -p 8222 :8222 -p 6222 :6222 --name nats-server -ti
```
```
Unable to find image 'nats:latest' locally
latest: Pulling from library/nats
2d3d00b0941f: Pull complete
24bc6bd33ea7: Pull complete
Digest: sha256:47b825feb34e545317c4ad122bd1a752a3172bbbc72104fc7fb5e57cf
Status: Downloaded newer image for nats:latest
```
```
[1] 2019/06/01 18:34:19.605144 [INF] Starting nats-server version 2.0.
[1] 2019/06/01 18:34:19.605191 [INF] Starting http monitor on 0.0.0.0:
[1] 2019/06/01 18:34:19.605286 [INF] Listening for client connections on
[1] 2019/06/01 18:34:19.605312 [INF] Server is ready
[1] 2019/06/01 18:34:19.608756 [INF] Listening for route connections on 0
```

##### An easy way to test the client connection port is through using telnet.

##### Expected result:

##### You can also test the monitoring endpoint, viewing http://localhost:8222 with a

##### browser.

```
telnet localhost 4222
```
```
Trying ::1...
Connected to localhost.
Escape character is '^]'.
INFO {"server_id":"NDP7NP2P2KADDDUUBUDG6VSSWKCW4IC5BQHAYVMLVAJEGZITE5XP7O
```

# Docker Swarm

##### Create an overlay network for the cluster (in this example, nats-cluster-example), and

##### instantiate an initial NATS server.

##### First create an overlay network:

##### Next instantiate an initial "seed" server for a NATS cluster listening for other servers to

##### join route to it on port 6222:

##### The 2nd step is to create another service which connects to the NATS server within the

##### overlay network. Note that we connect to to the server at nats-cluster-node-1:

```
docker network create --driver overlay nats-cluster-example
```
```
docker service create --network nats-cluster-example --name nats-cluster-
```
## Step 1:

## Step 2:


##### Now you can add more nodes to the Swarm cluster via more docker services, referencing

##### the seed server in the -routes parameter:

##### In this case, nats-cluster-node-1 is seeding the rest of the cluster through the

##### autodiscovery feature. Now NATS servers nats-cluster-node-1 and

##### nats-cluster-node-2 are clustered together.

##### Add in more replicas of the subscriber:

##### Then conrm the distribution on the Docker Swarm cluster:

```
docker service create --name ruby-nats --network nats-cluster-example wal
NATS.on_error do |e|
puts "ERROR: #{e}"
end
NATS.start(:servers => ["nats://nats-cluster-node-1:4222"]) do |nc|
inbox = NATS.create_inbox
puts "[#{Time.now}] Connected to NATS at #{nc.connected_server}, inbo
```
```
nc.subscribe(inbox) do |msg, reply|
puts "[#{Time.now}] Received reply - #{msg}"
end
```
```
nc.subscribe("hello") do |msg, reply|
next if reply == inbox
puts "[#{Time.now}] Received greeting - #{msg} - #{reply}"
nc.publish(reply, "world")
end
```
```
EM.add_periodic_timer(1) do
puts "[#{Time.now}] Saying hi (servers in pool: #{nc.server_pool}"
nc.publish("hello", "hi", inbox)
end
end'
```
```
docker service create --network nats-cluster-example --name nats-cluster-
```
```
docker service scale ruby-nats= 3
```
### Step 3:


##### The sample output after adding more NATS server nodes to the cluster, is below - and

##### notice that the client is dynamically aware of more nodes being part of the cluster via

##### auto discovery!

##### Sample output after adding more workers which can reply back (since ignoring own

##### responses):

##### From here you can experiment adding to the NATS cluster by simply adding servers with

##### new service names, that route to the seed server nats-cluster-node-1. As you've seen

##### above, clients will automatically be updated to know that new servers are available in the

##### cluster.

```
docker service ps ruby-nats
```
###### ID NAME IMAGE

```
25skxso8honyhuznu15e4989m ruby-nats.1 wallyqs/ruby-nats:ruby-2.3.1-nats
0017lut0u3wj153yvp0uxr8yo ruby-nats.2 wallyqs/ruby-nats:ruby-2.3.1-nats
2sxl8rw6vm99x622efbdmkb96 ruby-nats.3 wallyqs/ruby-nats:ruby-2.3.1-nats
```
```
[2016-08-15 12:51:52 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Ge
[2016-08-15 12:51:53 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Ge
[2016-08-15 12:51:54 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Ge
[2016-08-15 12:51:55 +0000] Saying hi (servers in pool: [{:uri=>#<URI::Ge
```
```
[2016-08-15 16:06:26 +0000] Received reply - world
[2016-08-15 16:06:26 +0000] Received reply - world
[2016-08-15 16:06:27 +0000] Received greeting - hi - _INBOX.b8d8c01753d
[2016-08-15 16:06:27 +0000] Received greeting - hi - _INBOX.4c35d
```
```
docker service create --network nats-cluster-example --name nats-cluster-
```
## And so forth...


# Python and NGS Running in Docker

##### Start a lightweight Docker container:

##### Or you can also mount local creds via a volume:

##### Install nats.py and dependencies to install nkeys:

##### Get the Python examples using curl:

##### Create a subscription that lingers:

##### Publish a message:

```
docker run --entrypoint /bin/bash -it python:3.8-slim-buster
```
```
docker run --entrypoint /bin/bash -v $HOME/.nkeys/creds/synadia/NGS/:/cre
```
```
apt-get update && apt-get install -y build-essential curl
pip install asyncio-nats-client[nkeys]
```
```
curl -o nats-pub.py -O -L https://raw.githubusercontent.com/nats-io/nats.
curl -o nats-sub.py -O -L https://raw.githubusercontent.com/nats-io/nats.
```
```
python nats-sub.py --creds /creds/NGS.creds -s tls://connect.ngs.global:
```
```
python nats-pub.py --creds /creds/NGS.creds -s tls://connect.ngs.global:
```

# JetStream

##### This mini-tutorial shows how to run a NATS server with JetStream enabled in a local

##### Docker container. This enables quick and consequence-free experimentation with the

##### many features of JetStream.

##### Using the ocial nats image, start a server. The -js option is passed to the server to

##### enable JetStream. The -p option forwards your local 4222 port to the server inside the

##### container, 4222 is the default client connection port.

##### With the server running, use nats bench to create a stream and publish some

##### messages to it.

##### JetStream persists the messages (on disk by default). Now consume them with:

##### You can use nats to inspect various aspects of the stream, for example:

```
docker run -p 4222 :4222 nats -js
```
```
nats bench -s localhost:4222 benchsubject --js --pub 1 --msgs=
```
```
nats bench -s localhost:4222 benchsubject --js --sub 3 --msgs=
```
```
nats -s localhost:4222 stream list
```
#### ╭─────────────────────────────────────────────

#### │ Streams

#### ├─────────────┬─────────────┬─────────────────

#### │ Name │ Description │ Created │ Messages │ Size │

#### ├─────────────┼─────────────┼─────────────────

#### │ benchstream │ │ 2024 -06-07 20 :26:38 │ 100 ,000 │ 16 MiB │

#### ╰─────────────┴─────────────┴─────────────────

##### Ocial Docker image for the NATS server on GitHub and issues

##### nats images on DockerHub

## Related and useful:


##### nats CLI tool and nats bench

Administer JetStream


# NGS Leaf Nodes

##### This mini-tutorial shows how to run 2 NATS server in local Docker containers

##### interconnected via Synadia Cloud Platform. NGS is a global managed NATS network of

##### NATS, and the local containers will connect to it as leaf nodes.

##### Start by creating a free account on https://cloud.synadia.com/.

##### Once you are logged in, go into the default account (you can manage multiple isolated

##### NGS account within your Synadia Cloud account).

##### In Settings > Limits, increase Leaf Nodes to 2. Save the conguration change.

##### (Your free account comes with up to 2 leaf connection, but the account is congured to

##### use at most 1 initially).

##### Now navigate to the Users section of your default account and create 2 users, red

##### and blue. (Users are another way you can isolate parts of your systems customizing

##### permissions, access to data, limits and more)

##### For each of the two users, select Get Connected and Download Credentials.

##### You should now have 2 les on your computer: default-red.creds and

##### default-blue.creds.

##### Create a minimal NATS Server conguration le leafnode.conf, it will work for both

##### leaf nodes:

##### Let's start the rst leafnode (for user red) with:

```
leafnodes {
remotes = [
{
url: "tls://connect.ngs.global"
credentials: "ngs.creds"
},
]
}
```

##### -p 4222:4222 maps the server port 4222 inside the container to your local port 4222.

##### -v leafnode.conf:/leafnode.conf mounts the conguration le created above at

##### location /leafnode.conf in the container.

##### -v /etc/ssl/cert.pem:/etc/ssl/cert.pem installs root certicates in the container,

##### since the nats image does not bundle them, and they are required to verify the TLS

##### certicate presented by NGS. -v default-red.creds:/ngs.creds installs the

##### credentials for user red at location /ngs.creds inside the container.

##### -c /leafnode.conf are arguments passed to the container entry point (nats-server

##### ).

##### Launching the container, you should see the NATS server starting successfully:

##### Now start the second leaf nodes with two minor tweaks to the command:

##### Notice we bind to local port 4333 (since 4222 ) is busy, and we mount blue

##### credentials.

##### Congratulations, you have 2 leaf nodes connected to the NGS global network. Despite

##### this being a global shared environment, your account is completely isolated from the rest

##### of the trac, and vice versa.

##### Now let's make 2 clients connected to the 2 leaf nodes talk to each other.

##### Let us start a simple service on the Leafnode of user red:

##### Using the LeafNode run by user blue, let's send a request:

```
docker run -p 4222:4222 -v leafnode.conf:/leafnode.conf -v /etc/ssl/cert
```
```
[1] 2024/06/14 18:03:51.810719 [INF] Server is ready
[1] 2024/06/14 18:03:52.075951 [INF] 34.159.142.0:7422 - lid:5 - Leafnode
[1] 2024/06/14 18:03:52.331354 [INF] 34.159.142.0:7422 - lid:5 - JetStrea
```
```
docker run -p 4333:4222 -v leafnode.conf:/leafnode.conf -v /etc/ssl/cert
```
```
nats -s localhost:4222 reply docker-leaf-test "At {{Time}}, I received yo
```

##### Congratulations, you just connected 2 Leaf nodes to the global NGS network and used

##### them to send a request and receive a response.

##### Your messages were routed transparently with millions of others, but they were not

##### visible to anyone outside of your Synadia Cloud account.

```
$ nats -s localhost:4333 request docker-leaf-test "Hello World"
```
```
At 8 :15PM, I received your request: Hello World
```
##### Ocial Docker image for the NATS server on GitHub and issues

##### nats images on DockerHub

##### nats CLI tool and nats bench

##### Leaf Nodes conguration

### Related and useful:


