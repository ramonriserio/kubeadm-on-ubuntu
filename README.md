# TUTORIAL 
# How to install a Kubernetes cluster on Ubuntu with kubeadm
https://www.youtube.com/watch?v=wIZamzt7MkM

## STEP 1: Installing kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

_All STEP 1 actions have to be performed both control plane and worker nodes._

### I - VM Requirements
- 2GB of RAM
- 2 CPUs for control plane machines
- Disabled **swap** just now: * *sudo swapoff -a_ _
- Disabled **swap** permanently: Comment the swap line in the _/etc/fstab_ file

### II - Install a container runtime
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

1. Enable IPv4 packet forward
2. Install containerd

- docker.com
- Docs
- Docker Engine
- Install Docker Engine
- Ubuntu
- Installing using apt repository
- Steps 1 and 2 for **containerd.io**

### III - cgroup drivers ###
```
containerd config default | sed 's/SystemdCgroup=false/SystemdCgroup=true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl status containerd
```

### IV - Install kubeadm, kubelet, and kubectl
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## STEP 2: Only in master node

### V - Create a cluster with kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
```
kubeadm init --pod-network-cidr=10.200.0.0/16 --apiserver-advertise-address=<CONTROL PLANE IP>
```
Then copy and paste the command after the text "To start using your cluster, you need to run the following as a regular user:"

This command is like below:
```
kubeadm join <api-server-ip:port> --token <token-value> --discovery-token-ca-cert-hash sha256:<hash value>
```
### VI - Verify the INTERNAL IP of the control plane running the command:
```
kubectl get node -o wide
```
The **INTERNAL IP** have to be equal to the **node IP**. If they are differente, your host probably have more than one netwwork interface and kubeadm chose the wrong IP and you need to change it.

To change the INTERNAL IP you need to add one more node IP value in kubeadm arguments and restart.

First, type the command to edit the environment flag file of kubeadm:
```
sudo vi /var/lib/kubelet/kubeadm-flags.env
```
Then add the node IP like below:
```
KUBELET_KUBEADM_ARGS="--node-ip=<NODE IP> --container-runtime-endpoint=unix:///var/run/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10"
```
Then restart kubelet and verify control plane INTERNAL IP with the commands:
```
sudo systemctl restart kubelet
kubectl get node -o wide
```

## STEP 3: Only in worker nodes

### VII - Adding worker nodes
