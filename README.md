# Kubernetes_training
LOG into AWS console 
1. (AWS CLI on all OS:      https://aws.amazon.com/cli/).  This tool should be downloaded on your local workstation.
A. Create I am user with its access key and secret access key and the .pem key pair.

B.  Create 3 nodes, One for master and 2 for worker-nodes
Then ssh into the nodes from your workstation

OR USING VAGRANT VIRTUAL MACHINE

Commands for both master and worker nodes with a root user 
     Install cdrivergroup
sudo apt-get update
sudo apt install apt-transport-https ca-certificates  curl gnupg-agent software-properties-common  -y

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

      Configure Ip tables to see the Bridged Traffic
sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

vi /etc/sysctl.conf
Then add the below command inside the file
net.bridge.bridge-nf-call-iptables = 1
sudo sysctl -p

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
sudo apt-get update

   Install the transport
 sudo apt-get install -y apt-transport-https ca-certificates curl
 sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/   kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

 Temporary disable swap in the nodes for kubelet to work fine       sudo swapoff -a               
       
   Update fstab so that swap remains disabled after a reboot
 sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
 
    Install Kubeadm, kubelet kubectl and containerd for CRI
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl containerd

sudo systemctl daemon-reload 
sudo  systemctl enable --now containerd
sudo systemctl status containerd
q

    Run these commands on Master Node ONLY as a root user
  kubeadm init  
- -pod-network-cidr=192.168.0.0/16 (Optional)         // Initialises kubeadm with calico Network

Start using using kuberenets cluster
mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

       Checking for opened ports in the cluster 
ping <target hostname >  -c2      to get the ip of the server and copy the ip to use with nc command 
nc -v <nodeIp> 6443
Nectat <nodeIp> 6443


    Configure CNI with calico network
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

(Optional)
Kube init  —pod-network-cidr=192.168.0.0/16    (Optional)      Initialise the kubeadm with calico Network 

Kube init  —pod-network-cidr=10.30.0.0/12    (Optional)      Initialise the kubeadm with Weave Network 

  Configure CNI with weave network
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

 Run the join command only on the worker nodes as a root user 

Make a note of the kubeadm join command form the master for the token generated and keep the token safe 
Then copy and paste the join command to all the worker nodes to join the cluster as root user.

NOTE:
But the token expires after 24 hours so use this below command to re-generate the token

 try re-running the below commands to reset and initialise the kubeadm
kubeadm reset
Kubeadm init

And if it has passed 5 mins means it has delayed so use this command to get both the token and SHA
kubeadm token create  - -print-join-command
kubeadm token list        To verify 

sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

set it to the default KUBECONFIG location
export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

Or use this if the list one doesn’t work 
Kubeadm init    —apiserver-advertise-address=<MasterIP>  - -pod-network-cidr=192.168.0.0/16.

Then copy the kubeadm join command again

Commands using kubectl with the cluster 
Kubectl get pods 
Kubectl get deployment  <PodName> -o wide 
Kubectl get nodes
Kubectl get service <serviceName>
Kubectl edit svc <ServiceName>
Kubectl get pods —all-namespaces
Kubectl describe pod <podName>
Kubectl delete deployment <DepolymentName>

