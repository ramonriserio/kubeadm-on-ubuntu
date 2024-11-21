# TUTORIAL 
# How to install a Kubernetes cluster on Ubuntu with kubeadm
https://www.youtube.com/watch?v=wIZamzt7MkM

## STEP 1: Installing kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

&emsp;_All **STEP 1** actions have to be performed in both control plane and worker nodes._

### I - VM Requirements
- 2GB of RAM
- 2 CPUs for control plane machines
- Disabled **swap** just now: ```sudo swapoff -a```
- Disabled **swap** permanently: Comment the swap line in the _/etc/fstab_ file

### II - Install a container runtime
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

**1. Enable IPv4 packet forward**
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional
```
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```
&emsp;Verify that net.ipv4.ip_forward is set to 1 with:
```
sysctl net.ipv4.ip_forward
```
**2. Install containerd**

https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

&emsp;Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

&emsp;2.1 - Set up Docker's apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
&emsp;2.2 - Install containerd package.

&emsp;&emsp;To install the latest version, run:
```
sudo apt-get install containerd.io
```
**3 - Configure kubelet and container runtime cgroup drivers**

On Linux, control groups are used to constrain resources that are allocated to processes.

Both the kubelet and the underlying container runtime (containerd) need to interface with control groups to enforce resource management for pods and containers and set resources such as cpu/memory requests and limits. To interface with control groups, the kubelet and the container runtime need to use a cgroup driver. It's critical that the kubelet and the container runtime use the same cgroup driver and are configured the same.

The cgroupfs driver is the default cgroup driver in the kubelet. When the cgroupfs driver is used, the kubelet and the container runtime directly interface with the cgroup filesystem to configure cgroups.

The cgroupfs driver is not recommended when systemd is the init system because systemd expects a single cgroup manager on the system.
```
# containerd config default | sed 's/SystemdCgroup=false/SystemdCgroup=true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml

containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.8"/sandbox_image = "registry.k8s.io\/pause:3.10"/' | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl status containerd
```

### IV - Install kubeadm, kubelet, and kubectl
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

&emsp;These instructions are for Kubernetes v1.31 on Debian-based distributions (like Ubuntu).

1. Update the apt package index and install packages needed to use the Kubernetes apt repository:
```
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
2. Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:
```
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
> [!NOTE]  
> In releases older than Debian 12 and Ubuntu 22.04, directory /etc/apt/keyrings does not exist by default, and it should be created before the curl command.

3 Add the appropriate Kubernetes ```apt``` repository. Please note that this repository have packages only for Kubernetes 1.31; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version (you should also check that you are reading the documentation for the version of Kubernetes that you plan to install).
```
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
4. Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
5. (Optional) Enable the kubelet service before running kubeadm:
```
sudo systemctl enable --now kubelet
```
## STEP 2: Only on the master node

### V - Create a cluster with kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

To initialize the control-plane node run:
```
kubeadm init --pod-network-cidr=10.200.0.0/16 --apiserver-advertise-address=<CONTROL PLANE IP>
```
> [!NOTE]
> Change <<CONTROL PLANE IP>> for your node IP.

&emsp;Then copy and paste the command after the text "To start using your cluster, you need to run the following as a regular user:"

&emsp;This command is like below:
```
kubeadm join <api-server-ip:port> --token <token-value> --discovery-token-ca-cert-hash sha256:<hash value>
```
&emsp;To start using your cluster, you need to run the following as a regular user: 
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
&emsp;Alternatively, if you are the root user, you can run:
```
  export KUBECONFIG=/etc/kubernetes/admin.conf
```
### VI - Verify the INTERNAL IP of the control plane running the command:
```
kubectl get node -o wide
```
&emsp;The **INTERNAL IP** have to be equal to the **node IP**. If they are differente, your host probably have more than one netwwork interface and kubeadm chose the wrong IP and you need to change it.

&emsp;To change the INTERNAL IP you need to add one more node IP value in kubeadm arguments and restart.

&emsp;First, type the command to edit the environment flag file of kubeadm:
```
sudo vi /var/lib/kubelet/kubeadm-flags.env
```
&emsp;Then add the node IP like below:
```
KUBELET_KUBEADM_ARGS="--node-ip=<NODE IP> --container-runtime-endpoint=unix:///var/run/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10"
```
&emsp;Then restart kubelet and verify control plane INTERNAL IP with the commands:
```
sudo systemctl restart kubelet
kubectl get node -o wide
```

## STEP 3: Only on the worker nodes

### VII - Add worker nodes

&emsp;You have to use the _kubeadm join_ command that appears as result of _kubeadm init_ command executed on master node.

&emsp;To recover kubeadm join command do:
```
sudo kubeadm token create --print-join-command
```

### VIII - Change workers INTERNAL IP addresses

&emsp;Just like you change control plane INTERNAL IP address.

## STEP 4: Install a Pod network add-on

&emsp;I chose flannel.

**1. Got to the github falnnel page:** https://github.com/flannel-io/flannel

**2. Download the manifest _kube-flannel.yml_ to the master node**
```
curl -fsSL https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml -o kube-flannel.yml
```
**3. Edit this manifest and change _"Network"_ value**

&emsp;_"Network"_ value have to be the same defined by flag _pod-network-cidr_ in _kubeadm init_ command. Like this: ```"Network": "10.200.0.0/10"```


**4. Verify if the _iface_ flag of the manifest corresponds to the name of the interface of the INTERNAL IP mater node interface**
```
ip a
```
```
...
containers:
- args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=enp0s8
```
&emsp;If _iface_ flag is different, change it.

**5. Apply and check the creation of Pods:**
```
kubedtl apply -f kube-flannel.yml
```

## STEP 5: Now, test the cluster

**1. Create several Pods**

**2. Deploy your application**

**3. Test your application**
