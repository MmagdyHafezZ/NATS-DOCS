# Running Workloads on NATS

The NATS Execution Engine (Nex) is an optional add-on to NATS that overlays your existing NATS infrastructure, giving you the ability to deploy and run workloads. Building applications that rely on NATS as a backbone already provides a lot of power and features out of the box, including message communications, global reach, reliability, and additional services such as streams, key-value stores, and object stores. However, prior to the availability of Nex, we couldn't use that infrastructure as a deployment target.

**Note**: The NATS execution engine is currently experimental. The documentation, feature set, and APIs are likely to change frequently.

## Nex Workloads: Services and Functions

Nex supports two fundamental types of workloads: **services** and **functions**.

### Services

In the context of Nex, a service is a long-running process. When it is deployed, it is launched with a given environment and runs continually until terminated. Typical examples of service-type workloads are web servers, monoliths, API endpoints, and applications that maintain long-running subscriptions to NATS subjects. The default, secure Nex services are statically-linked, 64-bit Linux (ELF) binaries, but you can also run native Mac or Windows binaries as services, and you can customize the root file system used for the Firecracker sandbox. Support for OCI (e.g., Docker) images is coming soon.

### Functions

Nex functions are short-lived processes executed in response to some trigger, where the functions can process the trigger's data and optionally supply a return value. Nex functions are small and can be deployed either as JavaScript functions or as WebAssembly modules.

# Getting Started

This getting started guide will help you deploy workloads with Nex, including setup, installation, and developing and deploying both services and functions.

**Note**: If you intend to use the secure sandbox mode, make sure you're running on a 64-bit Linux machine with virtualization enabled in the kernel. Nex's default secure mode makes extensive use of Firecracker, which has strict environmental requirements.

# Installing Nex

All of the functionality you need for Nex is conveniently wrapped up in a single command-line tool. To install it, enter the following command in a terminal:

```sh
curl -sSf https://nex.synadia.com/install.sh | sh
```

Depending on your operating system and user privileges, you may need to change `sh` to `sudo sh` so the script can place the nex binary in your path. If you're not comfortable running this command, you can manually install Nex by downloading the latest version from the releases page and placing the nex binary somewhere in your path.

**Note**: While the nex binary can be run on any operating system, some functionality may only be available on 64-bit Linux due to requirements dictated by Firecracker. Running Linux inside Docker won't satisfy those requirements.

After installation, you should be able to check the CLI version with `nex version`. If you get the help banner and version, you can proceed to the next step.

# Performing the Preflight Check

Starting a Nex node can involve using the Linux kernel, the Firecracker binary, CNI configuration files, an ext4 root file system, machine configuration, and other dependencies. Nex provides a preflight check to ensure everything is set up correctly.

## Creating a Node Configuration

The easiest way to create a node configuration file is to copy one from the Nex examples folder, such as the simple.json file:

```json
{
  "default_resource_dir": "/tmp/wd",
  "machine_pool_size": 1,
  "cni": {
    "network_name": "fcnet",
    "interface_name": "veth0"
  },
  "machine_template": {
    "vcpu_count": 1,
    "memsize_mib": 256
  },
  "tags": {
    "simple": "true"
  }
}
```

This configuration file will look for a Linux kernel file (vmlinux) and a root file system (rootfs.ext4) in the default resource directory. You can override these by supplying the `kernel_file` or `rootfs_file` fields. Place this configuration file anywhere you like, but `preflight` will check `./config.json` by default. For each dependency `preflight` doesn't find, it can create default configuration files and download missing dependencies such as the Firecracker binary, a Linux kernel, and a vetted root file system.

After running `preflight` and downloading all missing components, run it again to ensure all checklist items are satisfied.

```sh
$ nex node preflight --config=../examples/nodeconfigs/simple.json
Validating - Required CNI Plugins [/opt/cni/bin]
✅ Dependency Satisfied - /opt/cni/bin/host-local [host-local CNI plugin]
✅ Dependency Satisfied - /opt/cni/bin/ptp [ptp CNI plugin]
✅ Dependency Satisfied - /opt/cni/bin/tc-redirect-tap [tc-redirect-tap CNI plugin]
Validating - Required binaries [/usr/local/bin]
✅ Dependency Satisfied - /usr/local/bin/firecracker [Firecracker VM]
Validating - CNI configuration requirements [/etc/cni/conf.d]
✅ Dependency Satisfied - /etc/cni/conf.d/fcnet.conflist [CNI Configuration]
Validating - User provided files []
✅ Dependency Satisfied - /tmp/wd/vmlinux [VMLinux Kernel]
✅ Dependency Satisfied - /tmp/wd/rootfs.ext4 [Root Filesystem Template]
```

