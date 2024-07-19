# Managing and Monitoring your

# NATS Server Infrastructure

#### Managing a NATS Server is simple, typical lifecycle operations include:

#### Using the nats CLI tool to check server cluster connectivity and latencies, as well as

#### get account information, and manage and interact with streams (and other NATS

#### applications). Try the following examples to learn about the most common ways to

#### use nats.

```
nats cheat
```
```
nats cheat server
```
#### nats stream --help to monitor, manage and interact with streams

#### nats consumer --help to monitor, manage stream consumers

#### nats context --help if you need to switch between servers, clusters or user

#### credentials

#### Using the nsc CLI tool when using JWT based authentication and authorization, to

#### create, revoke operators, accounts, and user (i.e. client applications) JWTs and keys.

#### Sending signals to a server to reload a conguration or rotate log les

#### Upgrading a server (or cluster)

#### Understanding slow consumers

#### Monitoring the server via:

#### The monitoring endpoint and tools like nats-top

#### By subscribing to system events

#### Gracefully shut down a server with Lame Duck Mode


# Monitoring

#### To monitor the NATS messaging system, nats-server provides a lightweight HTTP

#### server on a dedicated monitoring port. The monitoring server provides several endpoints,

#### providing statistics and other information about the following:

#### All endpoints return a JSON object.

#### The NATS monitoring endpoints support JSONP and CORS, making it easy to create

#### single page monitoring web applications. Part of the NATS ecosystem is a tool called

#### nats-top that visualizes data from these endpoints on the command line.

```
nats-server does not have authentication/authorization for the monitoring endpoint.
When you plan to open your nats-server to the internet make sure to not expose the
monitoring port as well. By default, monitoring binds to every interface 0.0.0.0 so
consider setting monitoring to localhost or have appropriate rewall rules.
```
#### General Server Information

#### Connections

#### Routing

#### Leaf Nodes

#### Subscription Routing

#### Account Information

#### Account Stats

#### JetStream Information

#### Health

## Monitoring NATS

## Enabling monitoring


#### Monitoring can be enabled in server conguration or as a server command-line option.

#### The conventional port is 8222.

#### As server conguration:

#### As a command-line option:

#### Once the server is running using one of the two methods, go to http://localhost:8222 to

#### browse the available endpoints detailed below.

#### The /varz endpoint returns general information about the server state and

#### conguration.

#### N/A

#### https://demo.nats.io:8222/varz

```
http_port: 8222
```
```
nats-server -m 8222
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
## Monitoring Endpoints

### General Information

#### Arguments

#### Example


#### Response


###### {

"server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW"
"version": "2.0.0",
"proto": 1 ,
"go": "go1.12",
"host": "0.0.0.0",
"port": 4222 ,
"max_connections": 65536 ,
"ping_interval": 120000000000 ,
"ping_max": 2 ,
"http_host": "0.0.0.0",
"http_port": 8222 ,
"https_port": 0 ,
"auth_timeout": 1 ,
"max_control_line": 4096 ,
"max_payload": 1048576 ,
"max_pending": 67108864 ,
"cluster": {},
"gateway": {},
"leaf": {},
"tls_timeout": 0.5,
"write_deadline": 2000000000 ,
"start": "2019-06-24T14:24:43.928582-07:00",
"now": "2019-06-24T14:24:46.894852-07:00",
"uptime": "2s",
"mem": 9617408 ,
"cores": 4 ,
"gomaxprocs": 4 ,
"cpu": 0 ,
"connections": 0 ,
"total_connections": 0 ,
"routes": 0 ,
"remotes": 0 ,
"leafnodes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"slow_consumers": 2 ,
"subscriptions": 0 ,
"http_req_stats": {
"/": 0 ,
"/connz": 0 ,
"/gatewayz": 0 ,
"/routez": 0 ,
"/subsz": 0 ,
"/varz": 1
},


#### The /connz endpoint reports more detailed information on current and recently closed

#### connections. It uses a paging mechanism which defaults to 1024 connections.

###### },

```
"config_load_time": "2019-06-24T14:24:43.928582-07:00",
"slow_consumer_stats": {
"clients": 1 ,
"routes": 1 ,
"gateways": 0 ,
"leafs": 0
}
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
##### sort (see sort options) Sorts the results. Default is connection ID.

```
auth true, 1, false, 0 Include username. Default is false.
```
```
subs
```
```
true, 1, false,
0 or detail
```
```
Include subscriptions. Default is false. When
set to detail a list with more detailed
subscription information will be returned.
```
```
offset number > 0 Pagination offset. Default is 0.
```
```
limit number > 0 Number of results to return. Default is 1024.
```
```
cid number, valid id Return a connection by its id
```
```
state open, *closed, any
```
```
Return connections of
particular state. Default is open.
```
### Connection Information

#### Arguments


#### The server will default to holding the last 10,000 closed connections.

#### Sort Options

#### Get up to 1024 connections: https://demo.nats.io:8222/connz

```
Argument Values Description
```
```
mqtt_client string Filter the connection with this MQTT client ID.
```
```
Option Sort by
```
```
cid Connection ID
```
```
start Connection start time, same as CID
```
```
subs Number of subscriptions
```
```
pending Amount of data in bytes waiting to be sent to client
```
```
msgs_to Number of messages sent
```
```
msgs_from Number of messages received
```
```
bytes_to Number of bytes sent
```
```
bytes_from Number of bytes received
```
```
last Last activity
```
```
idle Amount of inactivity
```
```
uptime Lifetime of the connection
```
```
stop Stop time for a closed connection
```
```
reason Reason for a closed connection
```
```
rtt Round trip time
```
#### Examples


#### Control limit and offset: https://demo.nats.io:8222/connz?limit=16&offset=128.

#### Get closed connection information: https://demo.nats.io:8222/connz?state=closed.

#### You can also report detailed subscription information on a per connection basis using

#### subs=1. For example: https://demo.nats.io:8222/connz?limit=1&offset=1&subs=1.

#### Response


###### {

