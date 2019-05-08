# kubernetes-lab
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

