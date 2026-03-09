# Create Virtual Machine (VM) For Minikube

Resources:

- [Virtual Box Downloads](https://www.virtualbox.org/wiki/Downloads)
- [Ubuntu Server Downloads](https://ubuntu.com/download/server)
- [Docker windows CLI releases](https://download.docker.com/win/static/stable/x86_64/)
- [Docker-Compose windows CLI releases](https://github.com/docker/compose/releases)

## Install VirtualBox

Download and install [VirtualBox](https://download.virtualbox.org/virtualbox/7.2.4/VirtualBox-7.2.4-170995-Win.exe)

**Optional**: set env variable for getting access to virtualbox CLI capabilities.
Use current user scope (do not want to set Machine scope just to keep things isolated and clean).

```powershell
[Environment]::SetEnvironmentVariable("Path", "$($env:path);C:\Program Files\Oracle\VirtualBox", [System.EnvironmentVariableTarget]::User)
```

## Install Ubuntu VM

Download [Ubuntu Server 24.04.3 LTS 3GB](https://mirrors.vinters.com/ubuntu-releases/24.04.3/ubuntu-24.04.3-live-server-amd64.iso)

Using virtualbox UI create virtual machine and install ubuntu server.  
Current setup expects to have 4gb ram, and 4 cpu cores (it should be enough for minikube).

> Virtualbox has CLI that can do any VM manipulations, but for simplicity we are not using it here.

## Configure Nat Network

Configure VM network in a way to have access to VM through SSH, and also for VM to have internet connection.

Expected VM lan configuration:

- Loopback address for VM: `127.0.0.10` (yuo can choose any loopback address)
- Loopback port for VM SSH: `3022`
- VM IP address: `10.0.2.3*`

```bash
# linux VM
# VM IP address may be checked on VM host by
sudo ip address
```

[VirtualBox Network Documentation](https://www.virtualbox.org/manual/ch06.html)

| Mode | VM→Host | VM←Host | VM1↔VM2 | VM→Net/LAN | VM←Net/LAN |
|---|---|---|---|---|---|
|Host-only | + | + | + | – | – |
| Internal | – | – | + | – | – |
| Bridged | + | + | + | + | + |
| NAT | + | Port forward | – | + | Port forward |
| NATservice | + | Port forward | + | + | Port forward |

Create new NAT Network:

- name: `NatMinikube`
- ip: `10.0.2.0/24`

Add port forwarding for SSH connection to VM (port forwarding should be added to **NatMinikube** network):

- name: `SSH Minikube`
- protocol: `TCP` (ssh traffic)
- host ip: `127.0.0.10` (can be any loopback ip, this config will allow only this ip to forward requests to VM)
- host port: `3022` (use `127.0.0.10:3022` to create SSH connections to VM)
- guest ip: `10.0.2.3` (ip address assigned to VM network adapter by virtualbox, can by checked on VM by `sudo ip address`)
- guest port: `22` (SSH server listen to `22` port inside VM)

Configure VM to use `NatMinikube` network:

- go to VM `Settings > Network`
- set `Attach To` = `NAT Network`
- set `Name` = `NatMinikube`

## Configure SSH Connection

First we need to configure VM to accept SSH connection (expect to have configured NAT Network at this point).

Check if SSH already installed on VM

```bash
# linux VM
# First connect to VM CLI and check if ssh installed
sudo systemctl status ssh
```

Install SSH server to VM (when it is missing)

```bash
# Install SSH (if necessary)
sudo apt update
sudo apt install openssh-server

# Enable SSH service on VM
sudo systemctl enable ssh --now
```

Check connection from host machine (when SSH is not configured - CLI ask for VM user password).

> If any error occurs during the check - try to find any mismatches in Nat network port forwarding configuration (maybe ip address of VM does not match port forwarding configuration)

```powershell
# windows Host
ssh -p 3022 vbox@127.0.0.10
```

**In order to avoid password prompt for each CLI connection to VM** - lets generate SSH key on windows host,
and copy public key VM.

- generate SSH key pair on windows host (use CLI tool `ssh-keygen`)
- configure windows host to use generated private key for VM ssh connections (`~/.ssh/config` file may have multiple configurations, do not delete existed settings),
- copy public key to VM `~/.ssh/authorized_keys`.
- test connection (if configured correctly should not ask for user password)

SSH key name: `id_ed25519_vm_vbox`  
SSH public key name: `id_ed25519_vm_vbox.pub`  

```powershell
# windows Host
# Navigate to your SSH directory
cd ~/.ssh
# Generate new SSH key with file name id_ed25519_vm_vbox (password is optional, leave empty if not needed)
ssh-keygen -t ed25519 -C "vbox" -f id_ed25519_vm_vbox

# Configure to use id_ed25519_vm_vbox key for VM
Add-Content -Path "config" -Value "Host 127.0.0.10"
Add-Content -Path "config" -Value " IdentityFile ~/.ssh/id_ed25519_vm_vbox"

# Display your public key
cat "~\.ssh\id_ed25519_vm_vbox.pub"

# Copy SSH public key to VM
ssh -p 3022 vbox@127.0.0.10 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo '$(Get-Content "~\.ssh\id_ed25519_vm_vbox.pub")' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Check SSH connection
# When configured correctly - will not ask for user password but use ssh public key instead
ssh -p 3022 vbox@127.0.0.10
```

## Install docker on windows Host (optional)

Install docker on host windows machine (not necessary when docker already installed).
Windows docker CLI allows to access and observe docker images and container on linux VM (and any other place, if configured).

Check for latest version:

- [Docker windows CLI releases](https://download.docker.com/win/static/stable/x86_64/),
- [Docker-Compose windows CLI releases](https://github.com/docker/compose/releases)

```powershell
# windows Host
# Install docker CLI on host windows machine
# Search for latest version at https://download.docker.com/win/static/stable/x86_64/
curl.exe -o docker.zip -LO https://download.docker.com/win/static/stable/x86_64/docker-29.0.4.zip
Expand-Archive docker.zip -DestinationPath "C:\"

# Install docker-compose on host windows machine
# Search for latest version at https://github.com/docker/compose/releases
Invoke-WebRequest "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile C:\docker\docker-compose.exe

# Update env Path to contain docker command for current machine
[Environment]::SetEnvironmentVariable("Path", "$($env:path);C:\\docker", [System.EnvironmentVariableTarget]::Machine)
# Update path cor current session
$env:Path = [System.Environment]::GetEnvironmentVariable("Path")
```

**Optional:** In case windows images are required - it is possible to configure windows docker service to run

```powershell
# windows Host
# Create docker service configuration
# See https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon
echo '{"hosts": ["tcp://0.0.0.0:2375", "npipe://"]}' >> C:\\ProgramData\docker\\config\\daemon.json
dockerd --register-service
Start-Service docker

# Add windows docker to docker CLI
docker context create win --default-stack-orchestrator=swarm --docker host=tcp://127.0.0.1:2375 --description "Windows tcp://localhost:2375"
```

## Install Docker for linux VM

Required for minikube to run docker containers

```bash
# linux VM
# Add Docker official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker should start automatically, check service status
sudo systemctl status docker

# If docker does not started - run it
sudo systemctl start docker

# Test installed docker
sudo docker run --rm hello-world

# Add your user to the 'docker' group
sudo usermod -aG docker $USER && newgrp docker
```

## Configure Docker SSH Connection

Configure VM docker daemon do accept connections form unix socket (from inside of VM itself), and SSH connections (from windows host)

```bash
# linux VM
# Configure Docker to accept Unix socket (form insed VM) and ssh connections (from network)
sudo tee /etc/docker/daemon.json <<EOF
{
 "hosts":["unix:///var/run/docker.sock", "ssh://0.0.0.0"]
}
EOF
```

Configure windows host docker CLI to connect VM docker through SSH

```powershell
# windows Host
# Create context to access VM docker
docker context create linux-minikube --description "Docker in VM minikube" --docker "host=ssh://vbox@127.0.0.10:3022"
```

## Install minikube

Installing minikube to linux VM

[Check available releases](https://github.com/kubernetes/minikube/releases)

```bash
# linux VM
# For latest version use: https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
curl -LO https://github.com/kubernetes/minikube/releases/download/v1.37.0/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

minikube version
# Output:
#
# minikube version: v1.37.0
# commit: 65318f4cfff9c12cc87ec9eb8f4cdd57b25047f3

# Run to test
minikube start --driver=docker
```

## Install kubectl

The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. For more information including a complete list of kubectl operations, see the [kubectl reference documentation](https://kubernetes.io/docs/reference/kubectl/).

You must use a kubectl version that is within one minor version difference of your cluster. For example, a v1.34 client can communicate with v1.33, v1.34, and v1.35 control planes. Using the latest compatible version of kubectl helps avoid unforeseen issues.

[Check for lates version](https://dl.k8s.io/release/stable.txt)

### (Use this) Install specific version from binaries

Use version `v1.34.2`

```bash
# linux VM
# Download binaries
curl -LO https://dl.k8s.io/release/v1.34.2/bin/linux/amd64/kubectl

# Download checksum
curl -LO https://dl.k8s.io/release/v1.34.2/bin/linux/amd64/kubectl.sha256

# Validate checksum
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Check installation

kubectl version --client

# Output:
#
# Client Version: v1.34.2
# Kustomize Version: v5.7.1
```

> The 'install' command in Linux is a versatile tool used for copying files and setting their attributes, such as permissions, ownership, and group. Unlike commands like 'cp' that simply copy files, 'install' allows you to fine-tune these settings, making it ideal for installation scripts or manual file management. It can copy files to a specified destination and set permissions, ownership, or group attributes, all in one go.

### (Optional) Install latest stable version

```bash
# linux VM
# Download binaries
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Download checksum
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

# Validate checksum
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Check installation

kubectl version --client

# Output:
#
# Client Version: v1.34.2
# Kustomize Version: v5.7.1
```

## Use Windows Terminal

Create Terminal Profile for connecting to created VM through SSH

```json
// Open Terminal Settings > Open Json file
// Copy fallowing code to { "profiles" : "list" : [ placeItHere ]}
{
    "commandline": "%SystemRoot%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe ssh -p 3022 vbox@127.0.0.10",
    "guid": "{9f0ded21-c8e5-495f-afc8-74fe7062a17a}",
    "hidden": false,
    "name": "SSH ubuntu-24-server-minikube"
}
```