"server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW"
"now": "2019-06-24T14:28:16.520365-07:00",
"num_connections": 2 ,
"total": 2 ,
"offset": 0 ,
"limit": 1024 ,
"connections": [
{
"cid": 5 ,
"kind": "Client",
"type": "nats",
"ip": "127.0.0.1",
"port": 62714 ,
"start": "2021-09-09T23:16:43.040862Z",
"last_activity": "2021-09-09T23:16:43.042364Z",
"rtt": "95μs",
"uptime": "5s",
"idle": "5s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 1 ,
"name": "NATS Benchmark",
"lang": "go",
"version": "1.12.1"
},
{
"cid": 6 ,
"kind": "Client",
"type": "nats",
"ip": "127.0.0.1",
"port": 62715 ,
"start": "2021-09-09T23:16:43.042557Z",
"last_activity": "2021-09-09T23:16:43.042811Z",
"rtt": "100μs",
"uptime": "5s",
"idle": "5s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 1 ,
"name": "NATS Benchmark",
"lang": "go",


#### The /routez endpoint reports information on active routes for a cluster. Routes are

#### expected to be low, so there is no paging mechanism with this endpoint.

```
lang : go,
"version": "1.12.1"
},
{
"cid": 7 ,
"kind": "Client",
"type": "mqtt",
"ip": "::1",
"port": 62718 ,
"start": "2021-09-09T23:16:45.391459Z",
"last_activity": "2021-09-09T23:16:45.395869Z",
"rtt": "0s",
"uptime": "2s",
"idle": "2s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 2 ,
"mqtt_client": "mqtt_sub"
}
]
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
subs
```
```
true, 1, false,
0 or detail
```
```
Include subscriptions. Default is false. When
set to detail a list with more detailed
subscription information will be returned.
```
### Route Information

#### Arguments


#### As noted above, the routez endpoint does support the subs argument from the

#### /connz endpoint. For example: https://demo.nats.io:8222/routez?subs=

#### The /gatewayz endpoint reports information about gateways used to create a NATS

#### supercluster. Like routes, the number of gateways are expected to be low, so there is no

#### paging mechanism with this endpoint.

#### Get route information: https://demo.nats.io:8222/routez?subs=

###### {

```
"server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW"
"now": "2019-06-24T14:29:16.046656-07:00",
"num_routes": 1 ,
"routes": [
{
"rid": 1 ,
"remote_id": "de475c0041418afc799bccf0fdd61b47",
"did_solicit": true,
"ip": "127.0.0.1",
"port": 61791 ,
"pending_size": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 0
}
]
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
#### Example

#### Response

### Gateway Information


```
Result Return Code
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
accs true, 1, false, 0 Include account information. Default is false.
```
```
gw_name string Return only remote gateways with this name.
```
```
acc_name string Limit the list of accounts to this account name.
```
#### Retrieve Gateway Information: https://demo.nats.io:8222/gatewayz

#### Arguments

#### Examples

#### Response


###### {

"server_id": "NANVBOU62MDUWTXWRQ5KH3PSMYNCHCEUHQV3TW3YH7WZLS7FMJE6END6"
"now": "2019-07-24T18:02:55.597398-06:00",
"name": "region1",
"host": "2601:283:4601:1350:1895:efda:2010:95a1",
"port": 4501 ,
"outbound_gateways": {
"region2": {
"configured": true,
"connection": {
"cid": 7 ,
"ip": "127.0.0.1",
"port": 5500 ,
"start": "2019-07-24T18:02:48.765621-06:00",
"last_activity": "2019-07-24T18:02:48.765621-06:00",
"uptime": "6s",
"idle": "6s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 0 ,
"name": "NCXBIYWT7MV7OAQTCR4QTKBN3X3HDFGSFWTURTCQ22ZZB6NKKJPO7MN
}
},
"region3": {
"configured": true,
"connection": {
"cid": 5 ,
"ip": "::1",
"port": 6500 ,
"start": "2019-07-24T18:02:48.764685-06:00",
"last_activity": "2019-07-24T18:02:48.764685-06:00",
"uptime": "6s",
"idle": "6s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 0 ,
"name": "NCVS7Q65WX3FGIL2YQRLI77CE6MQRWO2Y453HYVLNMBMTVLOKMPW7R6K
}
}
},
"inbound_gateways": {
"region2": [


region2 : [
{
"configured": false,
"connection": {
"cid": 9 ,
"ip": "::1",
"port": 52029 ,
"start": "2019-07-24T18:02:48.76677-06:00",
"last_activity": "2019-07-24T18:02:48.767096-06:00",
"uptime": "6s",
"idle": "6s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 0 ,
"name": "NCXBIYWT7MV7OAQTCR4QTKBN3X3HDFGSFWTURTCQ22ZZB6NKKJPO7M
}
}
],
"region3": [
{
"configured": false,
"connection": {
"cid": 4 ,
"ip": "::1",
"port": 52025 ,
"start": "2019-07-24T18:02:48.764577-06:00",
"last_activity": "2019-07-24T18:02:48.764994-06:00",
"uptime": "6s",
"idle": "6s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 0 ,
"name": "NCVS7Q65WX3FGIL2YQRLI77CE6MQRWO2Y453HYVLNMBMTVLOKMPW7R
}
},
{
"configured": false,
"connection": {
"cid": 8 ,
"ip": "127.0.0.1",
"port": 52026 ,
"start": "2019-07-24T18:02:48.766173-06:00",
"last_activity": "2019-07-24T18:02:48.766999-06:00",


#### The /leafz endpoint reports detailed information about the leaf node connections.

#### As noted above, the leafz endpoint does support the subs argument from the

#### /connz endpoint. For example: https://demo.nats.io:8222/leafz?subs=

```
"uptime": "6s",
"idle": "6s",
"pending_bytes": 0 ,
"in_msgs": 0 ,
"out_msgs": 0 ,
"in_bytes": 0 ,
"out_bytes": 0 ,
"subscriptions": 0 ,
"name": "NCKCYK5LE3VVGOJQ66F65KA27UFPCLBPX4N4YOPOXO3KHGMW24USPC
}
}
]
}
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
subs true, 1, false, 0 Include internal subscriptions. Default is false.
```
#### Get leaf nodes information: https://demo.nats.io:8222/leafz?subs=

### Leaf Node Information

#### Arguments

#### Example

#### Response


#### The /subsz endpoint reports detailed information about the current subscriptions and

#### the routing data structure. It is not normally used.

###### {

```
"server_id": "NC2FJCRMPBE5RI5OSRN7TKUCWQONCKNXHKJXCJIDVSAZ6727M7MQFVT3"
"now": "2019-08-27T09:07:05.841132-06:00",
"leafnodes": 1 ,
"leafs": [
{
"account": "$G",
"ip": "127.0.0.1",
"port": 6223 ,
"rtt": "200μs",
"in_msgs": 0 ,
"out_msgs": 10000 ,
"in_bytes": 0 ,
"out_bytes": 1280000 ,
"subscriptions": 1 ,
"subscriptions_list": ["foo"]
}
]
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
subs true, 1, false, 0 Include subscriptions. Default is false.
```
```
offset integer > 0 Pagination offset. Default is 0.
```
```
limit integer > 0 Number of results to return. Default is 1024.
```
### Subscription Routing Information

#### Arguments


#### The /accountz endpoint reports information on a server's active accounts. The default

#### behavior is to return a list of all accounts known to the server.

```
Argument Values Description
```
```
test subject Test whether a subsciption exists.
```
#### Get subscription routing information: https://demo.nats.io:8222/subsz

###### {

```
"num_subscriptions": 2 ,
"num_cache": 0 ,
"num_inserts": 2 ,
"num_removes": 0 ,
"num_matches": 0 ,
"cache_hit_rate": 0 ,
"max_fanout": 0 ,
"avg_fanout": 0
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Value Description
```
```
acc account name
```
```
Include metrics for the specied account. Default is
empty. When not set, a list of all accounts is included.
```
#### Example

#### Response

### Account Information

#### Example


#### Default behavior:

#### Retrieve specic account:

#### Get list of all accounts: https://demo.nats.io:8222/accountz

#### Get details for specic account $G: https://demo.nats.io:8222/accountz?acc=$G

###### {

```
"server_id": "NAB2EEQ3DLS2BHU4K2YMXMPIOOOAOFOAQAC5NQRIEUI4BHZKFBI4ZU4A"
"now": "2021-02-08T17:31:29.551146-05:00",
"system_account": "AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLK
"accounts": ["AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLKU7A5"
}
```
#### Response


###### {

"server_id": "NAB2EEQ3DLS2BHU4K2YMXMPIOOOAOFOAQAC5NQRIEUI4BHZKFBI4ZU4A"
"now": "2021-02-08T17:37:55.80856-05:00",
"system_account": "AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLK
"account_detail": {
"account_name": "AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLK
"update_time": "2021-02-08T17:31:22.390334-05:00",
"is_system": true,
"expired": false,
"complete": true,
"jetstream_enabled": false,
"leafnode_connections": 0 ,
"client_connections": 0 ,
"subscriptions": 42 ,
"exports": [
{
"subject": "$SYS.DEBUG.SUBSCRIBERS",
"type": "service",
"response_type": "Singleton"
}
],
"jwt": "eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJVVlU2VEpXR
"issuer_key": "OBU5O5FJ324UDPRBIVRGF7CNEOHGLPS7EYPBTVQZKSBHIIZIB6HD
"name_tag": "SYS",
"decoded_jwt": {
"jti": "UVU6TJWEO3KHXY6U2H33DBVIDOP7SNCNBO2S43WO5C6OTSL3UILA",
"iat": 1603473788 ,
"iss": "OBU5O5FJ324UDPRBIVRGF7CNEOHGLPS7EYPBTVQZKSBHIIZIB6HD66JF",
"name": "SYS",
"sub": "AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLKU7A5",
"nats": {
"limits": {
"subs": -1,
"data": -1,
"payload": -1,
"imports": -1,
"exports": -1,
"wildcards": true,
"conn": -1,
"leaf": -
},
"default_permissions": {
"pub": {},
"sub": {}
},
"type": "account",
"version": 1
}


#### The /accstatz endpoint reports per-account statistics such as the number of

#### connections, messages/bytes in/out, etc.

###### }

###### },

```
"sublist_stats": {
"num_subscriptions": 42 ,
"num_cache": 6 ,
"num_inserts": 42 ,
"num_removes": 0 ,
"num_matches": 6 ,
"cache_hit_rate": 0 ,
"max_fanout": 1 ,
"avg_fanout": 0.
}
}
}
```
```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
unused true, 1, false, 0
```
```
If true, include accounts that do not have
any current connections. Default is false.
```
#### Accounts with active connections - https://demo.nats.io:8222/accstatz

#### Include ones without any connections (in this case $SYS)-

#### https://demo.nats.io:8222/accstatz?unused=

### Account Statistics

#### Arguments

#### Examples

#### Response


#### The /jsz endpoint reports more detailed information on JetStream. For accounts, it

#### uses a paging mechanism that defaults to 1024 connections.

###### {

```
"server_id": "NDJ5M4F5WAIBUA26NJ3QMH532AQPN7QNTJP3Y4SBHSHL4Y7QUAKNJEAF"
"now": "2022-10-19T17:16:20.881296749Z",
"account_statz": [
{
"acc": "default",
"conns": 31 ,
"leafnodes": 2 ,
"total_conns": 33 ,
"num_subscriptions": 45 ,
"sent": {
"msgs": 1876970 ,
"bytes": 246705616
},
"received": {
"msgs": 1347454 ,
"bytes": 219438308
},
"slow_consumers": 29
},
{
"acc": "$G",
"conns": 1 ,
"leafnodes": 0 ,
"total_conns": 1 ,
"num_subscriptions": 3 ,
"sent": {
"msgs": 0 ,
"bytes": 0
},
"received": {
"msgs": 107 ,
"bytes": 1094
},
"slow_consumers": 0
}
]
}
```
### JetStream Information


#### Note: If you're in a clustered environment, it is recommended to retrieve the

#### information from the stream's leader in order to get the most accurate and up-to-date

#### data.

```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
acc account name
```
```
Include metrics for the specied
account. Default is unset.
```
```
accounts true, 1, false, 0
```
```
Include account specic JetStream
information. Default is false.
```
```
streams true, 1, false, 0
```
```
Include streams. When set, implies
accounts=true. Default is false.
```
```
consumers true, 1, false, 0
```
```
Include consumer. When set, implies
streams=true. Default is false.
```
```
cong true, 1, false, 0
```
```
When stream or consumer are requested, include
their respective conguration. Default is false.
```
```
leader-only true, 1, false, 0 Only the leader responds. Default is false.
```
```
offset number > 0 Pagination offset. Default is 0.
```
```
limit number > 0 Number of results to return. Default is 1024.
```
```
raft true, 1, false, 0
```
```
Include information details about
the Raft group. Default is false.
```
#### Arguments

#### Examples


#### Get basic JetStream information: https://demo.nats.io:8222/jsz

#### Request accounts and control limit and offset: https://demo.nats.io:8222/jsz?

#### accounts=true&limit=16&offset=128.

#### You can also report detailed consumer information on a per connection basis using

#### consumer=true. For example: https://demo.nats.io:8222/jsz?consumers=true.

#### Response


###### {

"server_id": "NCVIDODSZ45C5OD67ZD7EJUIJPQDP6CM74SJX6TJIF2G7NLYS5LCVYHS"
"now": "2021-02-08T19:08:30.555533-05:00",
"config": {
"max_memory": 10485760 ,
"max_storage": 10485760 ,
"store_dir": "/var/folders/9h/6g_c9l6n6bb8gp331d_9y0_w0000gn/T/srv_75
"unique_tag": "az"
},
"memory": 0 ,
"storage": 66 ,
"api": {
"total": 5 ,
"errors": 0
},
"total_streams": 1 ,
"total_consumers": 1 ,
"total_messages": 1 ,
"total_message_bytes": 33 ,
"meta_cluster": {
"name": "cluster_name",
"replicas": [
{
"name": "server_5500",
"current": false,
"active": 2932926000
}
]
},
"account_details": [
{
"name": "BCC_TO_HAVE_ONE_EXTRA",
"id": "BCC_TO_HAVE_ONE_EXTRA",
"memory": 0 ,
"storage": 0 ,
"api": {
"total": 0 ,
"errors": 0
}
},
{
"name": "ACC",
"id": "ACC",
"memory": 0 ,
"storage": 66 ,
"api": {
"total": 5 ,
"errors": 0


errors : 0
},
"stream_detail": [
{
"name": "my-stream-replicated",
"cluster": {
"name": "cluster_name",
"replicas": [
{
"name": "server_5500",
"current": false,
"active": 2931517000
}
]
},
"state": {
"messages": 1 ,
"bytes": 33 ,
"first_seq": 1 ,
"first_ts": "2021-02-09T00:08:27.623735Z",
"last_seq": 1 ,
"last_ts": "2021-02-09T00:08:27.623735Z",
"consumer_count": 1
},
"consumer_detail": [
{
"stream_name": "my-stream-replicated",
"name": "my-consumer-replicated",
"created": "2021-02-09T00:08:27.427631Z",
"delivered": {
"consumer_seq": 0 ,
"stream_seq": 0
},
"ack_floor": {
"consumer_seq": 0 ,
"stream_seq": 0
},
"num_ack_pending": 0 ,
"num_redelivered": 0 ,
"num_waiting": 0 ,
"num_pending": 1 ,
"cluster": {
"name": "cluster_name",
"replicas": [
{
"name": "server_5500",
"current": false,
"active": 2933232000
}


#### The /healthz endpoint returns OK if the server is able to accept connections.

###### ] } } ] } ] } ] }

```
Result Return Code
```
```
Success 200 (OK)
```
```
Error 400 (Bad Request)
```
```
Argument Values Description
```
```
js-enabled-only true, 1 Returns an error if JetStream is disabled.
```
```
js-server-only true, 1
```
```
Skip health check of accounts,
streams, and consumers.
```
```
js-enabled true, 1
```
```
Returns an error if JetStream is disabled.
( Deprecated : use js-enabled-only instead).
```
#### Default - https://demo.nats.io:8222/healthz

#### Expect JetStream - https://demo.nats.io:8222/healthz?js-enabled-only=true

```
{ "status": "ok" }
```
### Health

#### Arguments

#### Example

#### Response


#### NATS monitoring endpoints support JSONP and CORS. You can easily create single page

#### web applications for monitoring. To do this you simply pass the callback query

#### parameter to any endpoint.

#### For example:

#### Here is a JQuery example implementation:

#### In addition to writing custom monitoring tools, you can monitor nats-server in

#### Prometheus. The Prometheus NATS Exporter allows you to congure the metrics you

#### want to observe and store in Prometheus, and there are Grafana dashboards available for

#### you to visualize the server metrics.

#### See the Walkthrough of Monitoring NATS with Prometheus and Grafana for more details.

```
https://demo.nats.io:8222/connz?callback=cb
```
```
$.getJSON("https://demo.nats.io:8222/connz?callback=?", function (data) {
console.log(data);
});
```
## Creating Monitoring Applications

## Monitoring Tools


# Monitoring JetStream

#### JetStream has a /jsz HTTP endpoint and advisories available.

#### JetStream publishes a number of advisories that can inform operations about the health

#### and the state of the Streams. These advisories are published to normal NATS subjects

#### below $JS.EVENT.ADVISORY.> and one can store these advisories in JetStream

#### Streams if desired.

#### The command nats event --js-advisory can view all these events on your console.

#### The Golang package jsm.go can consume and render these events and have data types

#### for each of these events.

#### All these events have JSON Schemas that describe them, schemas can be viewed on the

#### CLI using the nats schema show <schema kind> command.

```
Description Subject Kind
```
```
API interactions $JS.EVENT.ADVISORY.API
```
```
io.nats.jetstream.advis
ory.v1.api_audit
```
```
Stream CRUD operations
```
```
$JS.EVENT.ADVISORY.STREAM
.CREATED.<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.stream_action
```
```
Consumer CRUD
operations
```
```
$JS.EVENT.ADVISORY.CONSUM
ER.CREATED.<STREAM>.
<CONSUMER>
```
```
io.nats.jetstream.advis
ory.v1.consumer_action
```
```
Snapshot started using
nats stream backup
```
```
$JS.EVENT.ADVISORY.STREAM
.SNAPSHOT_CREATE.<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.snapshot_create
```
## Server Metrics

## Advisories


**Description Subject Kind**

Snapshot completed

```
$JS.EVENT.ADVISORY.STREAM
.SNAPSHOT_COMPLETE.
<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.snapshot_complet
e
```
Restore started using
nats stream restore

```
$JS.EVENT.ADVISORY.STREAM
.RESTORE_CREATE.<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.restore_create
```
Restore completed

```
$JS.EVENT.ADVISORY.STREAM
.RESTORE_COMPLETE.
<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.restore_complete
```
Consumer maximum
delivery reached

```
$JS.EVENT.ADVISORY.CONSUM
ER.MAX_DELIVERIES.
<STREAM>.<CONSUMER>
```
```
io.nats.jetstream.advis
ory.v1.max_deliver
```
Message delivery
naked using AckNak

```
$JS.EVENT.ADVISORY.CONSUM
ER.MSG_NAKED.<STREAM>.
<CONSUMER>
```
```
io.nats.jetstream.advis
ory.v1.nak
```
Message delivery
terminated using AckTerm

```
$JS.EVENT.ADVISORY.CONSUM
ER.MSG_TERMINATED.
<STREAM>.<CONSUMER>
```
```
io.nats.jetstream.advis
ory.v1.terminated
```
Message acknowledged
in a sampled Consumer

```
$JS.EVENT.METRIC.CONSUMER
.ACK.<STREAM>.<CONSUMER>
```
```
io.nats.jetstream.metri
c.v1.consumer_ack
```
Clustered Stream
elected a new leader

```
$JS.EVENT.ADVISORY.STREAM
.LEADER_ELECTED.<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.stream_leader_el
ected
```
Clustered Stream
lost quorum

```
$JS.EVENT.ADVISORY.STREAM
.QUORUM_LOST.<STREAM>
```
```
io.nats.jetstream.advis
ory.v1.stream_quorum_lo
st
```
Clustered Consumer
elected a new leader

```
$JS.EVENT.ADVISORY.CONSUM
ER.LEADER_ELECTED.
<STREAM>.<CONSUMER>
```
```
io.nats.jetstream.advis
ory.v1.consumer_leader_
elected
```

#### The NATS Surveyor system has initial support for passing JetStream metrics to

#### Prometheus, dashboards and more will be added towards nal release.

```
Description Subject Kind
```
```
Clustered Consumer
lost quorum
```
```
$JS.EVENT.ADVISORY.CONSUM
ER.QUORUM_LOST.<STREAM>.
<CONSUMER>
```
```
io.nats.jetstream.advis
ory.v1.consumer_quorum_
lost
```
## Dashboards


# Managing JetStream

#### Once the server is running it's time to use the management tool. This can be downloaded

#### from the GitHub Release Page. On OS X homebrew can be used to install the latest

#### version:

#### We'll walk through the above scenario and introduce features of the CLI and of JetStream

#### as we recreate the setup above.

#### Throughout this example, we'll show other commands like nats pub and nats sub to

#### interact with the system. These are normal existing core NATS commands and JetStream

#### is fully usable by only using core NATS.

#### We'll touch on some additional features but please review the section on the design

#### model to understand all possible permutations.

```
brew tap nats-io/nats-tools
brew install nats-io/nats-tools/nats
nats --help
nats cheat
```

# Account Information

#### JetStream is multi-tenant so you will need to check that your account is enabled for

#### JetStream and is not limited. You can view your limits as follows:

```
nats account info
```
```
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
## Account Information


# Naming Streams,

# Consumers, and Accounts

#### Stream, Consumer (durable name), and Account names are used in both the subject

#### namespace used by JetStream and the lesystem backing JetStream persistence. This

#### means that when naming streams, consumers, and accounts, names must adhere to

#### subject naming rules as well as being friendly to the le system.

#### We recommend the following guideline for stream, consumer, and account names:

#### We plan to address these limitations in a future release.

#### Alphanumeric values are recommended.

#### Spaces, tabs, period (. ), greater than ( > ) or asterisk ( * ) are prohibited.

#### Path separators (i.e. forward slash and backward slash) are prohibited.

#### Limit name length: The JetStream storage directories will include the account, stream

#### name, and consumer name, so a generally safe approach would be to keep names

#### under 32 characters.

#### Do not use reserved le names like NUL, LPT1, etc.

#### Be aware that some le systems are case insensitive so do not use stream or account

#### names that would collide in a le system. For example, Foo and foo would collide

#### on a Windows or Mac OSx System.


# Streams

#### The rst step is to set up storage for our ORDERS related messages, these arrive on a

#### wildcard of subjects all owing into the same Stream and they are kept for 1 year.

```
nats str add ORDERS
```
```
? Subjects to consume ORDERS.*
? Storage backend file
? Retention Policy Limits
? Discard Policy Old
? Message count limit -1
? Message size limit -1
? Maximum message age limit 1y
? Maximum individual message size [? for help] (-1) -1
Stream ORDERS was created
```
```
Information for Stream ORDERS
```
```
Configuration:
```
```
Subjects: ORDERS.*
Acknowledgements: true
Retention: File - Limits
Replicas: 1
Maximum Messages: -1
Maximum Bytes: -1
Maximum Age: 8760h0m0s
Maximum Message Size: -1
Maximum Consumers: -1
```
```
Statistics:
```
```
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```
## Creating


#### You can get prompted interactively for missing information as above, or do it all on one

#### command. Pressing? in the CLI will help you map prompts to CLI options:

#### Additionally one can store the conguration in a JSON le, the format of this is the same

#### as $ nats str info ORDERS -j | jq .config:

#### We can conrm our Stream was created:

#### Information about the conguration of the Stream can be seen, and if you did not specify

#### the Stream like below, it will prompt you based on all known ones:

```
nats str add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes
```
```
nats str add ORDERS --config orders.json
```
```
nats str ls
```
```
Streams:
```
```
ORDERS
```
```
nats str info ORDERS
```
## Listing

## Querying


#### Most commands that show data as above support -j to show the results as JSON:

```
Information for Stream ORDERS created 2021-02-27T16:49:36-07:00
```
```
Configuration:
```
```
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
```
```
State:
```
```
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```
```
nats str info ORDERS -j
```

#### This is the general pattern for the entire nats utility as it relates to JetStream -

#### prompting for needed information but every action can be run non-interactively making it

#### usable as a CLI API. All information output like seen above can be turned into JSON using

#### -j.

#### A stream can be copied into another, which also allows the conguration of the new one

#### to be adjusted via CLI ags:

###### {

```
"config": {
"name": "ORDERS",
"subjects": [
"ORDERS.*"
],
"retention": "limits",
"max_consumers": -1,
"max_msgs": -1,
"max_bytes": -1,
"max_age": 31536000000000000 ,
"max_msg_size": -1,
"storage": "file",
"discard": "old",
"num_replicas": 1 ,
"duplicate_window": 120000000000
},
"created": "2021-02-27T23:49:36.700424Z",
"state": {
"messages": 0 ,
"bytes": 0 ,
"first_seq": 0 ,
"first_ts": "0001-01-01T00:00:00Z",
"last_seq": 0 ,
"last_ts": "0001-01-01T00:00:00Z",
"consumer_count": 0
}
}
```
```
nats str cp ORDERS ARCHIVE --subjects "ORDERS_ARCHIVE.*" --max-age 2 y
```
## Copying


#### A stream conguration can be edited, which allows the conguration to be adjusted via

#### CLI ags. Here I have an incorrectly created ORDERS stream that I x:

#### Change the subjects for the stream

```
Stream ORDERS was created
```
```
Information for Stream ORDERS created 2021-02-27T16:52:46-07:00
```
```
Configuration:
```
```
Subjects: ORDERS_ARCHIVE.*
Acknowledgements: true
Retention: File - Limits
Replicas: 1
Discard Policy: Old
Duplicate Window: 2m0s
Maximum Messages: unlimited
Maximum Bytes: unlimited
Maximum Age: 2y0d0h0m0s
Maximum Message Size: unlimited
Maximum Consumers: unlimited
```
```
State:
```
```
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```
```
nats str info ORDERS -j | jq .config.subjects
```
###### [

```
"ORDERS.new"
]
```
```
nats str edit ORDERS --subjects "ORDERS.*"
```
## Editing


#### Additionally, one can store the conguration in a JSON le, the format of this is the same

#### as $ nats str info ORDERS -j | jq .config:

