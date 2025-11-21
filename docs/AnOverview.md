# Overview

    Questions:
    - What is Kubernetes?
    - What it consist of?
    - What instruments exists?
    - Where it is useful?
    - Does it have any analogs?

## What is Kubernetes

Kubernetes is a container management system which operates in a cluster.
In addition it also does traffic routing for containers (it has load balancing capabilities).

Basically Kubernetes keep a definition of desired application state, and constantly observe current containers state - when desired and actual state are different it performs all necessary operation to reach desired state.
So developing with kubernetes mean - create a description of desired state (typically in yaml format, it also possible to use json).
In desired state it is possible to describe services, amount of container that should run those services, resource constrains (like cpu, ram, persistent storage requirements), network structure for application.
It is possible to setup and run tasks (one time tasks or scheduled repeated tasks).

> ## Kubernetes consist of
>
> - **Controller panel** - which hold information about all objects in cluster, it constantly monitors the cluster for any changes
> - **Nodes** - represents host machines, virtual or physical machines which are part of the cluster (Controller panel itself located inside of such nodes, typically its a single node that does not run anything else except Controller panel).
> - **Pods** - those are resource environment in which containers are executed, Pod may run single container (one pod per container) or multiple containers (in this case containers will share same resources)
> - **Containers** - an actual application which contain everything it need in order to run independently. Kubernetes use Container Runtime Interface (CRI) to interact with container. Kubernetes support such runtimes: Docker, Podman, CRI-O (basically anything that implements Kubernetes Container Runtime Interface)

## What instruments exists

Kubernetes use set of CLI tools in order to operate (basic list of tools):

- kubeadm - CLI tool for creating Kubernetes cluster
- kubectl - command line (CLI) tool allow to communicate with Kubernetes cluster control panel
- Docker and Docker registry - for container images build and run (Kubernetes support custom Docker registries but by default it goes to public one)
- minikube - CLI tool for creating local Kubernetes cluster (primary for education or testing purposes)
- K3s - open-source Lightweight Kubernetes. Easy to install, half the memory, all in a binary of less than 100 MB.
- MicroK8s - open-source system for automating deployment, scaling, and management of containerized applications. It provides the functionality of core Kubernetes components, in a small footprint, scalable from a single node to a high-availability production cluster.


## Where it is useful

Since Kubernetes is a container orchestrator - it is useful for application deployment, scaling, load balancing, traffic routing, logging and tracing (this one require additional work because Kubernetes only gives basic capabilities for logging), self-healing (keep track of running containers and can restart them when necessary)

## Analogs

- Docker Swarm - built into Docker engine
- Hashicorp Nomad - alternative approach
- VMware Tanzu
- AWS Elastic Container Service (ECS) - by cloud provider
- Rancher - Rancher is an open source container management platform built for organizations that deploy containers in production. Rancher makes it easy to run Kubernetes everywhere, meet IT requirements, and empower DevOps teams.

There lots of cloud providers which provides similar services (like AWS Elastic Container Service).
Many of the analogs are actually Kubernetes with some additional functionality or cloud integration (like AWS Elastic Kubernetes Service).