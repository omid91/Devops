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

