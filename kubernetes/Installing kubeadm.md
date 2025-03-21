# pre-installation steps 
---
## Step1: set the Hostname on each node and Update /etc/hosts
---
## step2: Swap configuration

The default behavior of a kubelet is to fail to start if swap memory is detected on a node. This means that swap should either be disabled or tolerated by kubelet.

```
sudo swapoff -a
```
it is not permanent

### To permanently disable swap
```
swapoff -a

sed -i '/swap/s/^\//\#\//g' /etc/fstab
```
## step3: Installing a container runtime

To run containers in Pods, Kubernetes uses a container runtime.

### You can use the link for installing
```
https://github.com/containerd/containerd/blob/main/docs/getting-started.md
```
Typically, you will have to install runc and CNI plugins from their official sites too.

### install containerd
```
Download containerd ---> https://github.com/containerd/containerd/releases
 tar Cxzvf /usr/local containerd-*-linux-amd64.tar.gz
```
### start containerd via systemd
```
vim /usr/local/lib/systemd/system/containerd.service
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```
### run the following commands
```
systemctl daemon-reload
systemctl enable --now containerd
```
## Installing runc
How Kubernetes Uses runc?
Kubernetes talks to container runtimes using the Container Runtime Interface (CRI).
Popular CRI-compatible runtimes:
containerd (default in Kubernetes) → Uses runc internally.
CRI-O (for OpenShift) → Uses runc internally.
runc is responsible for creating and starting containers within these runtimes.

```
Download runc ---> https://github.com/opencontainers/runc/releases
```

```
$ install -m 755 runc.amd64 /usr/local/sbin/runc
```
## Installing CNI plugins

A CNI (Container Network Interface) plugin is a standardized interface that defines how networking is configured for containers in Kubernetes or any containerized environment. It provides a way to set up networking and manage container networking environments.

```
Download CNI ---> https://github.com/containernetworking/plugins/releases
```

```
$ mkdir -p /opt/cni/bin
$ tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

## Step 4: Configure containerd

```
sudo containerd config default > /etc/containerd/config.toml

```

## Configuring the systemd cgroup driver 

To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set
set SystemdCgroup = true
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```
```
sudo systemctl restart containerd

sudo systemctl status containerd
```

## step 5:  Load Required Kernel Modules
```
sudo modprobe br_netfilter overlay
echo 'br_netfilter' | sudo tee -a /etc/modules-load.d/modules.conf
echo 'overlay' | sudo tee -a /etc/modules-load.d/modules.conf
lsmod | grep overlay
lsmod | grep br_netfilter
```
## step 6: Configure Kernel Parameters for Networking
```
sudo vim /etc/sysctl.conf

Add in sysctl.conf file--> net.ipv4.ip_forward=1

Reload sysctl Settings ---> sudo sysctl --system

Verify the Changes ---> sysctl net.ipv4.ip_forward

```
## step 7: Installing kubeadm, kubelet and kubectl
```
you can see document from this link ---> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime
```

## Update the apt package index and install packages needed to use the Kubernetes apt repository
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
```
## step 8: Configure kubelet service to use systemd

create file in path ---> vim /etc/kubernetes/kubelet-systemd.conf
add config file in kubelet-systemd.conf
```
  apiVersion:kubelet.config.k8s.io/v1beta1
  kind: kubeletConfiguration
  cgroupDriver: systemd
```

and add config to kuebelet.service to use kubelet-systemd.conf
```
 vim /lib/systemd/system/kubelet.service

ExecStart=/usr/bin/kubelet --config /etc/kubernetes/kubelet-systemd.conf

```

























