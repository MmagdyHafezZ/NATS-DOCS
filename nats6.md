# NATS and Docker

The NATS server is provided as a Docker image on Docker Hub, which you can run using the Docker daemon. The NATS server Docker image is extremely lightweight, coming in under 10 MB in size. Synadia actively maintains and supports the NATS server Docker image.

## Nightly Build Container

You can find the nightly build container on Docker Hub.

## Usage

To use the Docker container image, install Docker and pull the public image:

```bash
docker pull nats
```

Run the NATS server image:

```bash
docker run nats
```

By default, the NATS server exposes multiple ports:
- **4222**: for clients.
- **8222**: an HTTP management port for information reporting.
- **6222**: a routing port for clustering.

The default ports may be customized by providing either a `-p` or `-P` option on the `docker run` command line.

## Creating a NATS Cluster

To create a NATS cluster, follow these steps:

### Step 1: Create a Docker Network

First, create the Docker network 'nats':

```bash
docker network create nats
```

### Step 2: Start the Seed Server

Run a server with the ports exposed on the 'nats' Docker network:

```bash
docker run --name nats --network nats --rm -p 4222:4222 -p 8222:8222 nats
```

### Step 3: Start Additional Servers

Start additional servers pointing them to the seed server to form a cluster:

```bash
docker run --name nats-1 --network nats --rm nats --cluster_name NATS --cluster nats://nats:6222
docker run --name nats-2 --network nats --rm nats --cluster_name NATS --cluster nats://nats:6222
```

Since the Docker image protects routes using credentials, provide them as necessary. Extract the credentials from the Docker image configuration if needed.

### Verifying Routes

To verify the routes are connected, you can make a request to the monitoring endpoint on `/routez` and confirm that there are now 2 routes:

```bash
curl http://127.0.0.1:8222/routez
```

You should see output indicating the connected routes.

### Creating a NATS Cluster with Docker Compose

It is also straightforward to create a cluster using Docker Compose. Below is a simple example that uses a network named 'nats' to create a full mesh cluster.

Create a Docker Compose file `nats-cluster.yaml` with the following content:

```yaml
version: "3.5"
services:
  nats:
    image: nats
    ports:
      - "8222:8222"
    command: "--cluster_name NATS --cluster nats://0.0.0.0:6222 --http_port 8222"
    networks: ["nats"]

  nats-1:
    image: nats
    command: "--cluster_name NATS --cluster nats://0.0.0.0:6222 --routes=nats://nats:6222"
    networks: ["nats"]
    depends_on:
      - nats

  nats-2:
    image: nats
    command: "--cluster_name NATS --cluster nats://0.0.0.0:6222 --routes=nats://nats:6222"
    networks: ["nats"]
    depends_on:
      - nats

networks:
  nats:
    name: nats
```

Now use Docker Compose to create the cluster:

```bash
docker-compose -f nats-cluster.yaml up
```

### Testing the Cluster

Make a subscription on one of the nodes and publish it from another node. You should be able to receive the message without problems.

```bash
docker run --network nats --rm -it natsio/nats-box
nats sub -s nats://nats:4222 hello &
nats pub -s "nats://nats-1:4222" hello first
nats pub -s "nats://nats-2:4222" hello second
```

Stopping the seed node, which received the subscription, should trigger an automatic failover to the other nodes. Publishing again will continue to work after the reconnection.

```bash
docker-compose -f nats-cluster.yaml stop nats
```

Expected output after reconnection:

```bash
16e55f1c4f3c:~# 10:47:28 Disconnected due to: EOF, will attempt reconnect
10:47:28 Disconnected due to: EOF, will attempt reconnect
10:47:28 Reconnected [nats://172.18.0.4:4222]
```

Publish messages again:

```bash
nats pub -s "nats://nats-1:4222" hello again
nats pub -s "nats://nats-2:4222" hello again
```

See the NATS Docker tutorial for more instructions on using the NATS server Docker image.

# Tutorial

In this tutorial, you will run the NATS server Docker image. The Docker image provides an instance of the NATS Server, actively maintained and supported by Synadia.

1. **Set up Docker**: See "Get Started with Docker" for guidance. The easiest way to run Docker is to use the Docker Toolbox.

2. **Run the nats-server Docker image**:

```bash
docker run -p 4222:4222 -p 8222:8222 -p 6222:6222 --name nats-server -ti nats
```

3. **Verify that the NATS server is running**:

You should see output indicating that the NATS server is running, such as:

```bash
[1] 2019/06/01 18:34:19.605144 [INF] Starting nats-server version 2.0.0
[1] 2019/06/01 18:34:19.605191 [INF] Starting http monitor on 0.0.0.0:8222
[1] 2019/06/01 18:34:19.605286 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2019/06/01 18:34:19.605312 [INF] Server is ready
[1] 2019/06/01 18:34:19.608756 [INF] Listening for route connections on 0.0.0.0:6222
```

4. **Test the NATS server** to verify it is running. An easy way to test the client connection port is through using telnet:

```bash
telnet localhost 4222
```

Expected result:

```bash
Trying ::1...
Connected to localhost.
Escape character is '^]'.
INFO {"server_id":"NDP7NP2P2KADDDUUBUDG6VSSWKCW4IC5BQHAYVMLVAJEGZITE5XP7O"}
```

You can also test the monitoring endpoint by viewing http://localhost:8222 with a browser.

# Docker Swarm

Create an overlay network for the cluster (in this example, `nats-cluster-example`), and instantiate an initial NATS server.

### Step 1: Create an Overlay Network

```bash
docker network create --driver overlay nats-cluster-example
```

### Step 2: Instantiate an Initial Seed Server

