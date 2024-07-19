# Running Workloads on NATS

The NATS Execution Engine (we'll just call it **Nex** most of the time) is an optional add-on
to NATS that overlays your existing NATS infrastructure, giving you the ability to deploy
and run workloads.
Building applications that rely on NATS as a backbone already gives us a tremendous
amount of power and features out of the box, including message communications, global
reach, reliability, and additional services such as streams, key-value stores, and object
stores. However, prior to the availability of **Nex** , we haven't been able to use that
infrastructure as a deployment target.

```
The NATS execution engine is currany APIs are likely to change frequentlyently. experimental. The documentation, feature set, and
```
While you can build virtually any kind of application with Nex, the core building blocks are
made up of two fundamental types of workloads: **services** and **functions**.

Within the context of **Nex** , a service is just a long-running process. When it is deployed, it
is launched with a given environment and runs continually until terminated. Typical
examples of service-type workloads are web servers, monoliths, API endpoints, and
applications that maintain long-running subscriptions to NATS subjects.
The default, secure Nex services are statically-linked, 64-bit linux (elf) binaries but you
can also run native Mac or Windows binaries as services and you can customize the root
le system used for the Firecracker sandbox. Support for OCI (e.g. docker) images is
coming soon.

## Services

## Functions


Nex functions are short-lived processes. They are executed in response to some trigger
where the functions can process the trigger's data and optionally supply a return value.
Nex functions are small and can be deployed either as JavaScript functions or as
WebAssembly modules.


# Getting Started

This getting started guide will get you started deploying workloads with the new
execution engine. You'll get a taste of all of the basics of Nex, including setup,
installation, and developing and deploying both services and functions.

```
If you intend tmachine with viro use the securtualization enabled in the ke sandbox mode, makernel. Nex'e sure you're running on a 64-bit Linuxs default secure mode makes
extensive use of Firecracker, which makes some pretty strict demands of its environment.
```

# Installing Nex

All of the functionality you need for Nex is conveniently wrapped up in a single command
line tool. To install it, enter the following command in a terminal:

Depending on your operating system and user privileges, you may need to change sh to
sudo sh so the script can place the nex binary in your path.
If you're not comfortable running this command, you can manually install Nex by
downloading the latest version from the releases page and simply place the nex binary
somewhere in your path.

**Note** functionality ma that while the y only available on 64-bit Linux because of the rnex binary can be run on any operating system, some nodeequirements dictated b (^) y
Firecracker. Also note that running Linux inside docker won't satisfy those requirements.
Once you've installed it, you should be able to check the CLI version with nex version.
After you're able to get the help banner and version from nex, you can move on to the
next step in this guide.
Starting a Nex node can involve the use of the Linux kernel, the firecracker binary, CNI
conguration les, an ext4 root le system, machine conguration, and a handful of
other things. That's a lot to keep track of, so Nex has conveniently provided a preight
check. Before you can run a preight check, however, you need to create a node
conguration le.
curl -sSf https://nex.synadia.com/install.sh | sh

## Performing the Preflight Check

## Creating a Node Configuration


The easiest way to create a node conguration le is to copy one from the Nex examples
folder, such as the simple.json le, which contains the following JSON:

```
Note that if yafter a reboot and you do end up using ou'll have to run a pr/tmp/wdeight check again. as your resource directory, all of that will go away
```
This conguration le will look for a linux kernel le (vmlinux) and a root le system (
rootfs.ext4) in the default resource directory. You can override either of these
lenames by supplying the kernel_file or rootfs_file elds.
Put this conguration le anywhere you like, but preflight will check ./config.json
by default. For each dependency preflight doesn't nd, it can create default
conguration les and download missing dependencies such as the firecracker
binary, a Linux kernel, and our vetted root le system.
After you've run preflight and it downloaded all of the missing components, run it one
more time to make sure your output looks similar to what is shown below. There should
be a green checkmark for each of the checklist items. Make sure everything checks out
before continuing.

```
{ "default_resource_dir":"/tmp/wd",
"machine_pool_size""cni": { : 1 ,
"network_name""interface_name": "fcnet": "veth0",
} "machine_template", : {
```
"vcpu_count""memsize_mib": (^) : 1 , 256
} "tags", : {
(^) } "simple": "true"
}


With a running NATS server and a passing pre-ight checklist, you're ready to start
running workloads on NATS!

```
$ nex node preflight --config=../examples/nodeconfigs/simple.jsonValidating - Required CNI Plugins [/opt/cni/bin]
✅✅ Dependency Satisfied - /opt/cni/bin/host-local [host-local CNI plu Dependency Satisfied - /opt/cni/bin/ptp [ptp CNI plugin]
✅ Dependency Satisfied - /opt/cni/bin/tc-redirect-tap [tc-redirect-t
Validating - Required binaries [/usr/local/bin]✅ Dependency Satisfied - /usr/local/bin/firecracker [Firecracker VM
Validating - CNI configuration requirements [/etc/cni/conf.d]✅ Dependency Satisfied - /etc/cni/conf.d/fcnet.conflist [CNI Configu
Validating - User provided files []✅ Dependency Satisfied - /tmp/wd/vmlinux [VMLinux Kernel]
✅ Dependency Satisfied - /tmp/wd/rootfs.ext4 [Root Filesystem Templa
```

# Building a Service

Building a service destined for deployment via Nex is just like building any other service.
We don't make any demands of your application or its dependencies, only that it meet the
sandboxing requirements of the node on which it will run. There's no proprietary SDK, no
custom build tools, no complicated dependency tree.
If you're building "cloud native" or 12 factor services as single-binary deployments, you
should have no trouble running them in Nex.
A service, as understood by Nex, is nothing more than a long-running process. A service
needs to start (e.g. it has a main function/entrypoint) and it needs to continue running
until the host environment tells it to shut down.