#### Now let's add some messages to our Stream. You can use nats pub to add messages,

#### pass the --wait ag t o see the publish ack being returned.

#### You can publish without waiting for acknowledgement:

#### But if you want to be sure your messages got to JetStream and were persisted you can

#### make a request:

#### Keep checking the status of the Stream while doing this and you'll see its stored

#### messages increase.

```
Stream ORDERS was updated
```
```
Information for Stream ORDERS
```
```
Configuration:
```
```
Subjects: ORDERS.*
....
```
```
nats str edit ORDERS --config orders.json
```
```
nats pub ORDERS.scratch hello
```
```
nats req ORDERS.scratch hello
```
```
13:45:03 Sending request on [ORDERS.scratch]
13:45:03 Received on [_INBOX.M8drJkd8O5otORAo0sMNkg.scHnSafY]: '+OK'
```
```
nats str info ORDERS
```
## Publishing Into a Stream


#### After putting some throwaway data into the Stream, we can purge all the data out - while

#### keeping the Stream active:

#### To delete all data in a stream use purge:

#### A single message can be securely removed from the stream:

```
Information for Stream ORDERS
...
Statistics:
```
```
Messages: 3
Bytes: 147 B
FirstSeq: 1
LastSeq: 3
Active Consumers: 0
```
```
nats str purge ORDERS -f
```
###### ...

```
State:
```
```
Messages: 0
Bytes: 0 B
FirstSeq: 1,000,001
LastSeq: 1,000,000
Active Consumers: 0
```
```
nats str rmm ORDERS 1 -f
```
## Deleting All Data

## Deleting A Message


#### Finally, for demonstration purposes, you can also delete the whole Stream and recreate it.

#### Then we're ready for creating the Consumers:

```
nats str rm ORDERS -f
nats str add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes
```
## Deleting Sets


# Consumers

#### Messages are read or consumed from the Stream by Consumers. We support pull and

#### push-based Consumers and the example scenario has both, let's walk through that.

#### The NEW and DISPATCH Consumers are pull-based, meaning the services consuming

#### data from them have to ask the system for the next available message. This means you

#### can easily scale your services up by adding more workers and the messages will get

#### spread across the workers based on their availability.

#### Pull-based Consumers are created the same as push-based Consumers, you just don't

#### specify a delivery target.

#### We have no Consumers, lets add the NEW one:

#### I supply the --sample options on the CLI as this is not prompted for at present,

#### everything else is prompted. The help in the CLI explains each:

```
nats con ls ORDERS
```
```
No Consumers defined
```
```
nats con add --sample 100
```
## Creating Pull-Based Consumers


#### This is a pull-based Consumer (empty Delivery Target), it gets messages from the rst

#### available message and requires specic acknowledgement of each and every message.

#### It only received messages that originally entered the Stream on ORDERS.received.

#### Remember the Stream subscribes to ORDERS.*, this lets us select a subset of messages

#### from the Stream.

#### A Maximum Delivery limit of 20 is set, this means if the message is not acknowledged it

#### will be retried but only up to this maximum total deliveries.

#### Again this can all be done in a single CLI call, lets make the DISPATCH Consumer:

```
? Select a Stream ORDERS
? Consumer name NEW
? Delivery target
? Start policy (all, last, 1h, msg sequence) all
? Filter Stream by subject (blank for all) ORDERS.received
? Maximum Allowed Deliveries 20
Information for Consumer ORDERS > NEW
```
```
Configuration:
```
```
Durable Name: NEW
Pull Mode: true
Subject: ORDERS.received
Deliver All: true
Deliver Last: false
Ack Policy: explicit
Ack Wait: 30s
Replay Policy: instant
Maximum Deliveries: 20
Sampling Rate: 100
```
```
State:
```
```
Last Delivered Message: Consumer sequence: 1 Stream sequence: 1
Acknowledgment floor: Consumer sequence: 0 Stream sequence: 0
Pending Messages: 0
Redelivered Messages: 0
```
```
nats con add ORDERS DISPATCH --filter ORDERS.processed --ack explicit --p
```

#### Additionally, one can store the conguration in a JSON le, the format of this is the same

#### as $ nats con info ORDERS DISPATCH -j | jq .config:

#### Our MONITOR Consumer is push-based, has no ack and will only get new messages and

#### is not sampled:

#### Again you can do this with a single non-interactive command:

```
nats con add ORDERS MONITOR --config monitor.json
```
```
nats con add
```
```
? Select a Stream ORDERS
? Consumer name MONITOR
? Delivery target monitor.ORDERS
? Start policy (all, last, 1h, msg sequence) last
? Acknowledgement policy none
? Replay policy instant
? Filter Stream by subject (blank for all)
? Maximum Allowed Deliveries -1
Information for Consumer ORDERS > MONITOR
```
```
Configuration:
```
```
Durable Name: MONITOR
Delivery Subject: monitor.ORDERS
Deliver All: false
Deliver Last: true
Ack Policy: none
Replay Policy: instant
```
```
State:
```
```
Last Delivered Message: Consumer sequence: 1 Stream sequence: 3
Acknowledgment floor: Consumer sequence: 0 Stream sequence: 2
Pending Messages: 0
Redelivered Messages: 0
```
## Creating Push-Based Consumers


#### Additionally one can store the conguration in a JSON le, the format of this is the same

#### as $ nats con info ORDERS MONITOR -j | jq .config:

#### You can get a quick list of all the Consumers for a specic Stream:

#### All details for a Consumer can be queried, lets rst look at a pull-based Consumer:

```
nats con add ORDERS MONITOR --ack none --target monitor.ORDERS --deliver
```
```
nats con add ORDERS --config monitor.json
```
```
nats con ls ORDERS
```
```
Consumers for Stream ORDERS:
```
```
DISPATCH
MONITOR
NEW
```
## Listing

## Querying


#### More details about the State section will be shown later when discussing the ack

#### models in depth.

#### The two number are not directly related: the Stream sequence number is the pointer to

#### the exact message, while the Consumer sequence number is an ever-increasing counter

#### for consumer actions.

#### So for example a stream with 1 message in it would have stream sequence of 1, but if the

#### consumer attempted 10 deliveries of that message consumer sequence would be 10 or

#### 11.

#### Pull-based Consumers require you to specically ask for messages and ack them,

#### typically you would do this with the client library Request() feature, but the nats utility

```
$ nats con info ORDERS DISPATCH
Information for Consumer ORDERS > DISPATCH
```
```
Configuration:
```
```
Durable Name: DISPATCH
Pull Mode: true
Subject: ORDERS.processed
Deliver All: true
Deliver Last: false
Ack Policy: explicit
Ack Wait: 30s
Replay Policy: instant
Sampling Rate: 100
```
```
State:
```
```
Last Delivered Message: Consumer sequence: 1 Stream sequence: 1
Acknowledgment floor: Consumer sequence: 0 Stream sequence: 0
Pending Messages: 0
Redelivered Messages: 0
```
### Stream vs Consumer sequence numbers

## Consuming Pull-Based Consumers


#### has a helper:

#### First, we ensure we have a message:

#### We can now read them using nats:

#### Consumer another one

#### You can prevent ACKs by supplying --no-ack.

#### To do this from code you'd send a Request() to

#### $JS.API.CONSUMER.MSG.NEXT.ORDERS.DISPATCH:

```
nats pub ORDERS.processed "order 1"
nats pub ORDERS.processed "order 2"
nats pub ORDERS.processed "order 3"
```
```
nats con next ORDERS DISPATCH
```
```
--- received on ORDERS.processed
order 1
```
```
Acknowledged message
```
```
nats con next ORDERS DISPATCH
```
```
--- received on ORDERS.processed
order 2
```
```
Acknowledged message
```
```
nats req '$JS.API.CONSUMER.MSG.NEXT.ORDERS.DISPATCH' ''
```
```
Published [$JS.API.CONSUMER.MSG.NEXT.ORDERS.DISPATCH] : ''
Received [ORDERS.processed] : 'order 3'
```

#### Here nats req cannot ack, but in your code you'd respond to the received message with

#### a nil payload as an Ack to JetStream.

#### Push-based Consumers will publish messages to a subject and anyone who subscribes

#### to the subject will get them, they support different Acknowledgement models covered

#### later, but here on the MONITOR Consumer we have no Acknowledgement.

#### Output extract

#### The Consumer is publishing to that subject, so let's listen there:

#### Note the subject here of the received message is reported as ORDERS.processed this

#### helps you distinguish what you're seeing in a Stream covering a wildcard, or multiple

#### subjects, subject space.

#### This Consumer needs no ack, so any new message into the ORDERS system will show up

#### here in real-time.

```
nats con info ORDERS MONITOR
```
###### ...

```
Delivery Subject: monitor.ORDERS
...
```
```
nats sub monitor.ORDERS
```
```
Listening on [monitor.ORDERS]
[#3] Received on [ORDERS.processed]: 'order 3'
[#4] Received on [ORDERS.processed]: 'order 4'
```
## Consuming Push-Based Consumers


# Data Replication

#### Replication allows you to move data between streams in either a 1:1 mirror style or by

#### multiplexing multiple source streams into a new stream. In future builds this will allow

#### data to be replicated between accounts as well, ideal for sending data from a Leafnode

#### into a central store.

#### Here we have 2 main streams - ORDERS and RETURNS - these streams are clustered

#### across 3 nodes. These Streams have short retention periods and are memory based.

#### We create a ARCHIVE stream that has 2 sources set, the ARCHIVE will pull data from the

#### sources into itself. This stream has a very long retention period and is le based and

#### replicated across 3 nodes. Additional messages can be added to the ARCHIVE by

#### sending to it directly.

#### Finally, we create a REPORT stream mirrored from ARCHIVE that is not clustered and

#### retains data for a month. The REPORT Stream does not listen for any incoming

#### messages, it can only consume data from ARCHIVE.


#### A mirror copies data from 1 other stream, as far as possible IDs and ordering will match

#### exactly the source. A mirror does not listen on a subject for any data to be added. A

#### mirror can lter by subject and the Start Sequence and Start Time can be set. A stream

#### can only have 1 mirror and if it is a mirror it cannot also have any source.

#### A source is a stream where data is copied from, one stream can have multiple sources

#### and will read data in from them all. The stream will also listen for messages on it's own

#### subject. We can therefore not maintain absolute ordering, but data from 1 single source

#### will be in the correct order but mixed in with other streams. You might also nd the

#### timestamps of streams can be older and newer mixed in together as a result.

#### A Stream with sources may also listen on subjects, but could have no listening subject.

#### When using the nats CLI to create sourced streams use --subjects to supply

#### subjects to listen on.

#### A source can have Start Time or Start Sequence and can lter by a subject.

#### The ORDERS and RETURNS streams as normal, I will not show how to create them.

```
nats s report
```
## Mirrors

## Sources

## Configuration


#### We now add the ARCHIVE:

```
Obtaining Stream stats
```
```
+---------+---------+-----------+----------+-------+------+---------+----
| Stream | Storage | Consumers | Messages | Bytes | Lost | Deleted | Clu
+---------+---------+-----------+----------+-------+------+---------+----
| ORDERS | Memory | 0 | 0 | 0 B | 0 | 0 | n1-
| RETURNS | Memory | 0 | 0 | 0 B | 0 | 0 | n1-
+---------+---------+-----------+----------+-------+------+---------+----
```
```
nats s add ARCHIVE --source ORDERS --source RETURNS
```

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
Sources: ORDERS
RETURNS

State:

Messages: 0
Bytes: 0 B


#### And we add the REPORT:

```
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```
```
nats s add REPORT --mirror ARCHIVE
```

#### When congured we'll see some additional information in a nats stream info output:

