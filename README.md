# TUTORIAL 
# How to install a Kubernetes cluster on Ubuntu with kubeadm
https://www.youtube.com/watch?v=wIZamzt7MkM

## STEP 1: Installing kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### I - VM Requirements
- 2GB of RAM
- 2 CPUs for control plane machines
- Disabled **swap** just now: * *sudo swapoff -a_ _
- Disabled **swap** permanently: Comment the swap line in the * */etc/fstab_ _ file

### II - Install a container runtime
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

**1. Enable IPv4 packet forward**
**2. Install containerd**
=> teste
     -> docker.com
       > Docs
         > Docker Engine
           > Install Docker Engine
             > Ubuntu
               > Installing using apt repository
                 > Steps 1 and 2 for **containerd.io**

### III - cgroup drivers ###