For this example, we're going to use Go to create a simple executable. We'll use the NATS
services framework to expose an endpoint for service discovery.
Open up your favorite editor and edit a main.go le with the following content:

## Creating a NATS Service


package main
import"context" (
"fmt""os"
"strings"
"github.com/nats-io/nats.go"services "github.com/nats-io/nats.go/micro"
)

func (^) ctx main:=() { context.Background()
natsUrl if len(strings.:= os.GetenvTrimSpace("NATS_URL"(natsUrl)) ) == 0 {
} natsUrl = nats.DefaultURL
fmt.nc, err Printf:=( nats."Echo service using NATS url 'Connect(natsUrl) %s'\n", natsUrl)
if err panic!= (^) (err)nil {
}
// request handlerechoHandler := func(req services.Request) {
} req.Respond(req.Data())
fmt.Println("Starting echo service")
_, err Name: = services."EchoService"AddService, (nc, services.Config{
Version: // base handler"1.0.0",
Endpoint: Subject: &services"svc.echo".EndpointConfig, {
}, Handler: services.HandlerFunc(echoHandler),
})
if err panic!= (^) (err)nil {
}
} <-ctx.Done()


However you choose to satisfy the dependencies (e.g. with a go.mod le), y ou'll need
the Go NATS client SDK:

The code is pretty straightforward if you're already used to building applications atop
NATS infrastructure. We establish a connection to a NATS server as indicated by the
NATS_URL environment variable.
Next, we create a discoverable service called EchoService with an endpoint on the
svc.echo subject. This means that the handler we've dened (echoHandler) will
respond to requests on that subject.

To make sure that there are no cards up our proverbial sleeves, let's run this service
without using Nex at all. In one terminal, issue a go run main.go command, while in
another terminal, make a request on the svc.echo subject:

Finally, let's also make sure that this service is behaving properly according to the NATS
services specication:

```
}
```
```
$ go get github.com/nats-io/nats.go
```
```
$ nats req svc.echo 'this is a test'17:10:53 Sending request on "svc.echo"
17:10:53 Received with rtt 567.537μsthis is a test
```
#### $ nats micro ls╭─────────────────────────────────────────────

#### │├─────────────┬─────────┬───────────────────── All Micro Services │

#### │├─────────────┼─────────┼───────────────────── Name │ Version │ ID │ Description │

#### │╰─────────────┴─────────┴───────────────────── EchoService │ 1.0.0 │ FKIuiivKgSB8VWyjDYBpEc │ │

## Running our Service


Perfect! Our service is doing everything it's supposed to, and we don't need Nex to test it.
This might seem like a subtle point, but it's incredibly powerful that Nex services are **not
tightly coupled** to their means of deployment or scheduling runtime.

While it's easy enough to test our service locally via go run ..., in order for our service
to be deployable via Nex, it needs to be a statically linked executable. Thankfully, Go
makes this easy.
In the same directory as your main.go, run the following Go command:

If everything went well, you should see no output, but you'll be able to verify that your
service is the right kind of le:

In addition to Go, many other languages are just as capable of producing statically linked
binaries. For example, if you're using Rust you can just set your target to
x86_64-unknown-linux-musl or aarch64-unknown-linux-musl for the same effect.
With our statically compiled service in hand, we have one more thing to do before we can
start deploying, and that's starting a Nex node process.

```
$ CGO_ENABLED=0 go build -tags netgo -ldflags '-extldflags "-static"'
```
```
$ file echoserviceechoservice: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statica
```
## Static Compilation


# Starting a Node

If we've performed our pre-ight checks and they've passed, then starting a Nex node is
as easy as pointing the nex node command at the checked conguration le.

The nex node process makes a number of secure kernel calls in order to set up CNI
networks and start the firecracker binary. While it might be possible to manually craft
a user with just the right set of permissions such that sudo is unnecessary, when you're
in your development loop, sudo is probably the easiest way to go.

The command to bring up a new node process is node up. There are a lot of options
that you can specify here, as well as options available in the conguration le. All of those
details can be found in the reference section.

```
$ sudo ./nex node up --config=/home/kevin/simple.jsonINFO[0000] Established node NATS connection to: nats://127.0.0.1:
INFO[0000] Loaded node configuration from '/home/kevin/simple.json' INFO[0000] Virtual machine manager starting
INFO[0000] Internal NATS server started client_url="natsINFO[0000] Use this key as the recipient for encrypted run requests publ
INFO[0000] NATS execution engine awaiting commands id=NCOBPU3MCEA7LINFO[0000] Called startVMM(), setting up a VMM on /tmp/.firecracker.sock-
INFO[0000] VMM metrics disabled. INFO[0000] refreshMachineConfiguration: [GET /machine-config][200] getMac
INFO[0000] PutGuestBootSource: [PUT /boot-source][204] putGuestBootSourceINFO[0000] Attaching drive /tmp/rootfs-cmjg61n52omq8dovolmg.ext4, slot 1,
INFO[0000] Attached drive /tmp/rootfs-cmjg61n52omq8dovolmg.ext4: [PUT /drINFO[0000] Attaching NIC tap0 (hwaddr 5a:65:8e:fa:7f:25) at index 1
INFO[0000] startInstance successful: [PUT /actions][204] createSyncActionINFO[0000] SetMetadata successful
INFO[0000] Machine started gateway=192.168.INFO[0000] Adding new VM to warm pool ip=192.168.127.
INFO[0001] Received agent handshake message="Host-su
```
## Superuser Permissions

## Node Up


This might seem a bit spammy, but the vast majority of this is full of useful information.
The rst thing you see is that the node established a connection to NATS on
127.0.0.1:4222. This is the control plane connection, e.g. the connection through
which Nex remote management commands are issued. This should not be confused with
the NATS connection that individual workloads use, as that should absolutely be a
different connection for security reasons.

The next two lines of the node output log also give important details:

First, you see an **Xkey** , which is the target public key that should be used when encrypting
data bound for this particular node. This is used to encrypt the environment variables that
are supplied with workload deployment requests. Don't worry, though, the nex CLI takes
care of this for you.

Next up in the node's output log is a line like this one:

This indicates that we've actually started a Firecracker virtual machine and, for
information and debugging purposes, we output the gateway IP address, the tap device
name, the internal NATS host, internal nats port, and ultimately the ID of the virtual
machine.
The Nex node process keeps a pool of "warm" virtual machines, the size of which is
congurable in the node conguration le. When you issue a work deployment request,
the next available warm VM is taken from the pool and the work is dispatched to it.

```
INFO[0000] Use this key as the recipient for encrypted run requests publINFO[0000] NATS execution engine awaiting commands id=NCOBPU3MCEA7L
```
```
INFO[0000] Machine started gateway=192.168.
```
## Encrypted Environment Variables

## Filling the Virtual Machine Pool


Probably the single most important piece of log output is the indication of a successful
handshake. Each time we attempt to add a virtual machine to the pool, the **agent** running
inside tries to make contact with the host node process. If this exchange is successful,
you'll see the following message:

If you don't, and instead see an error indicating a timeout or failure to perform the
handshake, the node will terminate. If a node can't properly dispense virtual machines,
then there's no reason for it to continue running.

With this node still running, open another terminal and issue the following nex
command:

This will show you the summary information of all discovered nodes. Any given NATS
infrastructure can run as many Nex nodes in as many locations and clusters as desired.
Now let's move on to deploying our echo service.

```
INFO[0001] Received agent handshake message="Host-su
```
#### $ nex node ls╭─────────────────────────────────────────────

#### │├───────────────────────────────────────────── NATS Execution Nodes

#### │├───────────────────────────────────────────── ID │ Version │ U

#### │╰───────────────────────────────────────────── NCOBPU3MCEA7LF6XADFD4P74CHW2OL6GQZYPPRRNPDSBNQ5BJPFHHQB5 │ 0.0.1 │ 1

## Readiness Indicator

## Interrogating Nodes


# Deploying Services

There are a number of ways to deploy a service with Nex. For this guide we'll cover the
easiest option, devrun, which makes a number of assumptions about the fact that
you're running in a development environment. For all of the full production options, take a
look at the reference section or read through the CLI's extended help text.
Make sure that your node is still up and running by making sure that it's still discoverable
via nex:

We start services by issuing a deployment (which ends up being equivalent to "run" for
services) request to a specic node. When we use the devrun command, the CLI makes
it easy on us by choosing the rst discovered node as the target.
The next thing we need on hand when calling devrun is the statically linked binary.
Deploying workloads also requires a publisher key and an encryption **xkey** , but devrun
will create both of those if you haven't done so already.
As you saw, the echo service we built requires the NATS_URL environment variable. To
start the echo service on the rst available Nex node and supply an environment variable,
issue the following command (your path to the echoservice le ma y differ):

#### $ nex node ls╭─────────────────────────────────────────────

#### │├───────────────────────────────────────────── NATS Execution Nodes

#### │├───────────────────────────────────────────── ID │ Version │ U

#### │╰───────────────────────────────────────────── NCOBPU3MCEA7LF6XADFD4P74CHW2OL6GQZYPPRRNPDSBNQ5BJPFHHQB5 │ 0.0.1 │ 1

```
$ nex devrun ../examples/echoservice/echoservice nats_url=nats://192.168.Reusing existing issuer account key: /home/kevin/.nex/issuer.nk
Reusing existing publisher xkey: /home/kevin/.nex/publisher.xk🚀 Workload 'echoservice' accepted. You can now refer to this workload wi
```
## Sending a Deployment Request


There's a couple of important pieces of information here. The rst is that we've reused
some existing keys. If this is your rst time running a workload, you'll see those two keys
get created. Next, we see that we got an acknowledgement from the target node that
included the machine/workload ID.
The address 192.168.127.1 is the IP address of the host (the network on which nex is
running), as seen by any code running inside the r ecracker VM (guest). We use this as a
default because it makes things easy during development, but keep in mind that if you
supply your own custom CNI congurations, the IP address that works in your
environment may be different.
Let's run the same commands we ran before to test out our service.

And we can test this service out the same way we did before:

Now let's interrogate a single execution node to see the workload there (again note that
your node ID will differ from the one below):

#### $ nats micro ls╭─────────────────────────────────────────────

#### │├─────────────┬─────────┬───────────────────── All Micro Services │

#### │├─────────────┼─────────┼───────────────────── Name │ Version │ ID │ Description │

#### │╰─────────────┴─────────┴───────────────────── EchoService │ 1.0.0 │ NsMaTbN7u5ZPUNN47bSEI6 │ │

```
nats req svc.echo 'hey'19:40:22 Sending request on "svc.echo"
19:40:22 Received with rtt 446.27μshey
```

And just to prove that we're not interfering with the way the workload executes at all, we
can get the service stats and see that it is indeed still keeping track of request counts:

```
$ nex node info NBS3Y3NWXLTFNC73XMVD6USFJF2H5QXTLEJQNOPEBPYDUDVB5YYYZOGINEX Node Information
Node: NBS3Y3NWXLTFNC73XMVD6USFJF2H5QXTLEJQNOPEBPYDUDVB5YYYZOGI Xkey: XASQSWNSIKHM5MDKDOGPSPGBA3V6JMETMIJK2YTXKAJZNMAFKXER5RUK
Version: 0.0.1 Uptime: 8m47s
Tags: nex.arch=amd64, nex.cpucount=8, nex.os=linux, simple=true
Memory in kB:
Free: 33,545,884 Available: 56,529,
Total: 63,883,
Workloads:
Id: cmji29n52omrb71g07a0 Healthy: true
Runtime: 8m47s Name: echoservice
Description: Workload published in devmode
```

