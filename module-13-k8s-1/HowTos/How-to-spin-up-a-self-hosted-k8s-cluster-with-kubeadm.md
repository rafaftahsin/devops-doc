---
title: "How to spin up a self-hosted k8s cluster with kubeadm"
parent:  'HowTos'
layout: home
grand_parent: 'Module 13 k8s-1'
---

### Prerequisites

- Swap should be disabled
  - Check with `sudo swapon -s`
  - Disable with `sudo swapoff -a`
- Port 6443 should be open
  - Check with `sudo netstat -tulpn | grep 6443`
  - Open with `sudo ufw allow 6443` if using ufw.
- Enable IPv4 Packet forwarding
  - Check with `sudo sysctl net.ipv4.ip_forward`
  - Enable with 
- Each Node must have at least 2GB of RAM, 2 CPUs, and 10GB of free disk space.

```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

### Install container runtime

- Install `containerd`

```shell
### containerd
wget https://github.com/containerd/containerd/releases/download/v2.2.0/containerd-2.2.0-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-2.2.0-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system
sudo mv containerd.service /usr/local/lib/systemd/system/containerd.service
containerd config default > /etc/containerd/config.toml
# Configure systemd cgroup driver - https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd
# This should work - sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl daemon-reload
sudo systemctl enable containerd
sudo systemctl start containerd

### runc
wget https://github.com/opencontainers/runc/releases/download/v1.3.3/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

### CNI Plugins
wget https://github.com/containernetworking/plugins/releases/download/v1.8.0/cni-plugins-linux-amd64-v1.8.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.8.0.tgz
```
Ref: 

1. https://github.com/containerd/containerd/blob/main/docs/getting-started.md
2. https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
3. container systemd config update -> https://github.com/containerd/containerd/discussions/5413


### Installing `kubeadm`, `kubelet` and `kubectl` 

You will install these packages on all of your machines:

- `kubeadm`: the command to bootstrap the cluster.
- `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
- `kubectl`: the command line util to talk to your cluster.

```shell
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
# This wil prevent the kubelet from being upgraded automatically.
sudo apt-mark hold kubelet kubeadm kubectl

# Enable and start kubelet
sudo systemctl enable --now kubelet
```

Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### Creating a cluster

```shell
sudo kubeadm init --apiserver-advertise-address=10.43.60.172
```

#### Note: If you get the following error 

```shell
root@k8s-master:~# kubeadm init --apiserver-advertise-address=10.43.60.172
[init] Using Kubernetes version: v1.34.1
[preflight] Running pre-flight checks
[preflight] Some fatal errors occurred:
	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
	[ERROR Mem]: the system RAM (955 MB) is less than the minimum 1700 MB
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
error: error execution phase preflight: preflight checks failed
To see the stack trace of this error execute with --v=5 or higher

```

Ref: 

1.  https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
2. `kubeadm` reference guide -> https://kubernetes.io/docs/reference/setup-tools/kubeadm/ 