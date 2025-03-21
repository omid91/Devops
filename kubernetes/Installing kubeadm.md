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












