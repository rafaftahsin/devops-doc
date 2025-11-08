---
title: "How to spin up a self-hosted k8s cluster with kubeadm"
---

### Prerequisites

- Swap should be disabled
  - Check with `sudo swapon -s`
  - Disable with `sudo swapoff -a`
- Port 6443 should be open
  - Check with `sudo netstat -tulpn | grep 6443`
  - Open with `sudo ufw allow 6443` if using ufw.

### Install container runtime

- Install `containerd`

```shell
### containerd
wget https://github.com/containerd/containerd/releases/download/v2.2.0/containerd-2.2.0-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-2.2.0-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system
sudo mv containerd.service /usr/local/lib/systemd/system/containerd.service
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

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

Ref: 

1. https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
2. 
