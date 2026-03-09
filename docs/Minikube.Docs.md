# Overview

Minikube is a single node kubernetes cluster.
For current learn topic we will create Linux VM and run minikube inside this VM using docker driver.

[First Configure VM for Minikube](./Minikube.VM.md)

Then we need to run minikube inside VM

## Startup options

[minikube start --help](https://minikube.sigs.k8s.io/docs/commands/start/)

When minikube starts it will create kubernetes (k8s for short) cluster with single node.
It is ipmortat to tell minikube how to create k8s cluster by flag `--driver={driverName}` since minikube will take differen actions for each driver.
For current topic we well use `--driver=docker` inside dedicated VM.

```bash

# flag `--driver` expect dirver name
# --vm-driver is deprecated
minikube start --driver=docker
```

**(it also has old flag `--vm-driver` with the same meaning but is not recomended since it is deprecated)**

### For Linux

| Command | Documentation | Description |
|---|---|---|
| minikube start --driver=docker | [docs](https://minikube.sigs.k8s.io/docs/drivers/docker/) | docker container-based (preferred) |
| minikube start --driver=podman | [docs](https://minikube.sigs.k8s.io/docs/drivers/podman/) | alternative to docker, container-based (experimental) |
| minikube start --driver=virtualbox | [docs](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/) | Will create VM and run k8s cluster inside |
| minikube start --driver=kvm2 | [docs](https://minikube.sigs.k8s.io/docs/drivers/kvm2/) | Will create VM and run kubernetes cluster inside |
| minikube start --driver=qemu | [docs](https://minikube.sigs.k8s.io/docs/drivers/qemu/) | Will create VM and run kubernetes cluster inside |
| minikube start --driver=ssh --ssh-ip-address=vm.example.com | [docs](https://minikube.sigs.k8s.io/docs/drivers/ssh/) | connect to remote host and run k8s cluster inside |
| minikube start --driver=none | [docs](https://minikube.sigs.k8s.io/docs/drivers/none/) | bare-metal |

### For Windows

| Command | Documentation | Description |
|---|---|---|
| minikube start --driver=docker | [docs](https://minikube.sigs.k8s.io/docs/drivers/docker/) | VM + docker container-based (preferred) |
| minikube start --driver=podman | [docs](https://minikube.sigs.k8s.io/docs/drivers/podman/) | VM + alternative to docker, container-based (experimental) |
| minikube start --driver=hyperv | [docs](https://minikube.sigs.k8s.io/docs/drivers/hyperv/) | Will create VM and run k8s cluster inside |
| minikube start --driver=virtualbox | [docs](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/) | Will create VM and run k8s cluster inside |
| minikube start --driver=vmware | [docs](https://minikube.sigs.k8s.io/docs/drivers/vmware/) | Will create VM and run k8s cluster inside |
| minikube start --driver=qemu | [docs](https://minikube.sigs.k8s.io/docs/drivers/qemu/) | Will create VM and run kubernetes cluster inside |
| minikube start --driver=ssh --ssh-ip-address=vm.example.com | [docs](https://minikube.sigs.k8s.io/docs/drivers/ssh/) | connect to remote host and run k8s cluster inside |

### For macOS

| Command | Documentation | Description |
|---|---|---|
| minikube start --driver=docker | [docs](https://minikube.sigs.k8s.io/docs/drivers/docker/) | VM + docker container-based (preferred) |
| minikube start --driver=podman | [docs](https://minikube.sigs.k8s.io/docs/drivers/podman/) | VM + alternative to docker, container-based (experimental) |
| minikube start --driver=virtualbox | [docs](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/) | Will create VM and run k8s cluster inside |
| minikube start --driver=hyperkit | [docs](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/) | Will create VM and run k8s cluster inside |
| minikube start --driver=parallels | [docs](https://minikube.sigs.k8s.io/docs/drivers/parallels/) | Some kind of VM |
| minikube start --driver=vmware | [docs](https://minikube.sigs.k8s.io/docs/drivers/vmware/) | Will create VM and run k8s cluster inside |
| minikube start --driver=qemu | [docs](https://minikube.sigs.k8s.io/docs/drivers/qemu/) | Will create VM and run kubernetes cluster inside |
| minikube start --driver=vfkit --network vmnet-shared | [docs](https://minikube.sigs.k8s.io/docs/drivers/vfkit/) | Will create VM and run kubernetes cluster inside |
| minikube start --driver=krunkit | [docs](https://minikube.sigs.k8s.io/docs/drivers/krunkit/) | Will create VM and run kubernetes cluster inside |
| minikube start --driver=ssh --ssh-ip-address=vm.example.com | [docs](https://minikube.sigs.k8s.io/docs/drivers/ssh/) | connect to remote host and run k8s cluster inside |

## Commands

```bash
minikube --help

# Output:
minikube provisions and manages local Kubernetes clusters optimized for development workflows.

Basic Commands:
  start            Starts a local Kubernetes cluster
  status           Gets the status of a local Kubernetes cluster
  stop             Stops a running local Kubernetes cluster
  delete           Deletes a local Kubernetes cluster
  dashboard        Access the Kubernetes dashboard running within the minikube cluster
  pause            pause Kubernetes
  unpause          unpause Kubernetes

Images Commands:
  docker-env       Provides instructions to point your terminal's docker-cli to the Docker Engine inside minikube.
(Useful for building docker images directly inside minikube)
  podman-env       Configure environment to use minikube's Podman service
  cache            Manage cache for images
  image            Manage images

Configuration and Management Commands:
  addons           Enable or disable a minikube addon
  config           Modify persistent configuration values
  profile          Get or list the current profiles (clusters)
  update-context   Update kubeconfig in case of an IP or port change

Networking and Connectivity Commands:
  service          Returns a URL to connect to a service
  tunnel           Connect to LoadBalancer services

Advanced Commands:
  mount            Mounts the specified directory into minikube
  ssh              Log into the minikube environment (for debugging)
  kubectl          Run a kubectl binary matching the cluster version
  node             Add, remove, or list additional nodes
  cp               Copy the specified file into minikube

Troubleshooting Commands:
  ssh-key          Retrieve the ssh identity key path of the specified node
  ssh-host         Retrieve the ssh host key of the specified node
  ip               Retrieves the IP address of the specified node
  logs             Returns logs to debug a local Kubernetes cluster
  update-check     Print current and latest version number
  version          Print the version of minikube
  options          Show a list of global command-line options (applies to all commands).

Other Commands:
  completion       Generate command completion for a shell
  license          Outputs the licenses of dependencies to a directory


Use "minikube <command> --help" for more information about a given command.
```