That's it! Congratulations, you've got a running Nex node that is ready and willing to
accept and run any kind of workload you can throw at it! In the next section of this guide,
we'll create, deploy, and manage functions. To keep things easy, you should keep your
Nex node running throughout the rest of this guide.

```
$ nats micro info EchoServiceService Information
Service: EchoService (NsMaTbN7u5ZPUNN47bSEI6) Description:
Version: 1.0.
Endpoints:
Name: default Subject: svc.echo
Queue Group: q
Statistics for 1 Endpoint(s):
default Endpoint Statistics:
Requests: 1 in group q Processing Time: 15μs (average 15μs)
Started: 2024-01-16 19:40:09 (7m46s ago) Errors: 0
```

# Building a Function

Nex functions can be thought of as similar to cloud functions or "lambdas". Nex functions
are short-lived and are executed in response to some external trigger.
There are two kinds of Nex functions: **JavaScript** and **WebAssembly**. We'll take a look at
both of those in this section of the guide.

Nex functions ar1.0. Both JavaScript functions and We undergoing rapid change as we bring the entirebAssembly functions will experience a number of APIe project closer toward a (^)
changes as we prfor Wasm workloads. Makovide host sere sure you check the vices (see below) texampleso both and le folder frverage the component modelequently to see if
anything has changed.
Writing a JavaScript function is as easy as it sounds. Write the function that does what
you need it to do and you're ready to go. For example, here's a JavaScript function that
works in Nex and simply "echoes" the incoming request:
Here the function is passed two parameters:
The function returns a binary payload to be used as the response (or an empty payload
for no response). Note that as it stands, the function can produce very few side effects.
As we grow the "host services" (discussed shortly) functionality, JavaScript functions will
be able to leverage more of NATS's core functionality and JetStream.
(subject console, payload) .log(subject);=> {
(^) };return payload;
subjects in the next section on deplosubject - The subject on which the function was triggeryment) ed (we'll cover trigger^
payload - The raw binary payload that was supplied along with the trigger

## Writing a JavaScript Function


While you'll only be deploying the single .js le, y ou're free to use whatever other
testing and build tools you like.

In order to support the largest possible number of languages and runtimes, Nex's
WebAssembly functions follow the command pattern using WASI. This means that the
module's main (or start or _start depending on your perspective) function is
executed every time a trigger occurs and input is supplied via stdin and the function's
response is provided via stdout.
Nex-compatible WebAssembly functions can be written in any language that can
generate a freestanding wasm32-wasi module that has no host JavaScript
requirements. For example, if you're writing a wasm function in Go and you've imported
syscall/js directly or transitively, your function isn't going to work.
In this section we're going to create a WebAssembly function for Nex using Rust, one of
the most well-tooled languages in the WebAssembly ecosystem. Don't worry if you don't
have Rust installed, you can still follow along and deploy a pre-built module in the next
section.
Create a new Rust executable project (not library). Modify the Cargo.toml le t o read as
follows:

Now edit src/main.rs to have the following contents:

```
[package]name = "echofunction"
version edition == "0.1.0""2021"
[[bin]]name = "echofunction"
path = "src/main.rs"
```
## Writing a WebAssembly Function


This Rust WebAssembly function takes the subject on which it was triggered and an
incoming payload and returns that payload prepended by the subject. This way it's easy to
see what's happening when you trigger the function.
Build this function into a module using the following command:

This will put echofunction.wasm in the ./target/wasm32-wasi/release directory.
This function takes up about **2MB** but can be shrunk down below **700KB** using public
wasm tools. Your deployable WebAssembly functions use less disk space than even the
smallest of memes.

Small, fast functions like this are perfect for doing things like transforming data or
performing ad hoc calculations. As such, they lend themselves to being used as "pure

```
use std::{env, io::{self, Read, Write}};
fn mainlet() { args: Vec<String> = env::args().collect();
// When a WASI trigger executes:// argv[1] is the subject on which it was triggered
// stdin bytes is the raw input payload// stdout bytes is the raw output payload
```
(^) iolet:: mutstdin buf ().=read_to_end Vec::new();(&mut buf).unwrap();
(^) let mut subject = args[ 1 ].as_bytes().to_vec();
buf.append(&mut subject);
// This just returns the payload concatenated with the// subject
(^) io::stdout().write_all(&mut buf).unwrap();
}
$ cargo build --target wasm32-wasi --release

## Host Services


functions". A pure function is just a function that has no side effects.
However, not everything people need to do with functions can be represented as a pure
function with no I/O or side effects. This is where host services come in. In nearly every
FaaS or Cloud Function runtime available in the cloud, you usually get access to some
kind of SDK that grants your function some basic capabilities.
Lambda functions deployed in AWS have access to a subset of the AWS SDK, likewise
with functions deployed in Azure and Google. Even so-called "edge functions" have
access to tiny, optimized, edge versions of resources like key-value buckets, object
stores, and more.
During the rst experimental pre-release phase, we have a small number of services
available for functions. At the moment, JavaScript functions have access to the following
host services:

If the WASI component model for WebAssembly matures enough to the point where we
think it will give our developers a worthy experience, then we can provide host services
through the "WASI cloud" set of contracts. If the component model doesn't support a
good enough developer experience within our timeframe, then we may end up providing
our own contracts for host services.
Stay tuned to our blog and social media outlets for news as we enhance and provide
more host services.

```
Core Messaging, e.g. Pub, Sub, Request
Key-Value Store
Object Store
HTTP Client
```

# Deploying Functions

Deploying functions to Nex is just as easy as deploying services. The pattern is the same
for both WebAssembly and JavaScript type functions.

With Nex functions you can specify a list of trigger subjects (which can include
wildcards) used to activate them. So let's say you've deployed a calculator service, you
may have chosen calc.* as the trigger subject. This means that when a message
comes in on a subject like calc.add, your function will be called. It will be passed the
subject calc.add and the payload supplied on the core NATS message.
If your function returns a payload, and you used a request (instead of publish) to trigger
the function, that return payload will be supplied as the response body.
While the subject trigger mechanism is incredibly exible and powerful, we are actively
thinking of additional ways we might be able to trigger functions, such as pull consumers
on streams, watchers on K/V or object stores, etc.

