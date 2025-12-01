# Create Virtual Machine (VM) For Minikube

Resources:

- [Virtual Box Downloads](https://www.virtualbox.org/wiki/Downloads)
- [Ubuntu Server Downloads](https://ubuntu.com/download/server)
- [Docker windows CLI releases](https://download.docker.com/win/static/stable/x86_64/)
- [Docker-Compose windows CLI releases](https://github.com/docker/compose/releases)

First Install [VirtualBox](https://download.virtualbox.org/virtualbox/7.2.4/VirtualBox-7.2.4-170995-Win.exe)  
Download [Ubuntu Server 24.04.3 LTS 3GB](https://mirrors.vinters.com/ubuntu-releases/24.04.3/ubuntu-24.04.3-live-server-amd64.iso)

Create virtual machine:

## Configure Nat Network

[VirtualBox Network Documentation](https://www.virtualbox.org/manual/ch06.html)

| Mode | VM→Host | VM←Host | VM1↔VM2 | VM→Net/LAN | VM←Net/LAN |
|---|---|---|---|---|---|
|Host-only | + | + | + | – | – |
| Internal | – | – | + | – | – |
| Bridged | + | + | + | + | + |
| NAT | + | Port forward | – | + | Port forward |
| NATservice | + | Port forward | + | + | Port forward |

Loopback address for VM: **127.0.0.10**  
Loopback port for VM SSH: **3022**  
VM IP address: **10.0.2.3**

```bash
# Bash VM
# VM IP address may be checked on VM host by
sudo ip address
```

## Configure SSH Connection

First check (or install) SSH on Linux VM

```bash
# Bash VM
# First connect to VM CLI and check if ssh installed
sudo systemctl status ssh

# Install SSH (if nessecary)
sudo apt update
sudo apt install openssh-server

# Enable SSH service on VM
sudo systemctl enable ssh --now
```

Check connection from host machine

```powershell
# Powershell Host
# Will ask for user password
ssh -p 3022 vbox@127.0.0.10
```

Generate SSH key pair for VM

SSH key name: **id_ed25519_vm_vbox**  
SSH public key name: **id_ed25519_vm_vbox.pub**  

```powershell
# Powershell Host
# Navigate to your SSH directry
cd ~/.ssh
# Generete new SSH key with file name id_ed25519_vm_vbox (password is optional, leave empty if not needed)
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

## Install docker on windows host (optional)

Install docker on host windows machine (not nessecary when docker already installed).
Check for latest version:
[Docker windows CLI releases](https://download.docker.com/win/static/stable/x86_64/),
[Docker-Compose windows CLI releases](https://github.com/docker/compose/releases)

```powershell
# Powershell Host
# Intall docker CLI on host windows machine
# Search for latest version at https://download.docker.com/win/static/stable/x86_64/
curl.exe -o docker.zip -LO https://download.docker.com/win/static/stable/x86_64/docker-29.0.4.zip
Expand-Archive docker.zip -DestinationPath "C:\"

# Install docker-compose on host windows machine
# Search for latest version at https://github.com/docker/compose/releases
Invoke-WebRequest "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile C:\docker\docker-compose.exe

# Update env Path to containe docker command
[Environment]::SetEnvironmentVariable("Path", "$($env:path);C:\\docker", [System.EnvironmentVariableTarget]::Machine)
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
```

In case windows images are required - it is possible to configure windows docker service to run

```powershell
# Powershell Host
# Create docker service configureation
# See https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon
echo '{"hosts": ["tcp://0.0.0.0:2375", "npipe://"]}' >> C:\\ProgramData\docker\\config\\daemon.json
dockerd --register-service
Start-Service docker

# Add windows docker to docker CLI
docker context create win --default-stack-orchestrator=swarm --docker host=tcp://127.0.0.1:2375 --description "Windows tcp://localhost:2375"
```

## Install Docker for Linux VM

VM Configuration

```bash
# Bash VM
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

VM Configuration

```bash
# Bash VM
# Configure Docker to accept Unix socket (form insed VM) and ssh connections (from network)
sudo tee /etc/docker/daemon.json <<EOF
{
 "hosts":["unix:///var/run/docker.sock", "ssh://0.0.0.0"]
}
EOF
```

Configure host docker CLI to connect VM docker through SSH

```powershell
# Powershell Host
# Create context to access VM docker
docker context create linux-minikube --description "Docker in VM minikube" --docker "host=ssh://vbox@127.0.0.10:3022"
```