```
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
```
Information for Stream REPORT created 2022-01-21T11:50:55-08:00
```
```
Configuration:
```
```
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
Mirror: ARCHIVE
```
```
State:
```
```
Messages: 0
Bytes: 0 B
FirstSeq: 0
LastSeq: 0
Active Consumers: 0
```

#### Output extract

#### Here the Lag is how far behind we were reported as being last time we saw a message.

#### We can conrm all our setup using a nats stream report:

```
nats stream info ARCHIVE
```
###### ...

```
Source Information:
```
```
Stream Name: ORDERS
Lag: 0
Last Seen: 2m23s
```
```
Stream Name: RETURNS
Lag: 0
Last Seen: 2m15s
...
```
```
$ nats stream info REPORT
...
Mirror Information:
```
```
Stream Name: ARCHIVE
Lag: 0
Last Seen: 2m35s
...
```
```
nats s report
```

#### We then create some data in both ORDERS and RETURNS:

#### We can now see from a Stream Report that the data has been replicated:

###### +------------------------------------------------------------------------

```
| Stream Report
+---------+---------+-------------+-----------+----------+-------+------+
| Stream | Storage | Replication | Consumers | Messages | Bytes | Lost |
+---------+---------+-------------+-----------+----------+-------+------+
| ARCHIVE | File | Sourced | 1 | 0 | 0 B | 0 |
| ORDERS | Memory | | 1 | 0 | 0 B | 0 |
| REPORT | File | Mirror | 0 | 0 | 0 B | 0 |
| RETURNS | Memory | | 1 | 0 | 0 B | 0 |
+---------+---------+-------------+-----------+----------+-------+------+
```
```
+---------------------------------------------------------+
| Replication Report |
+---------+--------+---------------+--------+-----+-------+
| Stream | Kind | Source Stream | Active | Lag | Error |
+---------+--------+---------------+--------+-----+-------+
| ARCHIVE | Source | ORDERS | never | 0 | |
| ARCHIVE | Source | RETURNS | never | 0 | |
| REPORT | Mirror | ARCHIVE | never | 0 | |
+---------+--------+---------------+--------+-----+-------+
```
```
nats req ORDERS.new "ORDER {{Count}}" --count 100
nats req RETURNS.new "RETURN {{Count}}" --count 100
```
```
nats s report --dot replication.dot
```

#### Here we also pass the --dot replication.dot argument that writes a GraphViz format

#### map of the replication setup.

```
Obtaining Stream stats
```
```
+---------+---------+-----------+----------+---------+------+---------+--
| Stream | Storage | Consumers | Messages | Bytes | Lost | Deleted | C
+---------+---------+-----------+----------+---------+------+---------+--
| ORDERS | Memory | 1 | 100 | 3.3 KiB | 0 | 0 | n
| RETURNS | Memory | 1 | 100 | 3.5 KiB | 0 | 0 | n
| ARCHIVE | File | 1 | 200 | 27 KiB | 0 | 0 | n
| REPORT | File | 0 | 200 | 27 KiB | 0 | 0 | n
+---------+---------+-----------+----------+---------+------+---------+--
```
```
+---------------------------------------------------------+
| Replication Report |
+---------+--------+---------------+--------+-----+-------+
| Stream | Kind | Source Stream | Active | Lag | Error |
+---------+--------+---------------+--------+-----+-------+
| ARCHIVE | Source | ORDERS | 14.48s | 0 | |
| ARCHIVE | Source | RETURNS | 9.83s | 0 | |
| REPORT | Mirror | ARCHIVE | 9.82s | 0 | |
+---------+--------+---------------+--------+-----+-------+
```

# Disaster Recovery

#### In the event of unrecoverable JetStream message persistence on one (or more) server

#### nodes, there are two recovery scenarios:

```
For R1 streams, data is persisted on one server node only. If that server node is
unrecoverable then recovery from backup is the sole option.
```
#### NATS will create replacement stream replicas automatically under the following

#### conditions:

#### Snapshots (also known as backups) can pro-actively be made of any stream regardless

#### of replication conguration.

#### The backup includes (by default):

#### Automatic recovery from intact quorum nodes

#### Manual recovery from existing stream snapshots (backups)

#### Impacted stream is of replica conguration R3 (or greater)

#### Remaining intact nodes (stream replicas) meet minimum RAFT quorum: oor(R/2) + 1

#### Available node(s) in the stream's cluster for new replica(s)

#### Impacted node(s) removed from the stream's domain RAFT Meta group (e.g.

#### nats server raft peer-remove)

#### Stream conguration and state

#### Stream durable consumer conguration and state

#### All message payload data including metadata like timestamps and headers

## Automatic Recovery

## Manual Recovery


#### The nats stream backup CLI command is used to create snapshots of a stream and

#### its durable consumers.

```
As an account owner, if you wish to make a backup of ALL streams in your account, you may
use nats account backup instead.
```
#### Output

#### During a backup operation, the stream is placed in a status where it's conguration

#### cannot change and no data will be evicted based on stream retention policies.

```
Progress using the terminal bar can be disabled using --no-progress, it will then issue log
lines instead.
```
#### An existing backup (as above) can be restored to the same or a new NATS server (or

#### cluster) using the nats stream restore command.

```
If there are multiple streams in the backup directory, they will all be restored.
```
```
nats stream backup ORDERS '/data/js-backup/backup1'
```
```
Starting backup of Stream "ORDERS" with 13 data blocks
```
```
2.4 MiB/s [==============================================================
```
```
Received 13 MiB bytes of compressed data in 3368 chunks for stream "ORDER
```
```
nats stream restore '/data/js-backup/backup1'
```
### Backup

### Restore


#### Output

#### The /data/js-backup/ORDERS.tgz le can also be extracted into the data dir of a

#### stopped NATS Server.

#### Progress using the terminal bar can be disabled using --no-progress, it will then issue

#### log lines instead.

#### In environments where the nats CLI is used interactively to congure the server you do

#### not have a desired state to recreate the server from. This is not the ideal way to

#### administer the server, we recommend Conguration Management, but many will use this

#### approach.

#### Here you can back up the conguration into a directory from where you can recover the

#### conguration later. The data for File backed stores can also be backed up.

```
Starting restore of Stream "ORDERS" from file "/data/js-backup/backup1"
```
```
13 MiB/s [===============================================================
```
```
Restored stream "ORDERS" in 937.071149ms
```
```
Information for Stream ORDERS
```
```
Configuration:
```
```
Subjects: ORDERS.>
...
```
```
nats account backup /data/js-backup
```
```
15:56:11 Creating JetStream backup into /data/js-backup
15:56:11 Stream ORDERS to /data/js-backup/stream_ORDERS.json
15:56:11 Consumer ORDERS > NEW to /data/js-backup/stream_ORDERS_consumer_
15:56:11 Configuration backup complete
```
## Interactive CLI


#### This backs up Stream and Consumer conguration.

#### During the same process the data can also be backed up by passing --data, this will

#### create les like /data/js-backup/stream_ORDERS.tgz.

#### Later the data can be restored, for Streams we support editing the Stream conguration

#### in place to match what was in the backup.

#### The nats account restore tool does not support restoring data, the same process

#### using nats stream restore, as outlined earlier, can be used which will also restore

#### Stream and Consumer congurations and state.

```
On restore, if a stream already exists in the server of same name and account, you will
receive a Stream {name} already exist error.
```
```
nats account restore /tmp/backup --update-streams
```
```
15:57:42 Reading file /tmp/backup/stream_ORDERS.json
15:57:42 Reading file /tmp/backup/stream_ORDERS_consumer_NEW.json
15:57:42 Updating Stream ORDERS configuration
15:57:42 Restoring Consumer ORDERS > NEW
```

# Encryption at Rest

#### Supported since NATS server version 2.3.0

```
Note, although this feature is supported, we recommend le system encryption if available.
```
#### The NATS server can be congured to encrypt message blocks which includes message

#### headers and payloads. Other metadata les are encrypted as well, such as the stream

#### metadata le and consumer metadata les.

#### Two choices of ciphers are currently supported:

#### Enabling encryption is done through the jetstream conguration block on the server.

#### It is recommended to provide the encryption key through an environment variable at

#### runtime, such as $JS_KEY, so it will not be persisted in a le.

#### The variable can be exported in the environment or passed when the server starts up.

#### chachapoly - ChaCha20-Poly1305

#### aes - AES-GCM

```
jetstream : {
cipher: chachapoly
key : "6dYfBV0zzEkR3vxZCNjxmnVh/aIqgid1"
}
```
```
jetstream : {
cipher: chachapoly
key: $JS_KEY
}
```
```
JS_KEY="mykey" nats-server -c js.conf
```

#### Enabling encryption on a server with existing data is supported. Do note that existing

#### unencrypted message blocks will not be re-encrypted, however any new blocks that are

#### stored will be encrypted going forward.

#### If it is desired to encrypt the existing blocks, the stream can be backed up and restored

#### (which decrypts on backup and then re-encrypts when restoring it).

#### If encryption was enabled on the server and the server is restarted with a different key or

#### disabled all together, the server will fail to decrypt messages when attempting to load

#### them from the store. If this happens, you’ll see log messages like the following:

#### Note, that this will impact JetStream functionality, but the server will still support core

#### NATS functionality.

#### It is possible to change the cipher, however the same key must be used. The server will

#### properly encrypt new message blocks with the new cipher and decrypt existing messages

#### blocks with the existing cipher.

```
Error decrypting our stream metafile: chacha20poly1305: message authentic
```
## Changing encryption settings

### Enabling with existing data

### Disabling or changing the key

### Changing the cipher

## Performance considerations


#### Performance considerations: As expected, encryption is likely to decrease performance,

#### but by how much is hard to dene. In some performance tests on a MacbookPro 2.8 GHz

#### Intel Core i7 with SSD, we have observed as little as 1% decrease to more than 30%. In

#### addition to CPU cycles required for encryption, the encrypted les may be larger, which

#### results in more data being stored or read.


# Managing JWT Security

#### If you are using the JWT model of authentication to secure your NATS infrastructure or

#### implementing an Auth callout service, you can administer authentication and

#### authorization without having to change the servers' conguration les.

#### You can use the nsc CLI tool to manage identities. Identities take the form of nkeys.

#### Nkeys are a public-key signature system based on Ed25519 for the NATS ecosystem. The

#### nkey identities are associated with NATS conguration in the form of a JSON Web Token

#### (JWT). The JWT is digitally signed by the private key of an issuer forming a chain of trust.

#### The nsc tool creates and manages these identities and allows you to deploy them to a

#### JWT account server, which in turn makes the congurations available to nats-servers.

#### You can also use nk CLI tool and library to manage keys.

#### You can create, update and delete accounts and users programmatically using the

#### following libraries:

#### You can integrate NATS with your existing authentication/authorization system or create

#### your own custom authentication using the Auth callout.

#### Golang: see NKEYS and JWT.

#### Java: see NKey.java and JwtUtils.java

## Creating, updating and managing JWTs

## programmatically

## Integrating with your existing

## authentication/authorization system

## Examples


#### See NATS by Example under "Authentication and Authorization" for JWT and Auth callout

#### server implementation examples.

#### User creation in Golang

#### Golang example from https://natsbyexample.com/examples/auth/nkeys-jwts/go

#### User JWTs

#### Account JWTs


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

```
var (
accountSeed string
operatorSeed string
name string
)
```
```
flag.StringVar(&operatorSeed, "operator", "", "Operator seed for crea
flag.StringVar(&accountSeed, "account", "", "Account seed for creatin
flag.StringVar(&name, "name", "", "Account or user name to be created
```
```
flag.Parse()
```
```
if accountSeed != "" && operatorSeed != "" {
log.Fatal("operator and account cannot both be provided")
}
```
```
var (
jwt string
err error
)
```
```
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
```
```
fmt.Println(jwt)
```

fmt.Println(jwt)
}

func createAccount(operatorSeed, accountName string) (string, error) {
akp, err := nkeys.CreateAccount()
if err != nil {
return "", fmt.Errorf("unable to create account using nkeys: %w",
}

```
apub, err := akp.PublicKey()
if err != nil {
return "", fmt.Errorf("unable to retrieve public key: %w", err)
}
```
```
ac := jwt.NewAccountClaims(apub)
ac.Name = accountName
```
```
// Load operator key pair
okp, err := nkeys.FromSeed([]byte(operatorSeed))
if err != nil {
return "", fmt.Errorf("unable to create operator key pair from se
}
```
```
// Sign the account claims and convert it into a JWT string
ajwt, err := ac.Encode(okp)
if err != nil {
return "", fmt.Errorf("unable to sign the claims: %w", err)
}
```
return ajwt, nil
}

func createUser(accountSeed, userName string) (string, error) {
ukp, err := nkeys.CreateUser()
if err != nil {
return "", fmt.Errorf("unable to create user using nkeys: %w", er
}

```
upub, err := ukp.PublicKey()
if err != nil {
return "", fmt.Errorf("unable to retrieve public key: %w", err)
}
```
```
uc := jwt.NewUserClaims(upub)
uc.Name = userName
```
```
// Load account key pair
akp, err := nkeys.FromSeed([]byte(accountSeed))
if err != nil {
```

#### You can see the key (and any signing keys) of your operator using

#### nsc list keys --show-seeds, you should use a 'signing key' to create the account

#### JWTs (as singing keys can be revoked/rotated easily)

#### To delete accounts use the "$SYS.REQ.CLAIMS.DELETE" (see reference) and make sure

#### to enable JWT deletion in your nats-server resolver (config allow_delete: true in the

#### resolver stanza of the server conguration).

#### The system is just like any other account, the only difference is that it is listed as system

#### account in the operator's JWT (and the server cong).

```
return "", fmt.Errorf("unable to create account key pair from see
}
```
```
// Sign the user claims and convert it into a JWT string
ujwt, err := uc.Encode(akp)
if err != nil {
return "", fmt.Errorf("unable to sign the claims: %w", err)
}
```
```
return ujwt, nil
}
```
### Notes


# In Depth JWT Guide

#### This document provides a step by step deep dive into JWT usage within NATS. Starting

#### with related concepts, it will introduce JWTs and how they can be used in NATS. This will

#### NOT list every JWT/nsc option, but will focus on the important options and concepts.

#### Concepts

#### What are Accounts?

#### Key Takeaways

#### What are NKEYs?

#### Key Takeaways

#### JSON Web Tokens (JWT)

#### Motivation for JWT

#### Key Takeaways

#### Decentralized Authentication/Authorization using JWT

#### Key Takeaways

#### NATS JWT Hierarchy

#### Decentralized Chain of Trust

#### Obtain an Account JWT

#### JWT and Chain of Trust Verication

#### Obtain a User JWT - Client Connect

#### Key Takeaways

#### Deployment Models Enabled by Chain of Trust

#### Key Takeaways

#### Accounts Re-visited

#### Key Takeaways

#### Tooling And Key Management

#### nsc

#### Environment

#### Backup

#### NKEYS store directory


#### JWT store directory

#### Names in JWT

#### Setup an Operator

#### Create/Edit Operator - Operator Environment - All Deployment modes

#### Import Operator - Non Operator/Administrator Environment -

#### Decentralized/Self Service Deployment Modes

#### Import Operator - Self Service Deployment Modes

#### Setup an Account

#### Create/Edit Account - All Environments - All Deployment modes

#### Export Account - Non Operator/Administrator Environment - Decentralized

#### Deployment Modes

#### Export Account - Non Operator/Administrator Environment - Self Service

#### Deployment Modes

#### Publicize an Account with Push - Operator Environment/Environment with push

#### permissions - All Deployment Modes

#### nats-resolver setup and push example - Operator Environment/Environment

#### with push permissions - All Deployment Modes

#### Setup User

#### Create/Edit Account - All Environments - All Deployment modes

#### Automated sign up services - JWT and NKEY libraries

#### Simple user creation

#### Create user NKEY

#### Create user JWT

#### Distributed User Creation

#### User creation using NATS

#### Straight forward Setup

#### Account-based Setup

#### Stamping JWT in languages other than Go

#### System Account

#### Event Subjects

#### Service Subjects


#### To exercise listed examples please have the following installed:

#### Subjects always available

#### Subjects available when using NATS-based resolver

#### Old Subjects

#### Leaf Node Connections - Outgoing

#### Non-Operator Mode

#### Operator Mode

#### Connecting Accounts

#### Exports

#### Imports

#### Import Subjects

#### Import Remapping

#### Visualizing Export/Import Relationships

#### Managing Keys

#### Protect Identity NKEYs

#### Reissue Identity NKEYs

#### Operator

#### Account

#### Revocations

#### User

#### Activations

#### Accounts

#### Signing keys

#### nats-server: https://github.com/nats-io/nats-server

#### nats (cli): https://github.com/nats-io/natscli

#### nk (cli & library): https://github.com/nats-io/nkeys

#### nsc (cli): https://github.com/nats-io/nsc

#### jwt (library): https://github.com/nats-io/jwt


#### To install nats-server, nats, nk, nsc:

#### To practice the examples below:

#### Save the conguration below in a le named say server.conf, and start nats server

#### via:

#### So that the server started. Later when we change the conguration we can do a

#### reload like this:

#### Accounts are the NATS isolation context.

#### Messages published in one account won't be received in another.

#### Listen for any message on account a :

```
export GO111MODULE=on
go install github.com/nats-io/nats-server/v2@latest
go install github.com/nats-io/natscli/nats@latest
go install github.com/nats-io/nkeys/nk@latest
go install github.com/nats-io/nsc/v2@latest
```
```
nats-server -c server.conf
```
```
nats-server --signal reload
```
```
accounts: {
A: {
users: [{user: a, password: a}]
},
B: {
users: [{user: b, password: b}]
},
}
```
```
nats -s nats://a:a@localhost:4222 sub ">"
```
## Concepts

### What are Accounts?


#### Publish a message from account b :

#### Note that you do not see this message received by your subscriber.

#### Now publish a messages from account a :

#### This time the message is received by the subscriber:

#### The above example shows no message ow between user a associated with account

#### A and user b in account B. Messages are delivered only within the same account.

#### That is, unless you explicitly dene it.

#### Below is a similar example, this time with messages crossing explicit account

#### boundaries.

#### Modify server.conf and run nats-server --signal reload

#### Subscribe to everything as user 'a'

```
nats -s nats://b:b@localhost:4222 pub "foo" "user b"
```
```
nats -s nats://a:a@localhost:4222 pub "foo" "user a"
```
```
17:57:06 [#1] Received on "foo"
user a
```
```
accounts: {
A: {
users: [{user: a, password: a}]
imports: [{stream: {account: B, subject: "foo"}}]
},
B: {
users: [{user: b, password: b}]
exports: [{stream: "foo"}]
},
}
```

#### Publish on 'foo' as user 'b':

#### This time the message is received by the subscriber:

#### Accounts are a lot more powerful than what has been demonstrated here. Take a look at

#### the complete documentation of accounts and the users associated with them. All of this

#### is in a plain NATS cong le. (Copy the above cong and try it using this command:

#### nats-server -c <filename>) In order to make any changes, every participating nats-

#### server cong le in the same security domain has to change. This conguration is

#### typically controlled by one organization or the administrator.

#### NKEYs are decorated, Base32 encoded, CRC16 check-summed, Ed25519 keys.

#### Ed25519 is:

```
nats -s nats://a:a@localhost:4222 sub ">"
```
```
nats -s nats://b:b@localhost:4222 pub "foo" "user b"
```
```
18:28:25 [#1] Received on "foo"
user b
```
#### Accounts are isolated from each other.

#### One can selectively combine accounts,

#### Need to modify a cong le to add/remove/modify accounts and users,

#### The cong le can be applied to take effect via nats-server --signal reload.

#### a public key signature system. (can sign and verify signatures)

#### resistant to side channel attacks (no conditional jumps in algorithm)

#### Key Takeaways

### What are NKEYs?


#### NATS server can be congured with public NKEYs as user (identities). When a client

#### connects the nats-server sends a challenge for the client to sign in order to prove it is in

#### possession of the corresponding private key. The nats-server then veries the signed

#### challenge. Unlike with a password based scheme, the secret never left the client.

#### To assist with knowing what type of key one is looking at, in cong or logs, the keys are

#### decorated as follows:

#### NKEYs are generated as follows:

#### To view the key:

#### Create another key:

#### View the key:

#### Replacing the user/password with NKEY in account cong example:

#### Public Keys, have a one byte prex: O , A , U for various types, O for operator,

#### A for account, and U meaning user.

#### Private Keys, have a two byte prex SO, SA, SU. S stands for seed. The

#### remainders( O , A and U ) are the same meaning as in public keys.

```
nk -gen user -pubout > a.nk
```
```
cat a.nk
```
###### SUAAEZYNLTEA2MDTG7L5X7QODZXYHPOI2LT2KH5I4GD6YVP24SE766EGPA

###### UC435ZYS52HF72E2VMQF4GO6CUJOCHDUUPEBU7XDXW5AQLIC6JZ46PO5

```
nk -gen user -pubout > b.nk
```
```
cat b.nk
```
###### SUANS4XLL5NWBTM57GSVHLN4TMFW55WGGWNI5YXXSIOYFJQYFVNHJK5GFY

###### UARZVI6JAV7YMJTPRANXANOOW4K3ZCD45NYP6S7C7XKCBHPVN2TFZ7ZC


#### Simple example:

#### Subscribe with nats -s nats://localhost:4222 sub --nkey=a.nk ">"

#### Publish a message using

#### nats -s nats://localhost:4222 pub --nkey=b.nk foo nkey the subscriber should

#### receive it.

#### When the nats-server was started with -V tracing, you can see the signature in the

#### CONNECT message (formatting added manually):

#### On connect, clients are instantly sent the nonce to sign as part of the INFO message

#### (formatting added manually). Since telnet will not authenticate, the server closes the

#### connection after hitting the authorization timeout.

```
accounts: {
A: {
users: [{nkey:UC435ZYS52HF72E2VMQF4GO6CUJOCHDUUPEBU7XDXW5AQLIC6JZ4
imports: [{stream: {account: B, subject: "foo"}}]
},
B: {
users: [{nkey:UARZVI6JAV7YMJTPRANXANOOW4K3ZCD45NYP6S7C7XKCBHPVN2T
exports: [{stream: "foo"}]
},
}
```
```
[95184] 2020/10/26 12:15:44.350577 [TRC] [::1]:55551 - cid:2 - <<- [CONNE
"echo": true,
"headers": true,
"lang": "go",
"name": "NATS CLI",
"nkey": "UC435ZYS52HF72E2VMQF4GO6CUJOCHDUUPEBU7XDXW5AQLIC6JZ46PO5",
"no_responders": true,
"pedantic": false,
"protocol": 1,
"sig": "lopzgs98JBQYyRdw1zT_BoBpSFRDCfTvT4le5MYSKrt0IqGWZ2OXhPW1J_zo2
"tls_required": false,
"verbose": false,
"version": "1.11.0"
}]
```

#### In a large organization the centralized conguration approach can lead to less exibility

#### and more resistance to change when controlled by one entity. Alternatively, instead of

#### operating one infrastructure, it can be deployed more often (say per team) thus making

#### import/export relationships harder as they have to bridge separate systems. In order to

#### make accounts truly powerful, they should ideally be congured separately from the

#### infrastructure, only constrained by limits. This is similar for user. An account contains the

```
> telnet localhost 4222
Trying ::1...
Connected to localhost.
Escape character is '^]'.
INFO {
"auth_required": true,
"client_id": 3,
"client_ip": "::1",
"go": "go1.14.1",
"headers": true,
"host": "0.0.0.0",
"max_payload": 1048576,
"nonce": "-QPTE1Jsk8kI3rE",
"port": 4222,
"proto": 1,
"server_id": "NBSHIXACRHUODC4FY2Z3OYXSZSRUBRH6VWIKQNGVPKOTA7H4YTXWJRT
"server_name": "NBSHIXACRHUODC4FY2Z3OYXSZSRUBRH6VWIKQNGVPKOTA7H4YTXWJ
"version": "2.2.0-beta.26"
}
-ERR 'Authentication Timeout'
Connection closed by foreign host.
```
#### NKEYS are a secure way to authenticate clients,

#### Private keys are never accessed or stored by the NATS server,

#### The public key still needs to be congured in NATS server.

#### Key Takeaways

## JSON Web Tokens (JWT)

### Motivation for JWT


#### user but this relationship could be a reference as well, such that alterations to user do not

#### alter the account. Users of the same account should be able to connect from anywhere in

#### the same infrastructure and be able to exchange messages as long as they are in the

#### same authentication domain.

#### Account and User creation managed as separate artifacts in a decentralized fashion

#### using NKEYs. Relying upon a hierarchical chain of trust between three distinct NKEYs and

#### associated roles:

#### 1. Operator: corresponds to operator of a set of NATS servers in the same authentication

#### domain (entire topology, crossing gateways and leaf nodes),

#### 2. Account: corresponds to the set of a single account's conguration,

#### 3. User: corresponds to one user's conguration.

#### Each NKEY is referenced, together with additional conguration, in a JWT document.

#### Each JWT has a subject eld and its value is the public portion of an NKEY and serves as

#### identity. Names exist in JWT but as of now are only used by tooling, nats-server does

#### not read this value. The referenced NKEY's role determines the JWT content:

#### 1. Operator JWTs contain server conguration applicable throughout all operated NATS

#### servers,

#### JWT splits a nats-server conguration into separate artifacts manageable by different

#### entities.

#### Management of Accounts, Conguration, and Users are separated.

#### Accounts do NOT correspond to infrastructure, they correspond to teams or

#### applications.

#### Connect to any cluster in the same infrastructure and be able to communicate with all

#### other users in your account.

#### Infrastructure and its topology have nothing to do with Accounts and where an

#### Account's User connects from.

#### Key Takeaways

### Decentralized Authentication/Authorization using JWT


#### 2. Account JWTs contain Account specic conguration such as exports, imports, limits,

#### and default user permissions,

#### 3. User JWTs contain user specic conguration such as permissions and limits.

#### In addition, JWTs can contain settings related to their decentralized nature, such as

#### expiration/revocation/signing. At no point do JWTs contain the private portion of an

#### NKEY, only signatures that can be veried with public NKEY. JWT content can be viewed

#### as public, although it's content may reveal which subjects/limits/permissions exist.

#### A nats-server is congured to trust an operator. Meaning, the Operator JWT is part of

#### its server conguration and requires a restart or nats-server --signal reload once

#### the conguration changed. It is also congured with a way to obtain account JWT in one

#### of three ways (explained below).

#### Clients provide a User JWT when connecting. An Account JWT is not used by clients

#### talking to a nats-server. The clients also possess the private NKEY corresponding to

#### the JWT identity, so that they can prove their identity as described above.

#### The issuer eld of the User JWT identies the Account, and the nats-server then

#### independently obtains the current Account JWT from its congured source. The server

#### can then verify that signature on the User JWT was issued by an NKEY of the claimed

#### Account, and in turn that the Account has an issuer of the Operator and that an NKEY of

#### the Operator signed the Account JWT. The entire three-level hierarchy is veried.

#### JWTs are hierarchically organized in operator, account and user,

#### They carry corresponding conguration and the conguration has been dedicated to

#### the decentralized nature of NATS JWT usage.

#### Key Takeaways

### NATS JWT Hierarchy

#### Decentralized Chain of Trust

#### Obtain an Account JWT


#### To obtain an Account JWT, the nats-server is congured with one of three resolver types.

#### Which one to pick depends upon your needs:

#### mem-resolver: Very few or very static accounts

#### You are comfortable changing the server cong if the operator or any accounts

#### changed,

#### You can generate a user programmatically using NKEYs and a JWT library (more

#### about that later),

#### Users do not need to be known by nats-server.

#### url-resolver: Very large volume of accounts

#### Same as mem-resolver, except you do not need to modify the server

#### congurations when accounts are added or changed,

#### Changes to the operator still require a server reloading (only a few operations

#### require that),

#### Will download Accounts from a web server

#### Allows for easy publication of account JWTs programmatically generated using

#### NKEYs and the JWT library.

#### The nats-account-server is such a webserver. When set up correctly, it will

#### inform nats-server of Account JWT changes.

#### Depending on conguration, requires read and/or write access to persistent

#### storage.

#### nats-resolver: Same as url-resolver, just uses NATS instead of http

#### No separate binary to run/cong/monitor,

#### Easier clustering when compared to nats-account-server. Will eventually

#### converge on the union of all account JWTs known to every participating

#### nats-server,

#### Requires persistent storage in the form of a directory for nats-server to

#### exclusively write to (it can be on a shared Network File System, but the directories

#### themselves can not be shared between servers),

#### Optionally, directly supports Account JWT removal,

#### Between nats-resolver and url-resolver, the nats-resolver is the clear

#### recommendation.


#### JWT nats-resolver is recommended to use in production environment. With JWT

#### nats-resolver, you can manage huge accounts and users without server reloading.

#### But before adopting JWT nat-resolver, make sure you understand correctly how it

#### works. You can make use of static account settings(probably with NKEYs) and

#### memory-resolver as the necessary steps forwarding JWT nats-resolver fully

#### understanding.

#### Each JWT document has a subject(sub) it represents. This is the public identity NKEY

#### represented by the JWT document. JWT documents contain an issued at (iat) time of

#### signing. This time is in seconds since Unix epoch. It is also used to determine which of

#### two JWTs for the same subject is more recent. Furthermore JWT documents have an

#### issuer, this may be an (identity) NKEY or a dedicated signing NKEY of an item one level

#### above it in the trust hierarchy. A key is a signing key if it is listed as such in the JWT

#### (above). Signing NKEYs adhere to same NKEY roles and are additional keys that unlike

#### identity NKEY may change over time. In the hierarchy, signing keys can only be used to

#### sign JWT for the role right below them. User JWTs have no signing keys for this reason.

#### To modify one role's set of signing keys, the identity NKEY needs to be used.

#### Each JWT is signed as below:

#### If a JWT is valid, the JWT above it is validated as well. If all of them are valid, the chain of

#### trust between them is tested top down as follows:

```
jwt.sig = sign(hash(jwt.header + jwt.body), private-key(jwt.issuer))
(jwt.issuer is part of jwt.body)
```
```
Type Trust Rule Obtained
```
```
Operator jwt.issuer == jwt.subject (self signed)
```
```
congured
to trust
```
```
Account jwt.issuer == trusted issuing^
operator (signing/identity) key
```
```
congured
to obtain
```
#### JWT and Chain of Trust Verification


#### This is a conceptual view. While all these checks happen, the results of earlier evaluations

#### might be cached: if the Operator/Account is trusted already and the JWT did not change

#### since, then there is no reason to re-evaluate.

#### Below are examples of decoded JWT (iss == issuer, sub == subject, iat ==

#### issuedAt):

#### nsc describe account -n demo-test --json:

```
Type Trust Rule Obtained
```
```
User
```
```
jwt.issuer == trusted issuing account
(signing/identity) key && jwt.issuedAt >
issuing account revocations[jwt subject]
```
```
provided
on connect
```
```
nsc describe operator --json
```
###### {

```
"iat": 1603473819,
"iss": "OBU5O5FJ324UDPRBIVRGF7CNEOHGLPS7EYPBTVQZKSBHIIZIB6HD66JF",
"jti": "57BWRLW67I6JTVYMQAZQF54G2G37DJB5WG5IFIPVYI4PEYNX57ZQ",
"name": "DEMO",
"nats": {
"account_server_url": "nats://localhost:4222",
"system_account": "AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLK
},
"sub": "OBU5O5FJ324UDPRBIVRGF7CNEOHGLPS7EYPBTVQZKSBHIIZIB6HD66JF",
"type": "operator"
}
```

#### nsc describe user -a demo-test -n alpha --json:

#### When a client connects, the steps below have to succeed. The following nats-server

#### conguration is used (for ease of understanding, we are using url-resolver):

###### {

```
"iat": 1603474600,
"iss": "OBU5O5FJ324UDPRBIVRGF7CNEOHGLPS7EYPBTVQZKSBHIIZIB6HD66JF",
"jti": "CZDE4PM7MGFNYHRZSE6INTP6QDU4DSLACVHPQFA7XEYNJT6R6LLQ",
"name": "demo-test",
"nats": {
"limits": {
"conn": -1,
"data": -1,
"exports": -1,
"imports": -1,
"leaf": -1,
"payload": -1,
"subs": -1,
"wildcards": true
}
},
"sub": "ADKGAJU55CHYOIF5H432K2Z2ME3NPSJ5S3VY5Q42Q3OTYOCYRRG7WOWV",
"type": "account"
}
```
###### {

```
"iat": 1603475001,
"iss": "ADKGAJU55CHYOIF5H432K2Z2ME3NPSJ5S3VY5Q42Q3OTYOCYRRG7WOWV",
"jti": "GOOPXCFDWVMEU3U6I6MT344Z56MGBYIS42GDXMUXDFA3NYDR2RUQ",
"name": "alpha",
"nats": {
"pub": {},
"sub": {}
},
"sub": "UC56LV5NNMP5FURQZ7HZTGWCRRTWSMHZNNELQMHDLH3DCYNGX57B2TN6",
"type": "user"
}
>
```
#### Obtain a User JWT - Client Connect


#### 1. Client connects and the nats-server responds with INFO (identical to NKEYs) and

#### a containing nonce.

#### For ease of use, the NATS CLI uses a creds le that is the concatenation of JWT and

#### private user identity/NKEY.

```
operator: ./trustedOperator.jwt
resolver: URL(http://localhost:9090/jwt/v1/accouts/)
```
```
> telnet localhost 4222
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
INFO {
"auth_required": true,
"client_id": 5,
"client_ip": "127.0.0.1",
"go": "go1.14.1",
"headers": true,
"host": "localhost",
"max_payload": 1048576,
"nonce": "aN9-ZtS7taDoAZk",
"port": 4222,
"proto": 1,
"server_id": "NCIK6FX5MRIEPMEK22YL2ECLIWVJBH2SWFD5EQWSI5XRDQPKZXWK
"server_name": "NCIK6FX5MRIEPMEK22YL2ECLIWVJBH2SWFD5EQWSI5XRDQPKZX
"tls_required": true,
"version": "2.2.0-beta.26"
}
Connection closed by foreign host.
```
```
> cat user.creds
-----BEGIN NATS USER JWT-----
eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJXNkFYSFlSS1RHVTNFUk
------END NATS USER JWT------
```
```
************************* IMPORTANT *************************
NKEY Seed printed below can be used to sign and prove identity.
NKEYs are sensitive and should be treated as secrets.
```
```
-----BEGIN USER NKEY SEED-----
SUAAZU5G7UOUR7VXQ7DBD5RQTBW54O2COGSXAVIYWVZE4GCZ5C7OCZ5JLY
------END USER NKEY SEED------
```
```
*************************************************************
```

#### 2. The Client responds with a CONNECT message (formatting added manually),

#### containing a JWT and signed nonce. (output copied from nats-server started with

#### -V)

#### 3. Server veries if a JWT returned is a user JWT and if it is consistent:

#### sign(jwt.sig, jwt.issuer) == hash(jwt.header+jwt.body) (issuer is part of

#### body),

#### 4. Server veries if nonce matches JWT.subject, thus proving client's possession of

#### private user NKEY,

#### 5. Server either knows referenced account or downloads it from

```
http://localhost:9090/jwt/v1/accouts/AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNX
ZM6ARFGXP7BASNDGLKU7A5
```
#### ,

#### . Server veries downloaded JWT is an account JWT and if it is consistent:

#### sign(jwt.sig, jwt.issuer) == hash(jwt.header+jwt.body) (issuer is part of

#### body),

#### 7. Server veries if an account JWT issuer is in congured list of trusted operator keys

#### (derived from operator JWT in conguration),

#### . Server veries that a user JWT subject is not in the account's revoked list, or if

#### jwt.issuedAt eld has a higher value,

#### 9. Server veries that a user JWT issuer is either identical to the account JWT subject or

#### part of the account JWT signing keys,

#### 10. If all of the above holds true, the above invocation will succeed, only if the user JWT

#### does not contain permissions or limits restricting the operation otherwise

```
> nats -s localhost:4222 "--creds=user.creds" pub "foo" "hello world"
```
```
[98019] 2020/10/26 16:07:53.861612 [TRC] 127.0.0.1:56830 - cid:4 - <<-
"echo": true,
"headers": true,
"jwt": "eyJ0eXAiOiJqd3QiLCJhbGciOiJlZDI1NTE5In0.eyJqdGkiOiJXNkFYSF
"lang": "go",
"name": "NATS CLI",
"no_responders": true,
"pedantic": false,
"protocol": 1,
"sig": "VirwM--xq5i2RI9VEQiFYv_6JBs-IR4oObypglR7qVxYtXDUtIKIr1qXW_
"tls_required": true,
"verbose": false,
"version": "1.11.0"
}]
```

#### 11. Output if user.creds were to contain a JWT where the maximum message payload

#### is limited to 5 bytes

#### Depending on which entity has access to private Operator/Account identity or signing

#### NKEYs, different deployment models are enabled. When picking one, it is important to

#### pick the simplest deployment model that enables what you need it to do. Everything

#### beyond just results in unnecessary conguration and steps.

#### 1. Centralized cong: one (set of) user(s) has access to all private operator and account

#### NKEYs,

#### Administrators operating the shared infrastructure call all the shots

#### 2. Decentralized cong (with multiple nsc environments, explained later):

#### 1. Administrator/Operator(s) have access to private operator NKEYs to sign accounts.

#### By signing or not signing an account JWT, Administrators can enforce constraints

#### (such as limits).

```
> nats -s localhost:4222 "--creds=user.creds" pub "foo" "hello world"
> 16:56:02 Published 11 bytes to "foo"
```
```
> nats -s localhost:4222 "--creds=user.creds" pub "foo" "hello world"
nats: error: nats: Maximum Payload Violation, try --help
>
```
#### JWTs are secure,

#### JWTs carry conguration appropriate to their role as Operator/Accounts/User,

#### JWTs provide a basis for operating one single NATS infrastructure which serves

#### separate, yet optionally connected, entities,

#### Account resolvers are a way to obtain unknown Account JWTs,

#### On connect clients provide only the User JWT and use the NKEY for the JWT to

#### authenticate,

#### JWTs can be issued programmatically.

#### Key Takeaways

### Deployment Models Enabled by Chain of Trust


#### 2. Other sets of users (teams) have access to their respective private account

#### This can also be used bidentity/signing NKEYy a single entity ts and can issue/sign a user JWo not mix up nsc envirT. onments as well.

#### 3. Self-service, decentralized cong (shared dev cluster):

#### Is similar to 2, but sets of users in 2.1 have access to an operator private signing

#### NKEY.

#### This allows teams to add/modify their own accounts.

#### Since administrators give up control over limits, there should be at least one

#### organizational mechanism to prevent unchecked usage.

#### Administrators operating the infrastructure can add/revoke access by controlling the

#### set of operator signing keys.

#### 4. Mix of the above - as needed: separate sets of users (with multiple nsc

#### environments).

#### For some user/teams the Administrator operates everything.

#### Signing keys can not only be used by individuals in one or more nsc environments, but

#### also by programs facilitating JWT and NKEY libraries. This allows the implementation of

#### sign-up services.

#### Account signing key enabled on the y:

#### user generation (explained later),

#### export activation generation (explained later).

#### Operator signing key enables on the y account generation.

#### JWTs and the associated chain of trust allows for centralized, decentralized, or self-

#### service account conguration,

#### It is important to pick the deployment model that ts your needs, NOT the most

#### complicated one,

#### Distributing Operator/Account JWT NKEYs between Administrators and teams

#### enables these deployment models,

#### Sign-up services for Accounts/Users can be implemented by programs in possession

#### of the parent type's signing keys,

#### Key Takeaways


#### A deeper understanding of accounts will help you to best setup NATS JWT based

#### security.

#### What entity do accounts correspond to:

#### Our ocial suggestion is to scope accounts by application/service offered.

#### This is very ne gr ained and will require some conguration.

#### This is why some users gravitate to accounts per team. One account for all

#### Applications of a team.

#### It is possible to start out with less granular accounts and as applications grow in

#### importance or scale become more ne grained.

#### Compared to le based cong, Imports and Exports change slightly.

#### To control who gets to import an export, activation tokens are introduced.

#### These are JWTs that an importer can embed.

#### They comply to similar verication rules as user JWT, thus enabling a nats-server

#### to check if the exporting account gave explicit consent.

#### Due to the use of a token, the exporting account's JWT does not have to be modied

#### for each importing account.

#### Updates of JWTs are applied as nats-server discover them

#### How this is done depends on the resolver

#### mem-resolver require nats-server --signal reload to re-read all

#### congured account JWTs,

#### url-resolver and nats-resolver listen on a dedicated update subject of

#### the system account and applied if the le is valid,

#### nats-resolver will also also update the corresponding JWT le and

#### compensate in case the update message was not received due to temporary

#### disconnect.

#### User JWTs only depend on the issuing Account NKEY, they do NOT depend on a

#### particular version of an Account JWT,

#### Depending on the change, the internal Account representation will be updated and

#### existing connections re-evaluated.

## Accounts Re-visited


#### This section will introduce nsc cli to generate and manage operator/accounts/user.

#### Even if you intend to primarily generate your Accounts/User programmatically, in all

#### likelihood, you won't do so for an operator or all accounts. Key Management and how to

#### do so using nsc will also be part of this section.

#### nsc is a tool that uses the JWT and NKEY libraries to create NKEYs (if asked to) and all

#### types of JWT. It then stores these artifacts in separate directories.

#### It keeps track of the last operator/account used. Because of this, commands do not need

#### to reference operator/accounts but can be instructed to do so (recommended for

#### scripts). It supports an interactive mode when -i is provided. When used, referencing

#### accounts/keys is easier.

#### nsc env will show where NKEYS/JWT are stored and what current defaults are. For

#### testing you may want to switch between nsc environments: Changing the (JWT) store

#### directory: nsc env --store <different folder> Changing the (NKEY) store directory

#### by having an environment variable set: export NKEYS_PATH=<different folder>

#### The System Account is the account under which nats-server offers (administrative)

#### services and monitoring events.

#### Accounts can be arbitrarily scoped, from Application to Team,

#### Account Exports can be restricted by requiring use of activation tokens,

#### Receiving a more recent Account JWT causes the nats-server to apply changes and re

#### evaluate existing connections.

### Key Takeaways

## Tooling And Key Management

### nsc

#### Environment


#### Subsequent sections will refer to different environments in context of different

#### deployment modes. As such you can skip over all mentions for modes not of interest to

#### you. The mixed deployment mode is not mentioned and left as an exercise to the reader.

#### Possessing NKEYS gives access to the system. Backups should therefore best be oine

#### and access to them should be severely restricted. In cases where regenerating all/parts

#### of the operator/accounts is not an option, signing NKEYs must be used and identity

#### NKEYs should be archived and then removed from the original store directory, so that in

#### the event of a data breach you can recover without a ag-day change-over of identities.

#### Thus, depending on your scenario, relevant identity NKEYS need to only exist in very

#### secure oine backup(s).

#### The store directory contains JWTs for operators, accounts, and users. It does not contain

#### private keys. Therefore it is ok to back these up or even store them in a VCS such as git.

#### But be aware that depending on content, JWT may reveal which

#### permissions/subjects/public-nkeys exist. Knowing the content of a JWT does not grant

#### access; only private keys will. However, organizations may not wish to make those public

#### outright and thus have to make sure that these external systems are secured

#### appropriately.

#### When restoring an older version, be aware that:

#### All changes made since will be lost, specically revocations may be undone,

#### Time has moved on and thus JWTs that were once valid at the time of the backup or

#### commit may be expired now. Thus you may have to be edit them to match your

#### expectations again,

#### NKEYS are stored in a separate directory, so not to restore a JWT for which the NKEY

#### has been deleted since:

#### Either keep all keys around; or

### Backup

#### NKEYS store directory

#### JWT store directory


#### JWTs allow you to specify names. But names do NOT represent an identity, they are only

#### used to ease referencing of identities in our tooling. At no point are these names used to

#### reference each other, instead, the public identity NKEY is used for that. The

#### nats-server does not read them at all. Because names do not relate to identity, they

#### may collide. Therefore, when using nsc, these names need to be keep unique.

#### Create operator with system account and system account user:

#### The command nsc edit operator [flags] can subsequently be used to modify the

#### operator. For example if you are setting the account server url (used by url-resolver

#### and nats-resolver), nsc does not require them being specied on subsequent

#### commands.

```
nsc edit operator --account-jwt-server-url "nats://localhost:4222"
```
#### Note that if you update an operator JWT that is installed on a server you will need to

#### manually update the operator JWT and reload the server While nsc is able to update

#### accounts, it never updates the operator.

#### We always recommend using signing keys for an operator. Generate one for an operator (

#### -o) and store it in the key directory (--store). The output will display the public

#### portion of the signing key, use that to assign it to the operator (--sk O...).

#### nsc generate nkey -o --store followed by

```
nsc edit operator --sk
OB742OV63OE2U55Z7UZHUB2DUVGQHRA5QVR4RZU6NXNOKBKJGKF6WRTZ
```
#### Restore the NKEY directory in tandem.

```
nsc add operator -n <operator-name> --sys
```
#### Names in JWT

### Setup an Operator

#### Create/Edit Operator - Operator Environment - All Deployment modes


#### . To pick the operator signing key for account generation, provide the -i option when

#### doing so.

#### The system account is the account under which nats-server offers system services as

#### will be explained below in the system-account section. To access these services a user

#### with credentials for the system account is needed. Unless this user is restricted with

#### appropriate permissions, this user is essentially the admin user. They are created like any

#### other user.

#### For cases where signing keys are generated and immediately added --sk generate will

#### create an NKEY on the y and assign it as signing NKEY.

#### In order to import an Operator JWT, say the one just created, into a separate nsc

#### environment maintained by a different entity/team, the following has to happen:

#### 1. Obtain the operator JWT using: nsc describe operator --raw and store the output

#### in a le named operator.jwt. The option --raw causes the raw JWT to be emitted,

#### 2. Exchange that le or it's content in any way you like, email works ne (as there are no

#### credentials in the JWT),

#### 3. Import the operator JWT into the second environment with:

#### nsc add operator -u operator.jwt.

#### If the operator should been changed and an update is required, simply repeat these steps

#### but provide the --force option in the last step. This will overwrite the stored operator

#### JWT.

#### In addition to the previous step, self service deployments require an operator signing key

#### and a system account user. Ideally you would want an operator signing key per entity to

#### distribute a signing key too. Simply repeat the command shown earlier but:

#### 1. Perform nsc generate nkey -o --store in this environment instead,

#### Import Operator - Non Operator/Administrator Environment - Decentralized/Self Service

#### Deployment Modes

#### Import Operator - Self Service Deployment Modes


#### 2. Exchange the public key with the Administrator/Operator via a way that assures you

#### sent the public key and not someone elses,

#### 3. Perform nsc edit operator --sk in the operator environment,

#### 4. Refresh the operator JWT in this environment by performing the import steps using

```
--force
```
#### To import the system account user needed for administrative purposes as well as

#### monitoring, perform these steps:

#### 1. Perform nsc describe account -n SYS --raw and store the output in a le named

#### SYS.jwt.

#### The option -n species the (system) account named SYS.

#### 2. Exchange the le,

#### 3. Import the account nsc import account --file SYS.jwt,

#### 4. Perform nsc generate nkey -u --store in this environment,

#### 5. Exchange the public key printed by the command with the Administrator/Operator via

#### a way that assures you sent the public key and not someone elses,

#### . Create a system account user named (-n) any way you like (here named

#### sys-non-op) providing (-k) the exchanged public key

```
nsc add user -a SYS -n sys-non-op -k
UDJKPL7H6QY4KP4LISNHENU6Z434G6RLDEXL2C64YZXDABNCEOAZ4YY2
```
#### in the operator environment. (-a references the Account SYS.),

#### 7. If desired edit the user,

#### . Export the user nsc describe user -a SYS -n sys-non-op --raw from the

#### operator environment and store it in a le named sys.jwt. (-n references the user

#### sys-non-op),

#### 9. Exchange the le,

#### 10. Import the user in this environment using nsc import user --file sys.jwt

#### As a result of these operations, your operator environment should have these keys and

#### signing keys:

```
nsc list keys --all
```

#### And your account should have the following ones:

#### Between the two outputs, compare the Stored column.

#### Alternatively if the administrator is willing to exchange private keys and the exchange can

#### be done securely, a few of these steps fall away. The signing key and system account

#### user can be generated in the administrator/operator environment, omitting --store to

#### avoid unnecessary key copies. Then the public/private signing NKEYS are exchanged

#### together with the system account user as creds le. A creds le can be generated with

#### nsc generate creds -a SYS -n sys-non-op and imported into this environment with

#### nsc import user --file sys.jwt. If the signing key is generated before the operator

#### is imported into this environment, operator update falls away.

###### +------------------------------------------------------------------------

```
| Keys
+--------------+---------------------------------------------------------
| Entity | Key
+--------------+---------------------------------------------------------
| DEMO | OD5FHU4LXGDSGDHO7UNRMLW6I36QX5VPJXRQHFHMRUIKSHOPEDSHVPBB
| DEMO | OBYAIG4T4PVR6GVYDERN74RRW7VBKRWBTI7ULLMM6BRHUID4AAQL7SGA
| ACC | ADRB4JJYFDLWKIMX4DH6MX2DMKA3TENJWGMNVM5ILYLZTT6BN7QIF5ZX
| SYS | AAYVLZJC2ULKSH5HNSKMIKFMCEHCNU5VOV5KG56IRL7ENHLBUGZ27CZT
| sys | UBVZYLLCAFMHBXBUDKKKFKH62T4AW7Q5MAAE3R3KKAIRCZNYITZPDQZ3
| sys-non-op | UDJKPL7H6QY4KP4LISNHENU6Z434G6RLDEXL2C64YZXDABNCEOAZ4YY2
+--------------+---------------------------------------------------------
```
```
nsc list keys --all
```
###### +------------------------------------------------------------------------

```
| Keys
+--------------+---------------------------------------------------------
| Entity | Key
+--------------+---------------------------------------------------------
| DEMO | OD5FHU4LXGDSGDHO7UNRMLW6I36QX5VPJXRQHFHMRUIKSHOPEDSHVPBB
| DEMO | OBYAIG4T4PVR6GVYDERN74RRW7VBKRWBTI7ULLMM6BRHUID4AAQL7SGA
| SYS | AAYVLZJC2ULKSH5HNSKMIKFMCEHCNU5VOV5KG56IRL7ENHLBUGZ27CZT
| sys-non-op | UDJKPL7H6QY4KP4LISNHENU6Z434G6RLDEXL2C64YZXDABNCEOAZ4YY2
+--------------+---------------------------------------------------------
```
### Setup an Account


#### Create an account as follows:

#### In case you have multiple operator signing keys -i will prompt you to select one.

#### nsc edit account [flags] can subsequently be used to modify the account. (Edit is

#### also applicable to the system account)

#### Similar to the operator signing keys are recommended. Generate signing key for an

#### account (-a) and store it in the key directory maintained by nsc (--store) The output

#### will display the public portion of the signing key, use that to assign it to the account (

#### --sk A...) nsc generate nkey -a --store

```
nsc edit account --sk
ACW2QC262CIQUX4ACGOOS5XLKSZ2BY2QFBAAOF3VOP7AWAVI37E2OQZX
```
#### To pick the signing key for user generation, provide the -i option when doing so.

#### In this mode, the created account is self-signed. To have it signed by the operator

#### perform these steps:

#### 1. In this environment export the created account as a JWT like this

#### nsc describe account -n <account name> --raw.

#### Store the output in a le named import.jwt.

#### 2. Exchange the le with the Administrator/Operator via a way that assures it is your JWT

#### and not someone elses.

#### 3. In the operator environment import the account with

#### nsc import account --file import.jwt.

#### This step also re-signs the JWT so that it is no longer self-signed.

#### 4. The Administrator/operator can now modify the account with

```
nsc edit account [flags]
```
```
nsc add account -n <account name> -i
```
#### Create/Edit Account - All Environments - All Deployment modes

#### Export Account - Non Operator/Administrator Environment - Decentralized Deployment

#### Modes


#### If the account should be changed and an update is required, simply repeat these steps

#### but provide the --force option during the last step. This will overwrite the stored

#### account JWT.

#### This environment is set up with a signing key, thus the account is already created properly

#### signed. The only step that is needed is to push the Account into the NATS network.

#### However, this depends on your ability to do so. If you have no permissions, you have to

#### perform the same steps as for the decentralized deployment mode. The main difference

#### is that upon import, the account won't be re-signed.

#### How accounts can be publicized wholly depends on the resolver you are using:

#### nsc generate config <resolver-type> is an utility that generates the relevant NATS

#### cong. Where <resolver-type> can be --mem-resolver or --nats-resolver for

#### the corresponding resolver. Typically the generated output is stored in a le that is then

#### included by the NATS cong. Every server within the same authentication domain needs

#### to be congured with this conguration.

#### mem-resolver: The operator has to have all accounts imported and generate a new

#### cong,

#### url-resolver: nsc push will send an HTTP POST request to the hosting webserver or

#### nats-account-server,

#### nats-resolver: Every environment with a system account user that has permissions

#### to send properly signed account JWT as requests to:

#### $SYS.REQ.CLAIMS.UPDATE can upload and update all accounts. Currently,

#### nsc push uses this subject.

#### $SYS.REQ.ACCOUNT.*.CLAIMS.UPDATE can upload and update specic accounts.

#### Export Account - Non Operator/Administrator Environment - Self Service Deployment

#### Modes

#### Publicize an Account with Push - Operator Environment/Environment with push

#### permissions - All Deployment Modes

#### nats-resolver setup and push example - Operator Environment/Environment with push

#### permissions - All Deployment Modes


#### This is a quick demo of the nats-based resolver from operator creation to publishing a

#### message. Please be aware that the ability to push only relates to permissions to do so

#### and does not require an account keys. Thus, how accounts to be pushed into the

#### environment (outright creation/import) does not matter. For simplicity, this example uses

#### the operator environment.

#### Operator Setup:

#### Inspect the setup:

#### nsc describe operator:

```
nsc add operator -n DEMO --sys
```
```
[ OK ] generated and stored operator key "ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJY
[ OK ] added operator "DEMO"
[ OK ] created system_account: name:SYS id:AA6W5MRDIFIQWE6UE6D4YWQT5L4YZG
[ OK ] created system account user: name:sys id:UABM73CE5F3ZYFNC3ZDODAF7G
[ OK ] system account user creds file stored in `~/test/demo/env1/keys/cr
```
```
nsc edit operator --account-jwt-server-url nats://localhost:4222
```
```
[ OK ] set account jwt server url to "nats://localhost:4222"
[ OK ] edited operator "DEMO"
```
```
nsc list keys --all
```
###### +------------------------------------------------------------------------

```
| Keys
+--------+----------------------------------------------------------+----
| Entity | Key | Sig
+--------+----------------------------------------------------------+----
| DEMO | ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV7F7KUWADCM4K6 |
| SYS | AA6W5MRDIFIQWE6UE6D4YWQT5L4YZG7ZRHSKYCPF2VIEMUHRZH3VQZ27 |
| sys | UABM73CE5F3ZYFNC3ZDODAF7GIB62W2WXV5DOLMYLGEW4MEHYBC46PN4 |
+--------+----------------------------------------------------------+----
```

#### nsc describe account:

#### Generate the cong and start the server in the background. Also, inspect the generated

#### cong. It consists of the mandatory operator, explicitly lists the system account and

#### corresponding JWT:

###### +------------------------------------------------------------------------

```
| Operator Details
+--------------------+---------------------------------------------------
| Name | DEMO
| Operator ID | ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV7F7KUWA
| Issuer ID | ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV7F7KUWA
| Issued | 2020-11-04 19:25:25 UTC
| Expires |
| Account JWT Server | nats://localhost:4222
| System Account | AA6W5MRDIFIQWE6UE6D4YWQT5L4YZG7ZRHSKYCPF2VIEMUHRZH
+--------------------+---------------------------------------------------
```
###### +------------------------------------------------------------------------

```
| Account Details
+---------------------------+--------------------------------------------
| Name | SYS
| Account ID | AA6W5MRDIFIQWE6UE6D4YWQT5L4YZG7ZRHSKYCPF2VI
| Issuer ID | ODHUVOUVUA3XIBV25XSQS2NM2UN4IKJYLAMCGLWRFAV
| Issued | 2020-11-04 19:24:41 UTC
| Expires |
+---------------------------+--------------------------------------------
| Max Connections | Unlimited
| Max Leaf Node Connections | Unlimited
| Max Data | Unlimited
| Max Exports | Unlimited
| Max Imports | Unlimited
| Max Msg Payload | Unlimited
| Max Subscriptions | Unlimited
| Exports Allows Wildcards | True
+---------------------------+--------------------------------------------
| Imports | None
| Exports | None
+---------------------------+--------------------------------------------
```
```
nsc generate config --nats-resolver > nats-res.cfg
nats-server -c nats-res.cfg --addr localhost --port 4222 &
```

#### Add an account and a user for testing:

#### Without having pushed the account the user can't be used yet.

#### Doesn't work

###### [2] 30129

```
[30129] 2020/11/04 14:30:14.062132 [INF] Starting nats-server version 2.2
[30129] 2020/11/04 14:30:14.062215 [INF] Git commit [not set]
[30129] 2020/11/04 14:30:14.062219 [INF] Using configuration file: nats-r
[30129] 2020/11/04 14:30:14.062220 [INF] Trusted Operators
[30129] 2020/11/04 14:30:14.062224 [INF] System : ""
[30129] 2020/11/04 14:30:14.062226 [INF] Operator: "DEMO"
[30129] 2020/11/04 14:30:14.062241 [INF] Issued : 2020-11-04 14:25:25
[30129] 2020/11/04 14:30:14.062244 [INF] Expires : 1969-12-31 19:00:00
[30129] 2020/11/04 14:30:14.062652 [INF] Managing all jwt in exclusive di
[30129] 2020/11/04 14:30:14.065888 [INF] Listening for client connections
[30129] 2020/11/04 14:30:14.065896 [INF] Server id is NBQ6AG5YIRC6PRCUPCA
[30129] 2020/11/04 14:30:14.065898 [INF] Server name is NBQ6AG5YIRC6PRCUP
[30129] 2020/11/04 14:30:14.065900 [INF] Server is ready
>
```
```
nsc add account -n TEST
```
```
[ OK ] generated and stored account key "ADXDDDR2QJNNOSZZX44C2HYBPRUIPJSQ
[ OK ] added account "TEST"
```
```
nsc add user -a TEST -n foo
```
```
[ OK ] generated and stored user key "UA62PGBNKKQQWDTILKP5U4LYUYF3B6NQHVP
[ OK ] generated user creds file `/DEMO/TEST/foo.creds`
[ OK ] added user "foo" to account "TEST"
```
```
nats -s nats://localhost:4222 pub --creds=/DEMO/TEST/foo.creds "hello" "w
```

#### Push the account, or push all accounts:

#### For the NATS resolver, each nats-server that responds will be listed. In case you get

#### fewer responses than you have servers or a server reports an error, it is best practice to

#### resolve this issue and retry. The NATS resolver will gossip missing JWTs in an eventually

#### consistent way. Servers without a copy will perform a lookup from servers that do. If

#### during an initial push only one server responds there is a window where this server goes

#### down or worse, loses its disk. During that time the pushed account is not available to the

#### network at large. Because of this, it is important to make sure that initially, more servers

#### respond than what you are comfortable with losing in such a way at once.

#### Once the account is pushed, its user can be used:

```
nats: error: read tcp 127.0.0.1:60061->127.0.0.1:4222: i/o timeout, try -
[9174] 2020/11/05 16:49:34.331078 [WRN] Account [ADI4H2XRYMT5ENVBBS3UKYC2
[9174] 2020/11/05 16:49:34.331123 [WRN] Account fetch failed: fetching jw
[9174] 2020/11/05 16:49:34.331182 [ERR] 127.0.0.1:60061 - cid:5 - "v1.11.
[9174] 2020/11/05 16:49:34.331258 [WRN] 127.0.0.1:60061 - cid:5 - "v1.11.
```
```
nsc push -a TEST
```
```
[ OK ] push to nats-server "nats://localhost:4222" using system account "
[ OK ] push TEST to nats-server with nats account resolver:
[ OK ] pushed "TEST" to nats-server NBQ6AG5YIRC6PRCUPCAUSVC
[ OK ] pushed to a total of 1 nats-server
```
```
nsc push --all
```
```
[ OK ] push to nats-server "nats://localhost:4222" using system account "
[ OK ] push SYS to nats-server with nats account resolver:
[ OK ] pushed "SYS" to nats-server NBENVYIBPNQGYVP32Y3P6WLG
[ OK ] pushed to a total of 1 nats-server
[ OK ] push TEST to nats-server with nats account resolver:
[ OK ] pushed "TEST" to nats-server NBENVYIBPNQGYVP32Y3P6WL
[ OK ] pushed to a total of 1 nats-server
```
```
nats -s nats://localhost:4222 pub --creds=/DEMO/TEST/foo.creds "hello" "w
```

#### Create a user as follows:

#### nsc add user --account <account name> --name <user name> -i

#### nsc edit user [flags] can subsequently be used to modify the user. In case you

#### have multiple account signing keys, for either command, -i will prompt you to select

#### one.

#### In case you generate a user on behalf of another entity that has no nsc environment, you

#### may want to consider not exchanging the NKEY. 1. To do this, have the other entity

#### generate a user NKEY pair like this: nsc generate nkey -u (--store is omitted so as

#### to not have an unnecessary copy of the key) 2. Exchange the public key printed by the

#### command via a way that assures what is used is not someone elses. 3. Create the user

#### by providing (-k) the exchanged public key

```
nsc add user --account SYS -n sys-non-op -k
UDJKPL7H6QY4KP4LISNHENU6Z434G6RLDEXL2C64YZXDABNCEOAZ4YY2
```
#### in your environment. (system account user example) 4. If desired edit the user 5. Export

#### the user nsc describe user --account SYS -n sys-non-op --raw from your

#### environment and store the output in a JWT le. 6. Exchange the JWT le 7. Use the JWT

#### le and the NKEY pair in your application.

#### nsc essentially uses the NKEY and JWT libraries to generate operator/accounts/users.

#### You can use these libraries to generate the necessary artifacts as well. Because there is

#### only one, generating the operator this way makes little sense. Accounts only if you need

#### them dynamically, say for everyone of your customer. Dynamically provision user and

#### integrate that process with your existing infrastructure, say LDAP, is the most common

#### use case for these libraries.

#### The next sub sections demonstrate dynamic user generation. The mechanisms shown

#### are applicable to dynamic account creation as well. For dynamic user/account creation,

#### signing keys are highly recommended.

### Setup User

#### Create/Edit Account - All Environments - All Deployment modes

### Automated sign up services - JWT and NKEY libraries


#### By generating users or accounts dynamically, it becomes YOUR RESPONSIBILITY to

#### properly authenticate incoming requests for these users or accounts

#### For sign up service issued JWTs, ALWAYS set the SHORTEST POSSIBLE EXPIRATION

#### This example illustrates the linear ow of the algorithm and how to use the generated

#### artifacts. In a real world application you would want this algorithm to be distributed over

#### multiple processes. For simplicity of the examples, keys may be hard coded and error

#### handling is omitted.

### Simple user creation


```
func GetAccountSigningKey() nkeys.KeyPair {
// Content of the account signing key seed can come from a file or an
accSeed := []byte("SAAJGCAHPHHM6AVJJWQ2YAS3I4NETXMWVQSTCQMJ7VVTGAJF5U
accountSigningKey, err := nkeys.ParseDecoratedNKey(accSeed)
if err != nil {
panic(err)
}
return accountSigningKey
}
```
```
func RequestUser() {
// Setup! Obtain the account signing key!
accountPublicKey := GetAccountPublicKey()
accountSigningKey := GetAccountSigningKey()
userPublicKey, userSeed, userKeyPair := generateUserKey()
userJWT := generateUserJWT(userPublicKey, accountPublicKey, accountSi
// userJWT and userKeyPair can be used in conjunction with this nats.
var jwtAuthOption nats.Option
jwtAuthOption = nats.UserJWT(func() (string, error) {
return userJWT, nil
},
func(bytes []byte) ([]byte, error) {
return userKeyPair.Sign(bytes)
},
)
// Alternatively you can create a creds file and use it as nats.Optio
credsContent, err := jwt.FormatUserConfig(userJWT, userSeed);
if err != nil {
panic(err)
}
ioutil.WriteFile("my.creds", credsContent, 0644)
jwtAuthOption = nats.UserCredentials("my.creds")
// use in a connection as desired
nc, err := nats.Connect("nats://localhost:4222", jwtAuthOption)
// ...
}
```
#### Create user NKEY


#### Inspect the user claim for all available properties/limits/permissions to set. When using

#### an account claim instead, you can dynamically generate accounts. Additional steps are to

#### push the new account as outlined here. Depending on your needs, you may want to

#### consider exchanging the accounts identity NKEY in a similar way that the users key is

#### exchanged in the next section.

```
func generateUserKey() (userPublicKey string, userSeed []byte, userKeyPai
kp, err := nkeys.CreateUser()
if err != nil {
return "", nil, nil
}
if userSeed, err = kp.Seed(); err != nil {
return "", nil, nil
} else if userPublicKey, err = kp.PublicKey(); err != nil {
return "", nil, nil
}
return
}
```
```
func generateUserJWT(userPublicKey, accountPublicKey string, accountSigni
uc := jwt.NewUserClaims(userPublicKey)
uc.Pub.Allow.Add("subject.foo") // only allow publishing to subject.f
uc.Expires = time.Now().Add(time.Hour).Unix() // expire in an hour
uc.IssuerAccount = accountPublicKey
vr := jwt.ValidationResults{}
uc.Validate(&vr)
if vr.IsBlocking(true) {
panic("Generated user claim is invalid")
}
var err error
userJWT, err = uc.Encode(accountSigningKey)
if err != nil {
return ""
}
return
}
```
#### Create user JWT

#### Distributed User Creation


#### As mentioned earlier this example needs to be distributed. This example makes uses of

#### Go channels to encode the same algorithm, uses closures to encapsulate functionalities

#### and Go routines to show which processes exist. Sending and receiving from channels

#### basically illustrates the information ow. To realize this, you can pick HTTP, NATS itself

#### etc... (For simplicity, properly closing channels, error handling, waiting for Go routines to

#### nish is omitted.)

#### The above example did not need authentication mechanisms, RequestUser possessed

#### the signing key. How you decide to trust an incoming request is completely up to you.

#### Here are a few examples:

#### In this example, this logic is encapsulated as placeholder closures

#### ObtainAuthorizationToken and IsTokenAuthorized that do nothing.

#### everyone

#### username/password

#### 3rd party authentication token


func ObtainAuthorizationToken() interface{} {
// whatever you want, 3rd party token/username&password
return ""
}

func IsTokenAuthorized(token interface{}) bool {
// whatever logic to determine if the input authorizes the requester
return token.(string) == ""
}

// request struct to exchange data
type userRequest struct {
UserJWTResponseChan chan string
UserPublicKey string
AuthInfo interface{}
}

func startUserProvisioningService(isAuthorizedCb func(token interface{})
userRequestChan := make(chan userRequest) // channel to send requests
go func() {
accountSigningKey := GetAccountSigningKey() // Setup, obtain acco
for {
req := <-userRequestChan // receive request
if !isAuthorizedCb(req.AuthInfo) {
fmt.Printf("Request is not authorized to receive a JWT, t
} else if userJWT := generateUserJWT(req.UserPublicKey, accou
req.UserJWTResponseChan <- userJWT // respond with jwt
}
}
}()
return userRequestChan
}

func startUserProcess(userRequestChan chan userRequest, obtainAuthorizati
requestUser := func(userRequestChan chan userRequest, authInfo interf
userPublicKey, _, userKeyPair := generateUserKey()
respChan := make(chan string)
// request jwt
userRequestChan <- userRequest{
respChan,
userPublicKey,
authInfo,
}
userJWT := <-respChan // wait for response
// userJWT and userKeyPair can be used in conjunction with this n
jwtAuthOption = nats.UserJWT(func() (string, error) {
return userJWT, nil
},


#### In this example the users NKEY is generated by the requesting process and the public key

#### is sent to the user sign up service. This way the service does not need to know or send

#### the private key. Furthermore, any process receiving the initial request or even response,

#### may have the user JWT but will not be able to proof possession of private NKEY.

#### However, you can have the provisioning service generate the NKEY pair and respond with

#### the NKEY pair and the user JWT. This is less secure but would enable a less complicated

#### protocol where permissable.

#### The previous example used Go channels to demonstrate data ows. You can use all sorts

#### of protocols to achieve this data ow and pick whatever ts best in your existing

#### infrastructure. However, you can use NATS for this purpose as well.

###### },

```
func(bytes []byte) ([]byte, error) {
return userKeyPair.Sign(bytes)
},
)
// Alternatively you can create a creds file and use it as nats.O
return
}
go func() {
jwtAuthOption := requestUser(userRequestChan, obtainAuthorization
nc, err := nats.Connect("nats://localhost:4222", jwtAuthOption)
if err != nil {
return
}
defer nc.Close()
time.Sleep(time.Second) // simulate work one would want to do
}()
}
```
```
func RequestUserDistributed() {
reqChan := startUserProvisioningService(IsTokenAuthorized)
defer close(reqChan)
// start multiple user processes
for i := 0; i < 4; i++ {
startUserProcess(reqChan, ObtainAuthorizationToken)
}
time.Sleep(5 * time.Second)
}
```
#### User creation using NATS


#### You can replace send and receive <- with nats publish and subscribe or - for added

#### redundancy on the sign up service - queue subscribe. To do so, you will need connections

#### that enable the sign up service as well as the requestor to exchange messages. The sign

#### up service uses the same connection all of the time and (queue) subscribes to a well

#### known subject. The requestor uses the connection and sends a request to the well known

#### subject. Once the response is received the rst connection is closed and the obtained

#### JWT is used to establish a new connection.

#### Here in lies a chicken and and egg problem. The rst connection to request the JWT itself

#### needs credentials. The simplest approach is to set up a different NATS server/cluster that

#### does not require authentication, connect rst to cluster 1 and keep requesting the user

#### JWT. Once obtained disconnect from cluster 1 and connect to cluster 2 using the

#### obtained JWT.

#### The earlier setup can be simplied by using accounts instead of separate server/clusters.

#### But a JWT/operator based setup requires JWT authentication. Thus, would be,

#### connections to a different cluster are replaced by connections to the same cluster but

#### different accounts.

#### Connections to the signup accounts use two kinds of credentials. 1. Sign up service(s)

#### use(s) credentials generated for it/them. 2. All requestors use the same JWT and NKEY,

#### neither of which are used for actual authentication.

#### Cluster 1 translates to connections to a signup account.

#### Cluster 2 translates to connections to accounts to who's signing keys have been used

#### to sign the user JWT. (This happens the rst setup as well)

#### That JWT is probably generated using nsc itself.

#### Do not use this JWT/NKEY for anything else but contacting the sign up service.

#### You want to allow publish only to the well known subject.

#### Depending on your deployment you need to back up the account (signing) NKEY so

#### that the account can be re generated without invalidating deployed requestors (which

#### Straight forward Setup

#### Account based Setup


#### The NKEY library does exist or is incorporated in all languages where NATS supports

#### NKEY. The NATS JWT library on the other hand is written in Go. This may not be your

#### language of choice. Other than encoding JWTs, most of what the that library does is

#### maintain the NATS JWT schema. If you use nsc to generate a user as a template for the

#### sign up service and work off of that template you don't need the JWT library. The sample

#### shows how a program that takes an account identity NKEY and account signing NKEY as

#### arguments and outputs a valid creds le.

#### may be hard to replace).

#### Stamping JWT in languages other than Go


using System;
using System.Security.Cryptography;
using System.Text;
using NATS.Client;
using SimpleBase;

namespace nnsc
{
internal class Signer
{
private static string issueUserJWT(string userKeyPub)
{
// Load account signing key and account identity for
// the account you wish to issue users for
const string accSeed = "SAANWFZ3JINNPERWT3ALE45U7GYT2ZDW6GJUIVPDKUF
const string accId = "ACV63DGCZGOIT3P5ZA7PQT3KYJ6UDFFHZ7KETHYMDMZ4
NkeyPair accountSigningKey = Nkeys.FromSeed(accSeed);
string accSigningKeyPub = Nkeys.PublicKeyFromSeed(accSeed);

// Use nsc to create a user any way you like.
// Export the user as json using:
// nsc describe user --name <user name> --account <account name> --
// Turn the output into a format string and replace values you want
// Fields that need to be replaced are:
// iat (issued at), iss (issuer), sub (subject) and jti (claim hash
const string claimFmt = @"{{
""iat"": {0},
""iss"": ""{1}"",
""jti"": ""{2}"",
""name"": ""{3}"",
""nats"": {{
""data"": -1,
""issuer_account"": ""{4}"",
""payload"": -1,
""pub"": {{}},
""sub"": {{}},
""subs"": -1,
""type"": ""user"",
""version"": 2
}},
""sub"": ""{3}""
}}";
const string header = @"{
""typ"":""JWT"",
""alg"":""ed25519-nkey""
}";

```
// Issue At time is stored in unix seconds
```

// Issue At time is stored in unix seconds
long issuedAt = ((DateTimeOffset) DateTime.Now).ToUnixTimeSeconds()
// Generate a claim without jti so we can compute jti off of it
string claim = string.Format(
claimFmt,
issuedAt,
accSigningKeyPub,
"", /* blank jti */
userKeyPub,
accId);
// Compute jti, a base32 encoded sha256 hash
string jti = Base32.Rfc4648.Encode(
SHA256.Create().ComputeHash(Encoding.UTF8.GetBytes(claim)),
false);
// recreate full claim with jti set
claim = string.Format(
claimFmt,
issuedAt,
accSigningKeyPub,
jti,
userKeyPub,
accId
);
// all three components (header/body/signature) are base64url encod
string encHeader = ToBase64Url(Encoding.UTF8.GetBytes(header));
string encBody = ToBase64Url(Encoding.UTF8.GetBytes(claim));
// compute the signature off of header + body (. included on purpos
byte[] sig = Encoding.UTF8.GetBytes($"{encHeader}.{encBody}");
string encSig = ToBase64Url(accountSigningKey.Sign(sig));
// append signature to header and body and return it
return $"{encHeader}.{encBody}.{encSig}";
}

private static string issueUserCreds()
{
// Generate a user NKEY for the new user.
// The private portion of the NKEY is not needed when issuing the jw
// Therefore generating the key can also be done separately from th
// Say by the requester.
const string userSeed = Nkeys.CreateUserSeed();
const string userKeyPub = Nkeys.PublicKeyFromSeed(userSeed);
string jwt = issueUserJWT(userKeyPub);
// return jwt and corresponding user seed as creds
return $@"-----BEGIN NATS USER JWT-----
{jwt}
------END NATS USER JWT------

************************* IMPORTANT *************************
NKEY Seed printed below can be used to sign and prove identity.


#### The system account is the account under which nats-server offer services. To use it

#### either the operator JWT has to specify it, which happens during nsc init or when

#### providing --sys to nsc add operator. Alternatively you can encode it in the server

#### conguration by providing system_account with the public NKEY of the account you

#### want to be the system account:

#### It is NOT recommended to use this account to facilitate communication between your

#### own applications. Its sole purpose is to facilitate communication with and between

#### nats-server.

```
NKEYs are sensitive and should be treated as secrets.
```
```
-----BEGIN USER NKEY SEED-----
{userSeed}
------END USER NKEY SEED------
```
```
*************************************************************";
}
```
```
private static string ToBase64Url(byte[] input)
{
var stringBuilder = new StringBuilder(Convert.ToBase64String(input)
stringBuilder.Replace('+', '-');
stringBuilder.Replace('/', '_');
return stringBuilder.ToString();
}
```
```
private static void Main(string[] args)
{
string creds = issueUserCreds();
Console.WriteLine(creds);
}
}
}
```
```
system_account: AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLKU7A5
```
### System Account

#### Event Subjects


#### Events are published as they happen. But you MUST NOT rely on a particular ordering or

#### due to the possibility of loss, events matching. Say, CONNECT for a client always

#### matching a DISCONNECT for the same client. Your subscriber may simply be

#### disconnected when either event happens. Some messages carry aggregate data and are

#### periodically emitted. There, missing a message for one reason or another is

#### compensated by the next one.

#### The subject $SYS.SERVER.ACCOUNT.<account-id>.CONNS is still used but it is

#### recommended to subscribe to it's new name

#### $SYS.ACCOUNT.<account-id>.SERVER.CONNS.

```
Subjects to subscribe on Description Repeats
```
```
$SYS.SERVER.<server-
id>.SHUTDOWN
```
```
Sent when a server shuts down
```
```
$SYS.SERVER.<server-
id>.CLIENT.AUTH.ERR
```
```
Sent when client
fails to authenticate
```
```
$SYS.SERVER.<server-id>.STATSZ Basic server stats Periodically
```
```
$SYS.ACCOUNT.<account-
id>.LEAFNODE.CONNECT
```
```
Sent when Leafnode connected
```
```
$SYS.ACCOUNT.
<account-id>.CONNECT
```
```
Sent when client connected
```
```
$SYS.ACCOUNT.<account-
id>.DISCONNECT Sent when Client disconnected
```
```
$SYS.ACCOUNT.<account-
id>.SERVER.CONNS
```
```
Sent when an accounts
connections change
```
### Service Subjects

#### Subjects always available


**Subjects to publish requests to Description Message Output**

```
$SYS.REQ.SERVER.PING.
STATZ
```
```
Exposes the STATZ HTTP
monitoring endpoint, each server
will respond with one message
```
```
Same as
HTTP endpoint
```
```
$SYS.REQ.SERVER.PING.
VARZ
```
- same as above for - VARZ - same as above -

```
$SYS.REQ.SERVER.PING.
SUBZ
```
- same as above for - SUBZ - same as above -

```
$SYS.REQ.SERVER.PING.
CONNZ - same as above for - CONNZ - same as above -
```
```
$SYS.REQ.SERVER.PING.
ROUTEZ - same as above for - ROUTEZ - same as above -
```
```
$SYS.REQ.SERVER.PING.
GATEWAYZ
```
- same as above for - GATEWAYZ - same as above -

```
$SYS.REQ.SERVER.PING.
LEAFZ
```
- same as above for - LEAFZ - same as above -

```
$SYS.REQ.SERVER.PING.
ACCOUNTZ - same as above for - ACCOUNTZ - same as above -
```
```
$SYS.REQ.SERVER.PING.
JSZ - same as above for - JSZ - same as above -
```
```
$SYS.REQ.SERVER.
<server-id>.STATZ
```
```
Exposes the STATZ HTTP monitoring
endpoint, only requested server responds
```
```
Same as
HTTP endpoint
```
```
$SYS.REQ.SERVER.
<server-id>.VARZ
```
- same as above for - VARZ - same as above -

```
$SYS.REQ.SERVER.
<server-id>.SUBZ
```
- same as above for - SUBZ - same as above -

```
$SYS.REQ.SERVER.
<server-id>.CONNZ - same as above for - CONNZ - same as above -
```

#### Each of the subjects can be used without any input. However, for each request type (

#### STATZ, VARZ, SUBSZ, CONNS, ROUTEZ, GATEWAYZ, LEAFZ, ACCOUNTZ, JSZ) a json

#### with type specic options can be sent. Furthermore all subjects allow for ltering by

#### providing these values as json:

```
Subjects to publish requests to Description Message Output
```
```
$SYS.REQ.SERVER.
<server-id>.ROUTEZ
```
- same as above for - ROUTEZ - same as above -

```
$SYS.REQ.SERVER.
<server-id>.GATEWAYZ - same as above for - GATEWAYZ - same as above -
```
```
$SYS.REQ.SERVER.
<server-id>.LEAFZ
```
- same as above for - LEAFZ - same as above -

```
$SYS.REQ.SERVER.
<server-id>.ACCOUNTZ - same as above for - ACCOUNTZ
```
- same as above -

```
$SYS.REQ.SERVER.
<server-id>.JSZ
```
- same as above for - JSZ - same as above -

```
$SYS.REQ.ACCOUNT.
<account-id>.SUBSZ
```
```
Exposes the SUBSZ HTTP monitoring
endpoint, ltered by account-id.
```
```
Same as
HTTP endpoint
```
```
$SYS.REQ.ACCOUNT.
<account-id>.CONNZ
```
- same as above for CONNZ - - same as above -

```
$SYS.REQ.ACCOUNT.
<account-id>.LEAFZ - same as above for LEAFZ -
```
- same as above -

```
$SYS.REQ.ACCOUNT.
<account-id>.JSZ
```
- same as above for JSZ - - same as above -

```
$SYS.REQ.ACCOUNT.
<account-id>.CONNS
```
```
Exposes the event
$SYS.ACCOUNT.<account-
id>.SERVER.CONNS
as request
```
- same as above -

```
$SYS.REQ.ACCOUNT. Exposes account specic^ Similar to
```

```
Option Effect
```
```
server_name Only server with matching server name will respond.
```
```
cluster Only server with matching cluster name will respond.
```
```
host Only server running on that host will respond.
```
```
tags Filter responders by tags. All tags must match.
```
```
Subject Description Input Output
```
```
$SYS.REQ.ACCOUN
T.<account-
id>.CLAIMS.UPDA
TE
```
```
Update a particular
account JWT
(only possible if
properly signed)
```
```
JWT body
```
```
$SYS.REQ.ACCOUN
T.<account-
id>.CLAIMS.LOOK
UP
```
```
Responds with
requested JWT
```
```
JWT body
```
```
$SYS.REQ.CLAIMS
.PACK
```
```
Single responder
compares input,
sends all JWT
if different.
```
```
xor of all
sha256(stored-
jwt). Send empty
message to
download all JWT.
```
```
If different,
responds with all
stored JWT (one
message per JWT).
Empty message
to signify EOF
```
```
$SYS.REQ.CLAIMS
.LIST
```
```
Each server
responds with list of
account ids it stores
```
```
list of account ids
separated by newline
```
```
$SYS.REQ.CLAIMS
.UPDATE
```
```
Exposes
$SYS.REQ.ACCOUN
T..CLAIMS.UPDATE
without the need for
<account-id>
```
```
JWT body
```
```
$SYS.REQ.CLAIMS
.DELETE
```
```
When the resolver
is congured with
```
```
Generic operator
signed JWT
```
#### Subjects available when using NATS-based resolver


#### It is important to understand that leaf nodes do not multiplex between accounts. Every

#### account that you wish to connect across a leaf node connection needs to be explicitly

#### listed. Thus, the system account is not automatically connected, even if both ends of a

#### leaf node connection use the same system account. For leaf nodes connecting into a

#### cluster or super cluster, the system account needs to be explicitly connected as separate

#### remote to the same URL(s) used for the other account(s). The system account user

#### used by providing credentials can be heavily restricted and for example, only allow

#### publishing on some subjects. This also holds true when you don't use the system account

#### yourself, but indirectly need it for NATS based account resolver or centralized monitoring.

#### Examples in sub sections below assume that the cluster to connect into is in operator

#### mode.

#### The outgoing connection is not in Operator mode, thus the system account may differ

#### from the user account. This example shows how to congure a user account and the

#### system account in a leaf node. Credentials les provided have to contain credentials that

#### are valid server/cluster reachable by url. In the example, no accounts are explicitly

```
Subject Description Input Output
allow_delete:
true
, deleting accounts
is enabled.
```
```
claim with a eld
accounts
containing a list of
account ids.
```
```
Subject Alternative Mapping
```
```
$SYS.REQ.SERVER.PING $SYS.REQ.SERVER.PING.STATSZ
```
```
$SYS.ACCOUNT.<account-
id>.CLAIMS.UPDATE
```
```
$SYS.REQ.ACCOUNT.<account-
id>.CLAIMS.LOOKUP
```
#### Old Subjects

### Leaf Node Connections - Outgoing

#### Non Operator Mode


#### congured, yet some are referenced. These are the default Account $G and the default

#### system account $SYS

#### Outgoing connection is in operator mode as well. This example assumes usage of the

#### same operator and thus system account. However, using a different operator would look

#### almost identical. Only the credentials would be issued by accounts of the other operator.

```
leafnodes {
remotes = [
{
url: "nats://localhost:4222"
credentials: "./your-account.creds"
},
{
url: "nats://localhost:4222"
account: "$SYS"
credentials: "./system-account.creds"
},
]
}
```
```
operator: ./trustedOperator.jwt
system_account: AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLKU7A5
leafnodes {
remotes = [
{
url: "nats://localhost:4222"
account: "ADKGAJU55CHYOIF5H432K2Z2ME3NPSJ5S3VY5Q42Q3OTYOCYRRG7W
credentials: "./your-account.creds"
},
{
url: "nats://localhost:4222"
account: "AAAXAUVSGK7TCRHFIRAS4SYXVJ76EWDMNXZM6ARFGXP7BASNDGLKU
credentials: "./system-account.creds"
},
]
}
```
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