Let's deploy our JavaScript function. We're going to use the trigger subject js.echo so
we can differentiate from the WebAssembly function. Issue the following command (your
path to the JavaScript le will likely be different):

Let's make sure the function is alive and can be triggered on the right subject:

```
$ nex devrun /home/kevin/echofunction.js --trigger_subject=js.echoReusing existing issuer account key: /home/kevin/.nex/issuer.nk
Reusing existing publisher xkey: /home/kevin/.nex/publisher.xk🚀 Workload 'echofunctionjs' accepted. You can now refer to this workload
```
## Function Triggers

## Deploying JavaScript Functions


And let's make sure the workload is visible on the node (your node ID will be different):

Everything's working as intended. Great!

Now let's deploy our WebAssembly function. If you didn't build yours locally, there's a
downloadable echofunction.wasm in the examples folder in the Github repository.
Deploying this le works the same way as deploying the JavaScript function:

```
$ nats req js.echo 'heya'09:40:33 Sending request on "js.echo"
09:40:33 Received with rtt 2.600724ms"heya"
```
```
$ nex node info NC7PXV2DLGXC4LTVM7W7MXYL3WVQFA345IFKJOMYA5ZDZMACLZ53NIILNEX Node Information
Node: NC7PXV2DLGXC4LTVM7W7MXYL3WVQFA345IFKJOMYA5ZDZMACLZ53NIIL Xkey: XDKZMOZKVBXSY3YXPIXEFKGPML75PLD7APFHZ474EOCILZDQGPZSXJNZ
Version: 0.0.1 Uptime: 2m26s
Tags: nex.arch=amd64, nex.cpucount=8, nex.os=linux, simple=true
Memory in kB:
Free: 32,354,208 Available: 55,985,740
Total: 63,883,232
Workloads:
Id: cmjud7n52omhlsa377cg Healthy: true
Runtime: 2m26s Name: echofunctionjs
Description: Workload published in devmode
```
## Deploying WebAssembly Functions


Now we should be able to trigger the function on the wasm.echo subject:

As expected, we got the payload concatenated with the trigger subject wasm.echo. We
should be able to run the nats node info command again and see both of our function
workloads:

```
$ nex devrun ../examples/wasm/echofunction/echofunction.wasm --trigger_suReusing existing issuer account key: /home/kevin/.nex/issuer.nk
Reusing existing publisher xkey: /home/kevin/.nex/publisher.xk🚀 Workload 'echofunctionwasm' accepted. You can now refer to this worklo
```
```
$ nats req wasm.echo 'hello'09:45:24 Sending request on "wasm.echo"
09:45:24 Received with rtt 42.867014mshellowasm.echo
```

Congratulations, you've now used Nex to deploy full services compiled as static binaries,
JavaScript functions, and WebAssembly functions. Deploying your applications as a
combination of services and functions with Nex is fast, easy, and sets you up to joyfully
deploy distributed applications.

```
$ nex node info NC7PXV2DLGXC4LTVM7W7MXYL3WVQFA345IFKJOMYA5ZDZMACLZ53NIILNEX Node Information
Node: NC7PXV2DLGXC4LTVM7W7MXYL3WVQFA345IFKJOMYA5ZDZMACLZ53NIIL Xkey: XDKZMOZKVBXSY3YXPIXEFKGPML75PLD7APFHZ474EOCILZDQGPZSXJNZ
Version: 0.0.1 Uptime: 7m31s
Tags: nex.arch=amd64, nex.cpucount=8, nex.os=linux, simple=true
Memory in kB:
Free: 32,280,180 Available: 56,018,344
Total: 63,883,232
Workloads:
Id: cmjud7n52omhlsa377cg Healthy: true
Runtime: 7m31s Name: echofunctionjs
Description: Workload published in devmode
Id: cmjudmn52omhlsa377d0 Healthy: true
Runtime: 6m31s Name: echofunctionwasm
Description: Workload published in devmode
```

# Host Services

If we're building cloud native/12-factor application components, then we agree that
things like connection strings, credentials, database client libraries, etc, are all things that
should be treated as external services, congured with external conguration. Very rarely
does the typical application component care about which key value store is supplying
data, only that it gets the data the environment and those who operate it make available.
This is the driving philosophy behind host services. If you're using Nex, you're already
deploying workloads that consume NATS services. We make that much easier for
functions by giving them access to managed resources such as key-value buckets,
messaging subject spaces, object stores, and even HTTP clients.


# Javascript | V8

The JavaScript functions that Nex executes are run inside a V8 sandbox, which is often
also contained within a Firecracker sandbox. As a result, these JavaScript functions don't
have the same kind of freedom of access to capabilities that other function types (e.g.
Node.js in a cloud provider). However, we've made the most common use cases available
by default through host services.
Host services aren't just access to arbitrary capabilities, they're access to managed
capabilities that are designed to be pre-provisioned and ready to go, so function
developers don't have to write provisiong code, they can focus solely on the capability
abstractions they need.

If you want to make HTTP calls from your function, you can do so with the HTTP client:

All of the HTTP methods are supported as functions:

Every Nex-managed JavaScript function has access to an abstraction over a key-value
store, which is internally backed by a NATS Key-Value bucket.

```
get = this.hostServices.http.get('https://example.org');
```
```
get
post
put
patch
del
head
```
## HTTP Client

## Key-Value Store


Again, note that you don't have to manually or explicitly provision the bucket. You can
assume that your function has its own, isolated bucket and that it is safe in a multi-tenant
environment.

Every Nex-managed function has access to a managed object store. The following code
shows how JavaScript can interact with this store:

