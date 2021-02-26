# kubernetes-lab v1
Repo for K8S hands on lab

# Set up your kubernetes cluster – 3 node on Ubuntu 

Step 1 : Add the Docker Repository on all three servers 

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

```
Add the Kubernetes repository on all three servers.
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

```
Step 2 : Install Docker, Kubeadm, Kubelet, and Kubectl on all three servers.
```
sudo apt-get update
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.12.2-00 kubeadm=1.12.2-00 kubectl=1.12.2-00
sudo apt-mark hold docker-ce kubelet kubeadm kubectl

Enable net.bridge.bridge-nf-call-iptables on all three nodes.
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

```

Step 3 : On only the Kube Master server, initialize the cluster and configure kubectl.

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

Step 4 : Install the flannel networking plugin in the cluster by running this command on the Kube Master server.

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

```

The kubeadm init command that you ran on the master should output a kubeadm join command containing a token and hash. You will need to copy that command from the master and run it on both worker nodes with sudo.

```
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```

Now you are ready to verify that the cluster is up and running. On the Kube Master server, check the list of nodes.

```
kubectl get nodes

It should look something like this:

NAME                                          STATUS   ROLES    AGE   VERSION
ip-172-31-36-155.us-west-2.compute.internal   Ready    <none>   9h    v1.14.1
ip-172-31-43-227.us-west-2.compute.internal   Ready    master   10h   v1.14.1
ip-172-31-46-75.us-west-2.compute.internal    Ready    <none>   9h    v1.14.1

```

Make sure that all three of your nodes are listed and that all have a STATUS of Ready.



# Set up your kubernetes cluster – 3 node on Centos 7 

We will briefly go through how to bootstrap a cluster using CentOS 7 servers.

Step 1) Turn off swap on all servers.
```
sudo swapoff -a
sudo vi /etc/fstab
```

Look for the line in /etc/fstab that says /root/swap and add a # at the start of that line, so it looks like: #/root/swap. Then save the file.

Step 2) Install and configure Docker.

```
sudo yum -y install docker
sudo systemctl enable docker
sudo systemctl start docker
```

Step 3 ) Add the Kubernetes repo.

```
cat << EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

Turn off selinux.
```
sudo setenforce 0
sudo vi /etc/selinux/config
```

Step 4) 

Change the line that says SELINUX=enforcing to SELINUX=permissive and save the file.

Install Kubernetes Components.

```
sudo yum install -y kubelet kubeadm kubectl
sudo systemctl enable kubelet
sudo systemctl start kubelet
```
Step 5)  Configure sysctl.

```
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

Step 6) Initialize the Kube Master. Do this only on the master node.

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Step 7) Install flannel networking.
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

The kubeadm init command that you ran on the master should output a kubeadm join command containing a token and hash. You will need to copy that command from the master and run it on all worker nodes with sudo.
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash

Step 8) Now you are ready to verify that the cluster is up and running. On the Kube Master server, check the list of nodes.
```
kubectl get nodes
```

It should look something like this:

```
NAME                      STATUS   ROLES    AGE     VERSION
pnair4c.mylabserver.com   Ready    master   3m36s   v1.12.2
pnair5c.mylabserver.com   Ready    <none>   23s     v1.12.2
```

Make sure that all of your nodes are listed and that all have a STATUS of Ready.