# Building a Service

Building a service for deployment via Nex is like building any other service. There's no proprietary SDK, custom build tools, or complicated dependency tree. If you're building cloud-native or 12-factor services as single-binary deployments, you should have no trouble running them in Nex.

## Creating a NATS Service

For this example, we'll use Go to create a simple executable and use the NATS services framework to expose an endpoint for service discovery. Create a `main.go` file with the following content:

```go
package main

import (
  "context"
  "fmt"
  "os"
  "strings"

  "github.com/nats-io/nats.go"
  services "github.com/nats-io/nats.go/micro"
)

func main() {
  ctx := context.Background()
  natsUrl := os.Getenv("NATS_URL")
  if len(strings.TrimSpace(natsUrl)) == 0 {
    natsUrl = nats.DefaultURL
  }
  fmt.Printf("Echo service using NATS url '%s'\n", natsUrl)

  nc, err := nats.Connect(natsUrl)
  if err != nil {
    panic(err)
  }

  echoHandler := func(req services.Request) {
    req.Respond(req.Data())
  }

  fmt.Println("Starting echo service")
  _, err = services.AddService(nc, &services.Config{
    Name:    "EchoService",
    Version: "1.0.0",
    Endpoint: &services.EndpointConfig{
      Subject: "svc.echo",
      Handler: services.HandlerFunc(echoHandler),
    },
  })
  if err != nil {
    panic(err)
  }

  <-ctx.Done()
}
```

You'll need the Go NATS client SDK, which you can get by running:

```sh
$ go get github.com/nats-io/nats.go
```

This code establishes a connection to a NATS server as indicated by the `NATS_URL` environment variable and creates a discoverable service called `EchoService` with an endpoint on the `svc.echo` subject. The handler will respond to requests on that subject.

## Running the Service Locally

Run this service without using Nex:

```sh
$ go run main.go
```

In another terminal, make a request on the `svc.echo` subject:

```sh
$ nats req svc.echo 'this is a test'
17:10:53 Sending request on "svc.echo"
17:10:53 Received with rtt 567.537μs this is a test
```

You can also check the service status:

```sh
$ nats micro ls
╭─────────────────────────────────────────────
│ All Micro Services
├─────────────┬─────────┬─────────────────────
│ Name        │ Version │ ID                  │ Description
╰─────────────┴─────────┴─────────────────────
│ EchoService │ 1.0.0   │ FKIuiivKgSB8VWyjDYBpEc │
```

## Compiling the Service for Nex

To deploy via Nex, compile the service as a statically linked executable:

```sh
$ CGO_ENABLED=0 go build -tags netgo -ldflags '-extldflags "-static"'
```

Verify the binary:

```sh
$ file echoservice
echoservice: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked
```

With our statically compiled service, we can start deploying it.

# Starting a Nex Node

If the preflight checks passed, start a Nex node:

```sh
$ sudo ./nex node up --config=/home/kevin/simple.json
INFO[0000] Established node NATS connection to: nats://127.0.0.1:4222
INFO[0000] Loaded node configuration from '/home/kevin/simple.json'
INFO[0000] Virtual machine manager starting
INFO[0000] Internal NATS server started client_url="nats://127.0.0.1:4222"
INFO[0000] Use this key as the recipient for encrypted run requests
INFO[0000] NATS execution engine awaiting commands id=NCOBPU3MCEA7L
INFO[0000] Called startVMM

(), setting up a VMM on /tmp/.firecracker.sock
INFO[0000] VMM metrics disabled
INFO[0000] refreshMachineConfiguration: [GET /machine-config][200] getMachineConfigOK
INFO[0000] PutGuestBootSource: [PUT /boot-source][204] putGuestBootSourceNoContent
INFO[0000] Attaching drive /tmp/rootfs-cmjg61n52omq8dovolmg.ext4, slot 1
INFO[0000] Attached drive /tmp/rootfs-cmjg61n52omq8dovolmg.ext4: [PUT /drives/drive_id][204] putGuestDriveByIdNoContent
INFO[0000] Attaching NIC tap0 (hwaddr 5a:65:8e:fa:7f:25) at index 1
INFO[0000] startInstance successful: [PUT /actions][204] createSyncActionNoContent
INFO[0000] SetMetadata successful
INFO[0000] Machine started gateway=192.168.127.1
INFO[0000] Adding new VM to warm pool ip=192.168.127.2
INFO[0001] Received agent handshake message="Host-supervisor started"
```