JavaScript functions receive messages on their trigger subjects, which are specied at
deployment time. To publish or make requests, host services provides access to a core
NATS black box.

```
thisthis..hostServiceshostServices..kvkv.set.delete('hello'('hello', payload););
thisreturn.hostServices { .kv.set('hello2', payload);
keys hello2: this: this.hostServices.hostServices.kv..keyskv.get()(,'hello2')
}
```
```
(subject this.hostServices, payload) =>.objectStore { .put('hello', payload);
this.hostServices.objectStore.delete('hello');
thisreturn.hostServices { .objectStore.put('hello2', payload);
list hello2: this: String.hostServices.fromCharCode.objectStore(...this.list.hostServices(), .objectStore.get('hel
}};
```
## Object Store

## Core Messaging


this.hostServices.messaging.publish('hello.world', payload);
req = this.hostServices.messaging.request('hello.world.request', payload)
reqMany = this.hostServices.messaging.requestMany('hello.world.request.ma


# Nex Internals

This section of the documentation goes into detail on how Nex works, the various
components that work together in order to make it work, and the technology used at
every step.
While this information is certainly useful for application developers, it may be of even
more use to Nex operators and contributors.


# Architecture Overview

While the vast majority of the functionality of Nex can be found inside the single nex
binary, Nex is actually a suite of tools and components that all collaborate to give you the
power to deploy and manage workloads using your existing NATS infrastructure.

The following diagram provides an illustration of the Nex system architecture and how all
of the various componenents interoperate. It might seem a bit daunting at rst, so we'll
cover each part of the diagram in detail next.

Before we even get into discussing the various components, there's some context we
need to provide rst. Quite possibly one of the most important concepts to understand is
the distinction between the NATS system used by a workload and the NATS system used

```
architecture diagram
```
## Architecture Overview

## Workload NATS vs Control NATS


by Nex. It is crucial to understand this in order to properly (and securely) congure your
systems for use in production.

The connection(s) used by workloads are just that. If you're taking pre-existing services
and deploying them to Nex, then you'll know that these workloads need their own NATS
connections and, depending on the application, often have very specic connection and
security requirements.
In most applications like this, especially those inspired by the 12 factor guidelines, your
code gets its connection data from environment conguration. For example, it might pick
up the server URL(s) from the NATS_SERVER environment variable.
This connection stays the same whether the code is running within or outside of Nex.
The situation is easy for services because they rely on environment variables and, most
importantly, are allowed to create their own connections to NATS servers (within the
constraints of CNI conguration).
Functions play a more passive role in their NATS connection. Rather than creating their
own NATS connections, they rely on host services provided by the Nex node to
communicate with limited NATS resources such as key-value and object store buckets
and publication subjects. This connection is also dictated by the initial parameters to the
workload deployment.

All Nex nodes are remotely controllable. This is the key to the imperative-style control that
we've mentioned a few times. When a Nex node is brought up, it requires a NATS
connection. This connection is exclusively for command and control operations. As such,
when you're planning on how to structure accounts, signing keys, and operators, you
should keep this in mind. Even though Nex is quite secure, you still don't want untrusted
code to have unfettered access to control connections.

### Workload NATS

### Control NATS


All control operations take place on a topic namespace that starts with the $NEX prex,
making it easy to isolate and control access.

Unless yworkloads should neou're running in a test or dever overlap with connections used bvelopment environment, the connections used by the control interface. You don'ty (^)
necessarily need private servers, but you should be able to guarantee trac isolation.
There are a number of key Nex components contained in the preceding diagram, and
each is discussed in detail in this section.
There's yet another NATS connection in this diagram that we thought it best to leave until
the end of the discussion to avoid unnecessary confusion. This is the internal NATS
connection that is used by the Nex node and the agent to communicate.
This connection is not used by application code, nor is it used by remote control code. It
is only used internally and is only bound to an IP address usable on that local device. In
other words, even if you wanted to use this NATS connection remotely, you shouldn't be
able to.
In most cases, unless you're contributing to this part of the code, it's probably easiest to
ignore this connection.
Node Process
Nex Agent
Root File System
Control Interface

## Nex Components

## Internal (Hidden) NATS


# Node Process

The Nex node process is the core of the Nex ecosystem. It exposes a control interface
API over NATS, through which client applications can deploy and undeploy workloads as
well as interrogate the state of nodes and the workloads currently deployed.
The Nex node maintains a pool of Firecracker virtual machines. It is into this pool that
workloads are deployed. The reason we have a pool is that this allows us to have a
congurable number of "warm", ready-to-go virtual machines that are eagerly awaiting a
workload. This lets us deploy workloads incredibly fast.
In order to maintain this pool of Firecracker VMs, the Nex node needs to interact both
with the installed Firecracker application and with installed CNI (Container Networking
Initative) plugins, both of which can be downloaded and installed automatically by using
the nex CLI's preflight command.
Because the Nex node process spawns the firecracker process, which will in turn
attempt to use CNI plugins, this process almost always requires root privilege. When
developing locally it's easy to simply use sudo. However, in production, you may want to
create a special user context in which the Nex node can run.
In isolation, a single Nex node is capable of running hundreds of functions and services.
When you strategically place Nex nodes throughout your NATS infrastructure, you create
a unied execution cluster capable of scaling to meet virtually any demand.


# Nex Agent

The **Nex Agent** is responsible for managing only one workload. Through the agent's API,
the node can request that a workload be deployed, undeployed, or (for functions)
triggered.
Internal to the agent is the logic that determines how the workload is managed. We have
a provider system in the Go code that makes it easy for us to expand and enhance the
types of workloads we support.
Workloads are currently handled as follows:

All function-type workloads must rely on host services in order to interact with managed
resources like key-value buckets, publication/request, object stores, and more.

The agent process is the nex-agent binary produced by this Go code. This binary is not
executed directly by the Node process, nor is it ever launched by developers or users.
The agent resides within the root le system. When you launch a Firecracker virtual
machine, it isn't quite like issuing a Docker run command. Launching a Firecracker VM
is like booting an operating system. In order to tell a Linux operating system what
processes to start when launched (e.g. when the Firecracker VM boots), we need an init
system.

```
elf workload deplo (64-bit Linux) yment. The envirservice - This binaronment vy is executed as a child prariables from the deploocess of the agent upony request as passed
to the binary after decryption.
JavaScript congured set of stimuli dened in the deplo function - This function is deployed idle and then triggeryment request. For example, yed in response tou can o a
dene a set of subjects and wildcarfunction. ds that will be used to trigger the JavaScript
WebAssembly congured stimuli (declar function - This function is deploed the same way as with all function types).yed idle and then trigered in response to
```
## Agent Startup


Init systems can be confusing and intimidating. When you distill it down to the core, the
init process in Linux is just the rst process started during boot. Depending on which
application you use for init, you congure your startup services and other boot-time
launch activity differently.
For reasons that we won't get into here, it's not a good idea to make a process like
nex-agent be the init process. Rather, we want the init process to spawn and manage
the nex-agent.
To do this, we're using OpenRC. The default OpenRC conguration used for the agent is
as follows:

This script denes the basic properties of our OpenRC service. The service, "Nex Agent",
runs as the user nex within the group nex and the executable le is
/usr/local/bin/agent. It's also important to note that this service cannot be started
until after the net.eth0 device has been initialized.
For more information on how the agent is physically placed into the root le system,
continue on to the next section where we cover the root le system.

```
#!/sbin/openrc-run
name=$RC_SVCNAMEdescription="Nex Agent"
supervisor="supervise-daemon"command="/usr/local/bin/agent"
pidfile="/run/agent.pid"command_user="nex:nex"
output_log="/home/nex/nex.log"error_log="/home/nex/err.log"
depend() {after net.eth0
}
```

# No Sandbox Mode

Firecracker provides Nex operators with the condence of being able to safely and
securely run untrusted workloads. However, when developers are iterating over their
application code and they want to deploy via Nex during that loop, requiring Firecracker
can get in the way. Firecracker will only work on 64-bit Linux machines that have certain
virtualization options enabled in the kernel.
Not only does this limit the developer workstations capable of using the sandbox, but
there are a number of cloud virtual machine types that do not support this kind of
virtualization.
To make this easier during development, and to allow Nex to manage workloads at the
edge, on small devices, or even on Windows, you can run Nex in "no sandbox" mode.
The following is a conguration le that shows the use of the no_sandbox eld. It might
look a bit awkward to set no_sandbox to true, but this naming convention was chosen
specically to make it nearly impossible to accidentally launch a Nex node without a
sandbox.
**Example Conguration File**

## Running workloads without Firecracker


**NEX Agent**
When running in no_sandbox mode, the nex node will run workloads directly on your
machine. To do this, it spawns the nex-agent as a child process. To make sure the nex
node can nd the agent, you will need to ensure it is placed somewhere in your PATH.

We strongly recommend the use of the Firecracker sandbox when running in production
and suggest that unsafe mode should only be reserved for development and testing
environments or those environments (e.g. macOS, Windows, edge devices) incapable of
running Firecracker.

```
{ "kernel_filepath": "/path/to/vmlinux-5.10",
"rootfs_filepath""machine_pool_size": "/path/to/rootfs.ext4": 1 , ,
"cni""network_name": { : "fcnet",
```
(^) }, "interface_name": "veth0"
"machine_template""vcpu_count": : 1 {,
(^) }, "memsize_mib": 256
"tags""simple": { : "true"
} "no_sandbox", : true
}

