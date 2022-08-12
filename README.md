# Kubernetes_training
Configuring Kubernetes cluster in AWS using Kubeadm in linux 
LOG into AWS console 
1. (AWS CLI on all OS:      https://aws.amazon.com/cli/).  This tool should be downloaded on your local workstation.
A. Create I am user with its access key and secret access key and the .pem key pair.

B.  Create 3 nodes, One for master and 2 for worker-nodes
Then ssh into the nodes from your workstation

OR USING VAGRANT VIRTUAL MACHINE

Run below Commands for both master and worker nodes as a root user 
     
sudo apt-get update
sudo apt install apt-transport-https ca-certificates  curl gnupg-agent software-properties-common  -y

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

Configure Ip tables to see the Bridged Traffic
sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf

sudo cat <<EOF | sudo tee /etc/modules-load.d/containerd.confi
overlay
br_netfilter
EOF

sudo modprobe overlay 
sudo modprobe br_netfilter


cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF


sudo sysctl --system
sudo apt-get update

   Install the transport
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/   kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

 Temporary disable swap in all nodes for kubelet to work fine  and firewall  
sudo ufw disable  sudo swapoff -a               
       
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
  
sudo su -
kubeadm init  

NOTE: you will mostly like to encounter this error 
       error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher

SOLVING THE ERROR using the below commands:
sudo su -
kubeadm reset 
echo 1 > /proc/sys/net/ipv4/ip_forward

Or 
sudo sysctl -w net.ipv4.ip_forward=1

sudo swapoff -a 

sudo kubeadm init

mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

 Configure CNI with calico network
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml

Or with weave network


Or with Flannel network

wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

 Run the join command only on the worker nodes as a root user 

Make a note of the kubeadm join command form the master for the token generated and keep the token safe 
Then copy and paste the join command to all the worker nodes to join the cluster as root user.

If joining the worker nodes to master nodes fails with this error accepts at most 1 arg(s), received 3 To see the stack trace of this error execute with --v=5 or higher
then use the below command to recreate a new token on master node 

sudo su -
kubeadm token create --print-join-command



sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml
systemctl status containerd 

sudo kubeadm config images pull --cri-socket /run/containerd/containerd.sock --kubernetes-version v1.24.3

       Checking for opened ports in the cluster 
ping <target hostname >  -c2      to get the ip of the server and copy the ip to use with nc command 
nc -v <nodeIp> 6443
Nectat <nodeIp> 6443

NOTE:
But the token expires after 24 hours so use this below command to re-generate the token

 try re-running the below commands to reset and initialise the kubeadm as a root user 
kubeadm reset
kubeadm init
And repeat the process

And if it has passed 5 mins means it has delayed so use this command to get both the token and SHA
kubeadm token create  - -print-join-command
kubeadm token list        To verify 