This log output indicates a successful node startup. The node established a connection to NATS on `127.0.0.1:4222`. This is the control plane connection, not to be confused with the NATS connection used by individual workloads.

## Verifying Node Readiness

Open another terminal and list the nodes:

```sh
$ nex node ls
╭─────────────────────────────────────────────
│ NATS Execution Nodes
├─────────────┬─────────┬─────────────────────
│ ID          │ Version │ Uptime
╰─────────────┴─────────┴─────────────────────
│ NCOBPU3MCEA7L │ 0.0.1   │ 1m
```

# Deploying Services

There are multiple ways to deploy a service with Nex. For simplicity, we'll use the `devrun` command.

Ensure your node is discoverable:

```sh
$ nex node ls
╭─────────────────────────────────────────────
│ NATS Execution Nodes
├─────────────┬─────────┬─────────────────────
│ ID          │ Version │ Uptime
╰─────────────┴─────────┴─────────────────────
│ NCOBPU3MCEA7L │ 0.0.1   │ 1m
```

Deploy the echo service with the `devrun` command:

```sh
$ nex devrun ./echoservice NATS_URL=nats://192.168.127.1:4222
Reusing existing issuer account key: /home/kevin/.nex/issuer.nk
Reusing existing publisher xkey: /home/kevin/.nex/publisher.xk
🚀 Workload 'echoservice' accepted. You can now refer to this workload with its ID.
```

Check the workload status:

```sh
$ nats micro ls
╭─────────────────────────────────────────────
│ All Micro Services
├─────────────┬─────────┬─────────────────────
│ Name        │ Version │ ID                  │ Description
╰─────────────┴─────────┴─────────────────────
│ EchoService │ 1.0.0   │ NsMaTbN7u5ZPUNN47bSEI6 │
```

Test the service:

```sh
$ nats req svc.echo 'hey'
19:40:22 Sending request on "svc.echo"
19:40:22 Received with rtt 446.27μs hey
```

Interrogate the node to see the workload:

```sh
$ nex node info NCOBPU3MCEA7L
NEX Node Information
Node: NCOBPU3MCEA7L
Xkey: XASQSWNSIKHM5MDKDOGPSPGBA3V6JMETMIJK2YTXKAJZNMAFKXER5RUK
Version: 0.0.1
Uptime: 8m47s
Tags: nex.arch=amd64, nex.cpucount=8, nex.os=linux, simple=true
Memory in kB:
Free: 33,545,884 Available: 56,529,348
Total: 63,883,232
Workloads:
Id: cmji29n52omrb71g07a0 Healthy: true
Runtime: 8m47s Name: echoservice
Description: Workload published in devmode
```

Congratulations, you have a running Nex node ready to accept and run any kind of workload!

# Building a Function

Nex functions are similar to cloud functions or "lambdas". They are short-lived and are executed in response to some external trigger.

There are two kinds of Nex functions: **JavaScript** and **WebAssembly**.

## Writing a JavaScript Function

Writing a JavaScript function is straightforward. Here's a simple echo function:

```js
(subject, payload) => {
  console.log(subject);
  return payload;
};
```

Deploy the JavaScript function:

```sh
$ nex devrun /path/to/echo.js --trigger_subject=js.echo
Reusing existing issuer account key: /home/kevin/.nex/issuer.nk
Reusing existing publisher xkey: /home/kevin/.nex/publisher.xk
🚀 Workload 'echofunctionjs' accepted. You can now refer to this workload with its ID.
```

Test the function:

```sh
$ nats req js.echo 'heya'
09:40:33 Sending request on "js.echo"
09:40:33 Received with rtt 2.600724ms "heya"
```

## Writing a WebAssembly Function

Nex-compatible WebAssembly functions follow the command pattern using WASI. Here's a simple Rust WebAssembly function:

```rust
use std::{env, io::{self, Read, Write}};

fn main() {
  let args: Vec<String> = env::args().collect();

  // When a WASI trigger executes:
  // argv[1] is the subject on which it was triggered
  // stdin bytes is the raw input payload
  // stdout bytes is the raw output payload

  let mut buf = Vec::new();
  io::stdin().read_to_end(&mut buf).unwrap();
  
  let mut subject = args[1].as_bytes().to_vec();
  buf.append(&mut subject);
  
  // This returns the payload concatenated with the subject
  io::stdout().write_all(&mut buf).unwrap();
}
```

Build the function into a module:

```sh
$ cargo build --target wasm32-wasi --release
```

Deploy the WebAssembly function:

```sh
$ nex devrun ../examples/wasm/echofunction/echofunction.wasm --trigger_subject=wasm.echo
Reusing existing issuer account key: /home/kevin/.nex/issuer.nk
Reusing existing publisher xkey: /home/kevin/.nex/publisher.xk
🚀 Workload 'echofunctionwasm' accepted. You can now refer to this workload with its ID.
```

Test the function:

```sh
$ nats req wasm.echo 'hello'
09:45:24 Sending request on "wasm.echo"
09:45:24 Received with rtt 42.867014ms "hellowasm.echo"
```

Interrogate the node to see both function workloads:

```sh
$ nex node info NCOBPU3MCEA7L
NEX Node Information
Node: NCOBPU3MCEA7L
Xkey: XASQSWNSIKHM5MDKDOGPSPGBA3V6JMETMIJK2YTXKAJZNMAFKXER5RUK
Version: 0.0.1
Uptime: 7m31s
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

Nex provides access to managed resources such as key-value buckets, messaging subject spaces, object stores, and HTTP clients.

## HTTP Client

JavaScript functions can make HTTP calls:

```js
get = this.hostServices.http.get('https://example.org');
```

Supported methods include:

- `get`
- `post`
- `put`
- `patch`
- `del`
- `head`

## Key-Value Store

Access a key-value store:

```js
this.hostServices.kv.set('hello', payload);
this.hostServices.kv.delete('hello');
this.hostServices.kv.get('hello2');
```

## Object Store

Interact with an object store:

```js
this.hostServices.objectStore.put('hello', payload);
this.hostServices.objectStore.delete('hello');
this.hostServices.objectStore.list();
this.hostServices.objectStore.get('hello2');
```

## Core Messaging

Publish and request messages:

```js
this.hostServices.messaging.publish('hello.world', payload);
req = this.hostServices

.messaging.request('hello.world.request', payload);
reqMany = this.hostServices.messaging.requestMany('hello.world.request.many', payload);
```

# Nex Internals

This section details how Nex works, its components, and the technology used at every step.

## Architecture Overview

Nex is a suite of tools and components that work together to deploy and manage workloads using your existing NATS infrastructure. The diagram below illustrates the Nex system architecture.

## Workload NATS vs Control NATS

The NATS connection(s) used by workloads are separate from the NATS connection used by Nex for command and control operations.

### Workload NATS

Workloads use their own NATS connections, typically configured via environment variables such as `NATS_SERVER`.

### Control NATS

Nex nodes use a separate NATS connection for command and control operations. This connection is isolated from the workload NATS connections.

## Nex Components

Key Nex components include:

- Node Process
- Nex Agent
- Root File System
- Control Interface

### Node Process

The Nex node process maintains a pool of Firecracker virtual machines and exposes a control interface API over NATS.

### Nex Agent

The Nex Agent manages individual workloads and relies on host services to interact with managed resources.

### Root File System

The root file system used by Nex is an ext4 block device, representing the entire file system.

### Control Interface

The Nex control interface is an API for remotely controlling Nex nodes, including deploying and undeploying workloads and interrogating node and workload state.

# FAQ

## General

### What is Nex?

Nex is an open-source, lightweight execution engine that runs alongside NATS, allowing you to deploy services and functions in JavaScript, WebAssembly, and regular operating system executable binaries.

### Where can I find Nex?

The code for Nex is in the Synadia repository and is supported by Synadia, the creators of NATS.

### Who maintains Nex?

Nex is maintained by Synadia employees, with more maintainers always welcome.

### Is Nex free?

Yes, Nex is free and licensed under the Apache-2.0 license.

## Running Workloads

### What workload types can I run?

Nex supports two categories of workloads: **services** and **functions**. Services are long-lived, while functions are short-lived and triggered on demand. Services can be native, compiled binaries (64-bit ELF), and functions can be WebAssembly modules or JavaScript exported functions. OCI image support is coming soon.

### What is a root file system?

The root file system is a snapshot of the operating system disk image used when bringing up the virtual machine, similar to a Docker layer but more efficient.

### What are namespaces?

A namespace is a unit of multi-tenancy. All workloads are scoped to a namespace. If you deploy the `echoservice` workload within the `default` namespace and also deploy it within the `sample` namespace, they will not be treated the same by Nex. Namespaces are logical groupings, and Nex doesn't enforce a hard network boundary between them or enforce opinions about how you use namespaces.

### Are my workload configurations secure?

Absolutely! Every request to run a workload is sent to a specific Nex node. This Nex node has a public **Xkey** that is used to encrypt data, such as environment variables and other sensitive data, sent to that node. Only that node can decrypt data meant for it. Furthermore, only the entity that started a workload can terminate that workload.

### What is the difference between `run` and `devrun`?

In short, `nex run` is meant for production and real deployments, while `nex devrun` is meant for developer iteration loops and testing environments. A regular request to run a workload must contain all of the following:
- An issuer **nkey** that certifies the request.
- A publisher **xkey** for the source of encrypted data.
- A URL indicating the location of the workload stored in an object store, in the form of `nats://{BUCKET}/{key}`.
- A set of environment variables to be passed to the workload when it runs.
- A fixed and properly formatted workload name.
- A namespace in which the workload is to run.