## Production Use


# Root File System

The root le system used by Nex in its spawned Firecracker virtual machines is an ext4
(64-bit le system) block device. In oversimplied terms, it's basically a single le that
represents an entire le system.

As of April 2024, the Nex CLI has the ability to build a root lesystem (a .ext4 le). To
build a root lesystem, run the following command:

You will need to include the --agent ag at a minimum.
Keep in mind that you will need to make the rootfs large enough to hold any binary you'll
later run via nex run. The default size is 150MB and this tends to support a ~20MB
binary.

There are countless ways to populate an ext4 le, from programmatic to scripted. While
our current CI pipelines are more programmatic than scripted, the same underlying
principles still apply.
To build a root le system:

#### └─usage:❯ nex nex rootfs rootfs --help [<flags>]

```
Build custom rootfs
Flags: --script=script.sh Additional boot script ran during
--image --agent=="synadia/nex-rootfs:alpine"../path/to/nex-agent BasePath imageto agent for binaryrootfs build
--size= 157286400 Size of rootfs filesystem
```
## Building a Root File System

**Using nex CLI**

**Manual Approach**


1. Create an empty rootfs.ext4 le of a given size with empty blocks
2. Use the mkfs.ext4 utility to convert the block device into an ext4 le system
3. Fill in the les in the le system as needed.
An unexpected but incredibly useful trick is that we can use Docker for step 3. We can
mount the block device as a folder and then map that folder to a folder inside the Docker
image. If we run the setup script inside the Docker image and then unmount the le
system, our rootfs.ext4 le will be a snapshot of what the Docker image looked like
when it nished.
Here's a sample script that does just that:

Here we're using the public alpline Docker image to run a script, setup-alpine.sh
that will modify the le system to build what we're looking for. Note that we've actually
mounted the openrc-service.sh script to /etc/init.d/agent. This effectively
copies this le into the new root le system, setting up our OpenRC service.
Let's see what setup-alpine.sh might look like:

```
#!/bin/bash
set -xe
ddmkfs.ext4 if=/dev/zero rootfs.ext4 of=rootfs.ext4 bs= 1 M count= 100
mkdirmount -prootfs.ext4 /tmp/my-rootfs /tmp/my-rootfs
docker -v run/tmp/my-rootfs:/my-rootfs -i --rm \ \
-v-v "$("$(pwdpwd)/nex-agent/agent:/usr/local/bin/agent")/openrc-service.sh:/etc/init.d/agent" \ \
alpine sh <setup-alpine.sh
umount /tmp/my-rootfs
```

