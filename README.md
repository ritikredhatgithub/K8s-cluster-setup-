Kubernetes Cluster Setup POC Environment with kubeadm 

Kubernetes is a powerful container orchestration platform used for automating the deployment, scaling, and management of containerized applications.
 
Kubernetes Nodes
In a Kubernetes cluster, you will encounter two distinct categories of nodes:
Master Nodes: These nodes play a crucial role in managing the control API calls for various components within the Kubernetes cluster. This includes overseeing pods, replication controllers, services, nodes, and more.

Worker Nodes: Worker nodes are responsible for providing runtime environments for containers. It’s worth noting that a group of container pods can extend across multiple worker nodes, ensuring optimal resource allocation and management.
########################################################### 

Kubeadm Installation Guide
Pre-requisites
● Ubuntu OS 22.04
● sudo privileges
● Internet access
● t2.medium instance type or higher

Make sure your all instances are in the same Security group.
Expose ports are on master node - protocol - TCP
● 6443
● 2379-2380
● 10250
● 10259
● 10257
Expose ports are on worker node - protocol - TCP
- 10250
- 10256
- 30000-32767
###########################################################

Deployment Steps 

Execute on Both "Master" 

# disable swap
sudo swapoff -a

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter 

# sysctl params required by setup, params persist across
reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF 

# Apply sysctl params without reboot
sudo sysctl --system 

## Install CRIO Runtime(Containerd Runtime)
sudo apt-get update -y
sudo apt-get install -y software-properties-common curl
apt-transport-https ca-certificates gpg

# Enabling Docker Repository
sudo curl -fsSL
https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key |
sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg]
https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | sudo tee
/etc/apt/sources.list.d/cri-o.list

# Update the package list and install containerd
sudo apt-get update -y
sudo apt-get install -y cri-o

# Restart and enable containerd service
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service 

# Add Kubernetes APT repository and install required packages
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key |
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg]
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee
/etc/apt/sources.list.d/kubernetes.list

# Install kubectl,kubeadm and kubelet
sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# If install specific version use below command
[ sudo apt-get install -y kubelet="1.29.0-*" kubectl="1.29.0-*"
kubeadm="1.29.0-*" ]
sudo apt-get update -y
sudo apt-get install -y jq
sudo systemctl enable --now kubelet
sudo systemctl start kubelet


# Initialize kubernetes cluster with kubeadm (master node)
sudo kubeadm config images pull
sudo kubeadm init     

# After initiaisation is complete make a note of the kubeadm join command for future reference


mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config


# Network Plugin = calico
kubectl apply -f
“https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manife
sts/calico.yaml”



##################################################################
# Worker Node

# disable swap
sudo swapoff -a

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter 

# sysctl params required by setup, params persist across
reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

## Install CRIO Runtime
sudo apt-get update -y
sudo apt-get install -y software-properties-common curl
apt-transport-https ca-certificates gpg
sudo curl -fsSL
https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key |
sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg]
https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | sudo tee
/etc/apt/sources.list.d/cri-o.list
sudo apt-get update -y
sudo apt-get install -y cri-o
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service

# Add Kubernetes APT repository and install required packages
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key |
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg]
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee
/etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl

# If install specific version use below command
[ sudo apt-get install -y kubelet="1.29.0-*" kubectl="1.29.0-*"
kubeadm="1.29.0-*" ]
sudo apt-get update -y
sudo apt-get install -y jq
sudo systemctl enable --now kubelet
sudo systemctl start kubelet

# Execute on ALL of your Worker Node's
sudo “your-token” --v=5


# Verify Cluster Connection

# On Master Node:
kubectl get nodes

# Verify the cluster
kubectl get pods -n kube-system






