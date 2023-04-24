# Deploying KIND Using Podman on MacOS

***Tom Dean - 1/23/23***

###### tags: `MacOS`, `Docker`, `Podman`, `KIND`, `Apple`, `Silicon`, `Intel`, `Homebrew`, `Kubernetes`, `K8s`

## Introduction

Sometimes you just want a lightweight Kubernetes cluster.  You might want to brush up on your Kubernetes skills in preparation for a CNCF exam, prototype and test some code, or one of a million other possible scenarios.  Standing up a Kubernetes cluster, even with automation and the Cloud, can be slow, cumbersome and overkill for many of your K8s needs.

Modern Macs, even the Apple Silicon varients, are a great platform to work with KIND clusters.  KIND even has Rootless support, using Podman, allowing you to run an entire KIND cluster without requiring elevated permissions for users.

***Let's see how we set it up!***

## References

[KIND: Kubernetes in Docker](https://kind.sigs.k8s.io)
[KIND: Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)
[Rootless KIND](https://kind.sigs.k8s.io/docs/user/rootless/)
[Homebrew](https://brew.sh)

## Prerequisites

Most of the required software will be installed using ***Homebrew***.  If you don't already have Homebrew installed, you can install it using:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

More information on Homebrew is available via the link in the **References** section above.

### Install Podman

You will need ***Podman*** installed on your Mac.  If you don't already have Podman installed on your Mac, you can install it using Homebrew:

Install Podman:

```bash
brew update
brew install podman
```

If you'd like the GUI components to go with the CLI bits, install Podman Desktop:

```bash
brew install --cask podman-desktop
```

### Quick Test: Podman

Let's make sure Podman is working:
```bash
podman run hello-world
```

We should see something like the following:
```bash title=Output
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull quay.io/podman/hello:latest...
Getting image source signatures
Copying blob sha256:4fb94855f81a5f7ed2673bde6da8255c2acb758d24d42172c6d728deecf8bfe1
Copying config sha256:7adcf4906402b74cfb97b84d57f0920dcbc29203dfa7c0a4c2789e94df7b647b
Writing manifest to image destination
Storing signatures
!... Hello Podman World ...!

         .--"--.           
       / -     - \         
      / (O)   (O) \        
   ~~~| -=(,Y,)=- |         
    .---. /`  \   |~~      
 ~/  o  o \~~~~.----. ~~   
  | =(X)= |~  / (O (O) \   
   ~~~~~~~  ~| =(Y_)=-  |   
  ~~~~    ~~~|   U      |~~ 

Project:   https://github.com/containers/podman
Website:   https://podman.io
Documents: https://docs.podman.io
Twitter:   @Podman_io
```

### Install `go`

In order to install and use KIND, you will need to have `go` installed.  If you don't already have `go` installed, the most direct way to install it is using Homebrew:

```bash
brew update
brew install go
```

### Quick Test: `go`

Let's verify our `go` installation:
```bash
go version
```

We should see output like the following:
```bash
go version go1.19.5 darwin/amd64
```

Your version of `go` may be different, just make sure it's `1.17+`.

### Install `kubectl`

We're going to need a way to talk to the API of our KIND cluster, and `kubectl` is it.  If you don't already have `kubectl` installed, you can install it using Homebrew:
```bash
brew install kubernetes-cli
```

Checking our work:
```bash
kubectl version --output=yaml
```

We should see something similar to the following:
```bash
clientVersion:
  buildDate: "2023-01-18T15:51:24Z"
  compiler: gc
  gitCommit: 8f94681cd294aa8cfd3407b8191f6c70214973a4
  gitTreeState: clean
  gitVersion: v1.26.1
  goVersion: go1.19.5
  major: "1"
  minor: "26"
  platform: darwin/amd64
kustomizeVersion: v4.5.7
```

*Remember, versions and architectures can vary based on when you install and what you install it on!*

Our `kubectl` command is installed and ready to go!

### Configure Shell

Before we start working with KIND, we're going to want to set a couple of things in our shell environment to make working with KIND easier.

Recent versions of MacOS use ZSH as the default shell, so we're going to make our settings in the `.zshrc` file.  If you're using BASH, use the `.bashrc` file.

Add the following to your `.zshrc`/`.bashrc` file:

```bash
alias docker=podman
PATH=$PATH:~/go/bin
KIND_EXPERIMENTAL_PROVIDER=podman
```

Next, we'll source the file we edited to pick up the changes.

For ZSH:
```bash
source .zshrc
```

For BASH:
```bash
source .bashrc
```

***Now we're ready to roll up our sleeves and get started with KIND!***

## Installing KIND

Now that our prerequisites are satisfied and we have working Podman and `go` installations, let's install KIND.  KIND is installed using `go`, and will be installed in the `~/go/bin` directory.  We already added that directory to our path, so we can just type `kind` when we want to use KIND.

Install KIND:
```bash
go install sigs.k8s.io/kind@v0.17.0
```

We should see output like the following:
```bash
GET SOME OUTPUT
```

If we check the `~/go/bin` directory, we should see `kind`:
```bash
ls -al ~/go/bin
```

Checking KIND:
```bash
kind version
```

We should see something similar to the following:
```bash
kind v0.17.0 go1.19.5 darwin/amd64
```

*Again, versions and architectures can vary based on when you install and what you install it on!*

***Now that we have a working KIND installation, let's use it to deploy a KIND cluster!***

## Deploying a KIND Cluster

Before we dive right in and deploy our KIND cluster, let's take a quick look at how we use the `kind` command.  We have already added `~/go/bin` to our `PATH`, so we don't need to specify the full path to the `kind` executable.

We can use the `--help` switch to get information on the `kind` command:
```bash
kind --help
```

Help Output:
```bash
kind creates and manages local Kubernetes clusters using Docker container 'nodes'

Usage:
  kind [command]

Available Commands:
  build       Build one of [node-image]
  completion  Output shell completion code for the specified shell (bash, zsh or fish)
  create      Creates one of [cluster]
  delete      Deletes one of [cluster]
  export      Exports one of [kubeconfig, logs]
  get         Gets one of [clusters, nodes, kubeconfig]
  help        Help about any command
  load        Loads images into nodes
  version     Prints the kind CLI version

Flags:
  -h, --help              help for kind
      --loglevel string   DEPRECATED: see -v instead
  -q, --quiet             silence all stderr output
  -v, --verbosity int32   info log verbosity, higher value produces more output
      --version           version for kind

Use "kind [command] --help" for more information about a command.
```

The three verbs we're going to use in this tutorial are `get`, `create` and `delete`.

Let's check for existing KIND clusters before we proceed:
```bash
kind get clusters
```

We shouldn't see any clusters:
```bash
enabling experimental podman provider
No kind clusters found.
```

Let's create a cluster:
```bash
kind create cluster
```

We should see output similar to this:
```bash
enabling experimental podman provider
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Checking for clusters:
```bash
kind get clusters
```

Checking for our cluster:
```bash
enabling experimental podman provider
kind
```

We can see we have a KIND cluster named `kind`.  This cluster is actually a container, running under Podman:
```bash
podman ps -a
```

We can see our KIND container (`docker.io/kindest/node`):
```bash
CONTAINER ID  IMAGE                                                                                           COMMAND               CREATED         STATUS                     PORTS                      NAMES
274b6a12a161  docker.io/kindest/node@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1                        33 seconds ago  Up 32 seconds ago          127.0.0.1:61198->6443/tcp  kind-control-plane
```

Let's try deleting the cluster:
```bash
kind delete cluster
```

We should see output similar to this:
```bash
enabling experimental podman provider
Deleting cluster "kind" ...
```

Let's check for existing KIND clusters before we proceed:
```bash
kind get clusters
```

We shouldn't see any clusters:
```bash
enabling experimental podman provider
No kind clusters found.
```

## Summary

Even though Kubernetes and Podman are primarily Linux technologies, they can be leveraged on MacOS.  Using the power of Homebrew, we can build a nice, lightweight K8s environment on MacOS to learn and experiment with K8s.

If you'd like to learn some more advanced KIND concepts, check out the **KIND: Quick Start** link in the **References** section.

Enjoy!

*Tom Dean*