This script adds openrc and util-linux to to the bare alpine image, and then uses
rc-update to add the agent script to the boot phase.
We currently use a combination of code and scripts to automatically generate a vetted
root le system that can be automatically downloaded via the nex preflight
command.

```
#!/bin/sh
set -xe
apkapk addadd --no-cache--no-cache openrcutil-linux
```
lnecho -s ttyS0agetty > (^) /etc/securetty/etc/init.d/agetty.ttyS0
rc-update add agetty.ttyS0 default
echo "root:root" | chpasswd
echo "nameserver 1.1.1.1" >>/etc/resolv.conf
addgroup -g 1000 -S nex && adduser -u 1000 -S nex -G nex
rc-updaterc-update addadd devfsprocfs boot boot
rc-update add sysfs boot
# This is our script that runs nex-agentrc-update add agent boot
forfor d dir in bin etc lib root sbin usr; in dev proc run sys var tmp; dodo tar mkdir c "/$d" /my-rootfs/ | tar x${dir}; -C /my-rootfsdone ;
chmodmkdir (^1777) -p /my-rootfs/home/nex/ /my-rootfs/tmp
chown 1000 :1000 /my-rootfs/home/nex/


# Control Interface

The Nex control interface is an API through which you can remotely control Nex nodes.
This API provides the ability to ping/discover all nodes, interrogate node and workload
state, as well as deploy and undeploy workloads.
This API is currently undergoing rapid changes and so the code is the best resource at
the moment. Once the API stabilizes, we will have more concrete documentation in place.


# FAQ

Frequently Asked Questions about the NATS Execution Engine

Nex is an open source, lightweight execution engine that runs alongside NATS. Nex
nodes are run to form a universal pool in which you can deploy services and functions in

```
What is Nex?
Where can I nd Nex?
Who maintains Nex?
Is Nex free?
```
```
What workload types can I run?
What is a root le system?
```
```
What are namespaces?
Are my workload congurations secure?
What is the difference between run and devrun?
How does Nex compare to Kubernetes?
Do I run Nex inside Firecracker?
```
## General

## Running Workloads

## Technical Details

## General

## What is Nex?


JavaScript, WebAssembly, and even regular operating system executable binaries.

The code for Nex can be found in the appropriate Synadia repository. It is an open source
project that is supported by Synadia, the creators of NATS.

The primary maintainers of the Nex code are employees of Synadia, though more
maintainers are always welcome.

Yes, all of the code in the Nex repository is free and licensed under the **Apache-2.0**
license.

Nex supports two categories of workloads: **services** and **functions**. Services are long-
lived while functions are short-lived and triggered on demand by some stimulus.
Services can be native, compiled binaries (64-bit **elf** when running with Firecracker
enabled) and functions can be either a **WebAssembly** module or a **JavaScript** exported
function. OCI image support is coming soon.

### Where Can I Find Nex?

### Who Maintains Nex?

### Is Nex Free?

## Running Workloads

### What workload types can I run?

### What is a root file system?


By default, Nex uses Firecracker to launch small, secure, fast, and light weight virtual
machines. These machines each run an agent that manages the workload in that
machine as well as communications with the Nex node.
The root le system is essentially a snapshot of the operating system disk image that will
be used when bringing up the virtual machine. It's like a Docker layer, but much more
ecient.

A namespace is a unit of multi-tenancy. All workloads are scoped to a namespace. If you
deploy the echoservice workload within the default namespace and also deploy it
within the sample namespace, they will not be treated the same by Nex. Namespaces
are logical groupings and Nex doesn't enforce a hard network boundary between them or
enforce opinions about how you use namespaces.

Absolutely! Every request to run a workload is sent to a specic Nex node. This Nex node
has a public **Xkey** that is used to encrypt data, such as environment variables and other
sensitive data, sent to that node. Only that node can decrypt data meant for it.
Further, only the entity that started a workload can terminate that workload.

In short, nex run is meant for production and real deployments while nex devrun is
meant for developer iteration loops and testing environments. A regular request to run a
workload must contain all of the following:
An issuer **nkey** that counts as the entity certifying the request

## Technical Details

### What are namespaces?

### Are my workload configurations secure?

### What is the difference between run and devrun?


Manually supplying all this information when you're just trying to develop and test on your
local machine is cumbersome, so using devrun all you need do is supply a path to the
workload binary le (.wasm, .js, ELF binary) and the environment variables, and the
nex CLI will take care of managing the rest for you, including uploading your le to an
object store automatically.

Kubernetes is a distributed workload execution engine, as is Nex. This is where their
paths diverge. Nex is able to provide an ecient, tailored user experience because it is
opinionated and optimized for running NATS-based workloads, where Kubernetes is
designed to be so generalized that it can do practically anything given enough time or
money. This broad set of uses can make Kubernetes dicult to manage, deploy, and
congure.
While most users interact with a declarative layer in Kubernetes that is managed an
enormous amount of YAML, Nex is a purely imperative system easily managed via CLI
and a simple API.

In a word, no. The Nex node process is responsible for spawning Firecracker VMs that
contain the Nex agent. As a supervisor of Firecracker processes, the Nex node itself can't
be run inside a Firecracker VM. Most of that complexity should be hidden from end users.

```
A publisher xkey that is used as the source of encrypted data
A URL indicating the location of the workload that is stform of nats://{BUCKET}/{key}. ored in an object store, in the
A set of environment variables to be passed to the workload when it is run
A x ed and properly formatted workload name
A namespace (logical grouping) in which the workload is to be run.
```
### How does Nex Compare to Kubernetes?

### Do I run Nex inside Firecracker?


