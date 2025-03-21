# pre-installation steps 
---
## Step1: set the Hostname on each node and Update /etc/hosts
---
## step2: Swap configuration

The default behavior of a kubelet is to fail to start if swap memory is detected on a node. This means that swap should either be disabled or tolerated by kubelet.

```
sudo swapoff -a
```