Create an initial "seed" server for a NATS cluster listening for other servers to join route to it on port 6222:

```bash
docker service create --network nats-cluster-example --name nats-cluster-node-1 nats --cluster nats://0.0.0.0:6222
```

### Step 3: Add More Nodes

Add more nodes to the Swarm cluster via additional Docker services, referencing the seed server in the `-routes` parameter:

```bash
docker service create --network nats-cluster-example --name nats-cluster-node-2 nats --cluster nats://0.0.0.0:6222 --routes=nats://nats-cluster-node-1:6222
```

In this case, `nats-cluster-node-1` is seeding the rest of the cluster through the autodiscovery feature. Now, NATS servers `nats-cluster-node-1` and `nats-cluster-node-2` are clustered together.

### Scale and Confirm

Scale the service:

```bash
docker service scale nats-cluster-node-2=3
```

Confirm the distribution on the Docker Swarm cluster:

```bash
docker service ps nats-cluster-node-2
```

Sample output:

```bash
ID            NAME               IMAGE                NODE                DESIRED STATE  CURRENT STATE
25skxso8honh  nats-cluster-node-2.1  nats:latest        worker-node-1      Running        Running 5 minutes ago
0017lut0u3wj  nats-cluster-node-2.2  nats:latest        worker-node-2      Running        Running 5 minutes ago
2sxl8rw6vm99  nats-cluster-node-2.3  nats:latest        worker-node-3      Running        Running 5 minutes ago
```

The client is dynamically aware of more nodes being part of the cluster via autodiscovery.

# Python and NGS Running in Docker



1. **Start a lightweight Docker container**:

```bash
docker run --entrypoint /bin/bash -it python:3.8-slim-buster
```

2. **Mount local creds via a volume**:

```bash
docker run --entrypoint /bin/bash -v $HOME/.nkeys/creds/synadia/NGS/:/creds -it python:3.8-slim-buster
```

3. **Install nats.py and dependencies** to install nkeys:

```bash
apt-get update && apt-get install -y build-essential curl
pip install asyncio-nats-client[nkeys]
```

4. **Get the Python examples** using curl:

```bash
curl -o nats-pub.py -O -L https://raw.githubusercontent.com/nats-io/nats.py/master/examples/nats-pub.py
curl -o nats-sub.py -O -L https://raw.githubusercontent.com/nats-io/nats.py/master/examples/nats-sub.py
```

5. **Create a subscription that lingers**:

```bash
python nats-sub.py --creds /creds/NGS.creds -s tls://connect.ngs.global:4222
```

6. **Publish a message**:

```bash
python nats-pub.py --creds /creds/NGS.creds -s tls://connect.ngs.global:4222
```

# JetStream

This mini-tutorial shows how to run a NATS server with JetStream enabled in a local Docker container. This enables quick and consequence-free experimentation with the many features of JetStream.

1. **Start a NATS server with JetStream enabled**:

```bash
docker run -p 4222:4222 nats -js
```

2. **Create a stream and publish some messages to it**:

```bash
nats bench -s localhost:4222 benchsubject --js --pub 1 --msgs=100000
```

3. **Consume the messages**:

```bash
nats bench -s localhost:4222 benchsubject --js --sub 3 --msgs=100000
```

4. **Inspect various aspects of the stream**:

```bash
nats -s localhost:4222 stream list
```

Sample output:

```bash
╭─────────────────────────────────────────────
│ Streams
├─────────────┬─────────────┬─────────────────
│ Name        │ Description │ Created         │ Messages │ Size │
├─────────────┼─────────────┼─────────────────
│ benchstream │             │ 2024-06-07 20:26 │ 100,000  │ 16 MiB │
╰─────────────┴─────────────┴─────────────────
```

# NGS Leaf Nodes

This mini-tutorial shows how to run 2 NATS servers in local Docker containers interconnected via Synadia Cloud Platform. NGS is a global managed NATS network, and the local containers will connect to it as leaf nodes.

1. **Create a free account on Synadia Cloud**: [https://cloud.synadia.com/](https://cloud.synadia.com/).

2. **Increase Leaf Nodes to 2** in Settings > Limits.

3. **Create 2 users (red and blue)** in the Users section of your default account and download credentials for each user.

4. **Create a minimal NATS Server configuration file** `leafnode.conf`:

```bash
leafnodes {
  remotes = [
    {
      url: "tls://connect.ngs.global"
      credentials: "ngs.creds"
    }
  ]
}
```

5. **Start the first leafnode** (for user red):

```bash
docker run -p 4222:4222 -v leafnode.conf:/leafnode.conf -v /etc/ssl/cert.pem:/etc/ssl/cert.pem -v default-red.creds:/ngs.creds nats -c /leafnode.conf
```

6. **Start the second leafnode** (for user blue):

```bash
docker run -p 4333:4222 -v leafnode.conf:/leafnode.conf -v /etc/ssl/cert.pem:/etc/ssl/cert.pem -v default-blue.creds:/ngs.creds nats -c /leafnode.conf
```

Congratulations, you have 2 leaf nodes connected to the NGS global network. Now, let's make 2 clients connected to the 2 leaf nodes talk to each other.

7. **Start a simple service on the Leafnode of user red**:

```bash
nats -s localhost:4222 reply docker-leaf-test "At {{Time}}, I received your request: {{Request}}"
```

8. **Send a request using the LeafNode run by user blue**:

```bash
nats -s localhost:4333 request docker-leaf-test "Hello World"
```

Expected response:

```bash
At 8:15PM, I received your request: Hello World
```

Your messages were routed transparently through the NGS network but were isolated within your Synadia Cloud account.