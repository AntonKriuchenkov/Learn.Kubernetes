# Run Minikube

Check minikube status before start

```bash
minikube version

# Output:
#
# minikube version: v1.37.0
# commit: 65318f4cfff9c12cc87ec9eb8f4cdd57b25047f3

minikube status

# Output:
#
# minikube
# type: Control Plane
# host: Stopped
# kubelet: Stopped
# apiserver: Stopped
# kubeconfig: Stopped
```

Run kubernetes cluster in minikube

```bash
# flag `--driver` expect dirver name
# (it also has old flag `--vm-driver` with the same meaning but is not recomended since it is deprecated)
minikube start --driver=docker
```

Check minikube status when started

```bash
# Check current minikube status
minikube status

# Output:
#
# minikube
# type: Control Plane
# host: Running
# kubelet: Running
# apiserver: Running
# kubeconfig: Configured
```

Check docker images used by minikube

```bash
minikube image ls

# Output:
#
# registry.k8s.io/pause:3.10.1
# registry.k8s.io/kube-scheduler:v1.34.0
# registry.k8s.io/kube-proxy:v1.34.0
# registry.k8s.io/kube-controller-manager:v1.34.0
# registry.k8s.io/kube-apiserver:v1.34.0
# registry.k8s.io/etcd:3.6.4-0
# registry.k8s.io/coredns/coredns:v1.12.1
# gcr.io/k8s-minikube/storage-provisioner:v5
```

Show existed nodes managed by minikube

```bash
minikube node list

# Output:
#
# minikube        192.168.49.2
```

Get existed service urls (which should return empty list because we did not install aything yet)

```bash
minikube service --all

# Output:
#
# ┌───────────┬────────────┬─────────────┬──────────────┐
# │ NAMESPACE │    NAME    │ TARGET PORT │     URL      │
# ├───────────┼────────────┼─────────────┼──────────────┤
# │ default   │ kubernetes │             │ No node port │
# └───────────┴────────────┴─────────────┴──────────────┘
# 😿  service default/kubernetes has no node port
# ❗  Services [default/kubernetes] have type "ClusterIP" not meant to be exposed, however for local development minikube allows you to access this !
```

## Minikube and kubectl

Minikube is a CLI tool that creates and manage minimal kubernetes cluster.  
Kubectl is standard Kubernetes CLI tool - allows to run commands against k8s clusters.

How to connect kubectl to minikube on the same VM:  
When minikube starts - it creates cluster config `~/.kube/config` which is used by kubectl, so when kubectl installed on VM it will automatically use this config to operate with minikube cluster.

Check created minikube cluster

```bash
cat ~/.kube/config
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vbox/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Wed, 03 Dec 2025 18:22:05 UTC
        provider: minikube.sigs.k8s.io
        version: v1.37.0
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Wed, 03 Dec 2025 18:22:05 UTC
        provider: minikube.sigs.k8s.io
        version: v1.37.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/vbox/.minikube/profiles/minikube/client.crt
    client-key: /home/vbox/.minikube/profiles/minikube/client.key
```

## Use kubectl

[kubectl references](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

```bash

kubectl cluster-info

# Get control plane address
```

> TODO: how to install same https certificate as for minikube k8s