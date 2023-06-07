### k8s-cluster-setup Master nodes

* Create 3 Ubuntu instances(2 cpu, 4 GB Memory) with open the below port in SG

`Master Node Ports: 2379,6443,10250,10251,10252`
`Worker Node Ports: 10250,30000–32767`
`Default port range for NodePort Services -30000–32767`

* And using the `containerd` daemon, It is designed to be lightweight and efficient and is often used as an alternative to the Docker daemon
```
#Install containerd 
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y containerd.io

#Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
* Then, we're going to install kubeadm, kubectl, and kubelet into the master,
```
#install kubeadm, kubectl, kubelet,and kubernetes-cni 
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository -y "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt -y install kubeadm kubelet kubectl kubernetes-cni
```
* Next set kernel parameters in preparation for running Kubernetes:
```
#Load the br_netfilter module in the Linux kernel
sudo modprobe br_netfilter

#switch to root to 
sudo su 
echo 1 > /proc/sys/net/ipv4/ip_forward
exit
```
* Then we're going to initialize the Kubernetes cluster:
`sudo kubeadm init --pod-network-cidr=10.244.0.0/16`

* From the output we will get the JOIn command with token, we should SAVE THIS JOIN COMMAND to run on worker nodes later to attache the worker node to master.
The statement looks like below syntax,
`#Command to run on worker nodes
kubeadm join <control-plane-ip>:6443 --token <token> \
 --discovery-token-ca-cert-hash <hash>`
 
 * Then we should copy the kube configuration files
 ```
 #Allow kubectl to interact with the cluster
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#export KUBECONFIG as root
sudo su
export KUBECONFIG=/etc/kubernetes/admin.conf
exit
```
* Finally, we need to set up the network for Kubernetes,
```
#Install CNI Flannel
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```
* After kubeadm init completes, you will have a working Kubernetes cluster ready to run applications. You can then use kubectl it to deploy and manage applications on the cluster., you should see the Control Plane node when you run `kubectl get nodes`.
![k8s-nodes](https://github.com/Jishnu-CJ/k8s-cluster-setup/assets/89075369/541d0e73-92c4-4f41-a646-335c0c9b2908)
![ClusterSetup](https://github.com/Jishnu-CJ/k8s-cluster-setup/assets/89075369/69a57fb6-03e8-49cb-bd5e-b0e3b8f23d46)


### Building the Worker node

* For this first part, we can copy and paste the below commands into the terminal,
```
#update server and install apt-transport-https and curl
sudo apt-get update -y
sudo apt install -y apt-transport-https curl

#Install containerd 
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y containerd.io

#Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

#set SystemdCgroup = true within configs.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

#Restart containerd daemon
sudo systemctl restart containerd

#Enable containerd to start automatically at boot time
sudo systemctl enable containerd

#install kubeadm, kubectl, kubelet,and kubernetes-cni 
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository -y "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install -y kubeadm kubelet kubectl kubernetes-cni

#disable swap
sudo swapoff -a

#load the br_netfilter module in the Linux kernel
sudo modprobe br_netfilter

#enable ip-forwarding 
sudo su 
echo 1 > /proc/sys/net/ipv4/ip_forward
exit
```
* Finally,we should run the command that we saved when running Kubernetes on the Control Plane:
```
#Command to run on worker nodes
kubeadm join <control-plane-ip>:6443 --token <token> \
 --discovery-token-ca-cert-hash <hash>
 ```
 * If the joins have been done correctly on both worker nodes, we'll see the following result,
 ![k8s-nodes](https://github.com/Jishnu-CJ/k8s-cluster-setup/assets/89075369/541d0e73-92c4-4f41-a646-335c0c9b2908)

 