Manually supplying all this information when you're just trying to develop and test on your local machine is cumbersome. With `devrun`, you only need to supply a path to the workload binary file (.wasm, .js, ELF binary) and the environment variables. The `nex` CLI will take care of managing the rest, including uploading your file to an object store automatically.

### How does Nex compare to Kubernetes?

Kubernetes is a distributed workload execution engine, as is Nex. However, Nex is opinionated and optimized for running NATS-based workloads, whereas Kubernetes is designed to be highly generalized, capable of running a wide range of workloads given enough configuration. This generality can make Kubernetes complex to manage and deploy. Nex provides a more tailored and efficient user experience for NATS-based workloads, with a purely imperative system easily managed via CLI and a simple API.

### Do I run Nex inside Firecracker?

No, the Nex node process is responsible for spawning Firecracker VMs that contain the Nex agent. As a supervisor of Firecracker processes, the Nex node itself cannot be run inside a Firecracker VM. Most of this complexity is hidden from end users.

## Technical Details

### How does Nex ensure workload isolation?

Nex uses Firecracker to launch small, secure, fast, and lightweight virtual machines. These machines each run an agent that manages the workload in that machine as well as communications with the Nex node. The root file system is essentially a snapshot of the operating system disk image used when bringing up the virtual machine, similar to a Docker layer but more efficient.

### Can I run Nex without Firecracker?

Yes, Nex can run in "no sandbox" mode for development and testing environments or on platforms (e.g., macOS, Windows, edge devices) that cannot support Firecracker. In this mode, Nex runs workloads directly on your machine by spawning the `nex-agent` as a child process. This mode is not recommended for production.

### How do I create a root file system for Nex?

The root file system used by Nex is an ext4 block device. To build a root file system, you can use the `nex rootfs` command or manually create an ext4 file. Here’s a sample script using Docker to set up the file system:

```bash
#!/bin/bash
set -xe

# Create an empty ext4 file
dd if=/dev/zero of=rootfs.ext4 bs=1M count=100

# Make it an ext4 filesystem
mkfs.ext4 rootfs.ext4

# Mount it
mkdir -p /tmp/my-rootfs
mount -o loop rootfs.ext4 /tmp/my-rootfs

# Use Docker to populate the filesystem
docker run --rm -v "$(pwd)/nex-agent/agent:/usr/local/bin/agent" -v /tmp/my-rootfs:/my-rootfs alpine sh <<EOF
apk add --no-cache openrc util-linux
echo 'root:root' | chpasswd
rc-update add devfs boot
rc-update add procfs boot
rc-update add sysfs boot
rc-update add agetty.ttyS0 default
echo 'nameserver 1.1.1.1' >> /etc/resolv.conf
addgroup -g 1000 -S nex && adduser -u 1000 -S nex -G nex
mkdir -p /my-rootfs/home/nex /my-rootfs/tmp
chmod 1777 /my-rootfs/tmp
EOF

# Unmount the filesystem
umount /tmp/my-rootfs
```

This script creates an ext4 file, mounts it, and uses Docker to populate it with the necessary files and configurations.

# Conclusion

Congratulations! You've now used Nex to deploy full services compiled as static binaries, JavaScript functions, and WebAssembly functions. Deploying your applications as a combination of services and functions with Nex is fast, easy, and sets you up to joyfully deploy distributed applications.

By leveraging Nex, you can integrate seamlessly with your existing NATS infrastructure, ensuring secure, efficient, and scalable workload deployments. Whether you're building microservices, handling ephemeral tasks, or processing data streams, Nex provides a robust platform to meet your needs. 

Stay tuned to the Nex documentation, blog, and social media outlets for updates and new features. Happy coding!