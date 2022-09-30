KUBERNETES installation with kubeadm for production environment
RUN ON MASTER AND WORKER NODES

sudo su -
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu (lsb_release -cs) stable"
sudo apt-get install docker.io -y
sudo systemctl status docker
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay br_netfilter
EOF

sudo modprobe overlay && sudo modprobe br_netfilter

echo 1 > /proc/sys/net/ipv4/ip_forward

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo ufw disable 
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

sudo sysctl --system

sudo vim /etc/fstab   ……..Comment on the swap line and quit

vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
ENVIRONMENT="KUBELET_EXTRA_ARGS=--fall-swap-on=false"

vi /etc/default/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs"

sudo apt install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl containerd

ONLY on Master Node as a root user

sudo kubeadm init

ISSUE THES COMANDS A REGULAR USER
To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

USING CALICO NETWORK
sudo kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml

check if the system has .kube/ directory and check if it contains config file 
ll or ls -al
cd .kube/

vi /etc/docker/daemon.json
ADD THE BELOW CONTANT INSIDE THE FILE 

 {
   "exec-opts":["native.cgroupdriver=cgroupfs"],
    "log-driver":"json-file",
    "log-opts":{
     "max-size":"100m"
      },
     "storage-driver":"overlay2"
   }
NOTE: if cgroupfs is not working try with system or systemd

systemctl daemon-reload
systemctl restart kubelet
systemctl status kubelet

kubectl version --client
kubectl cluster-info dump

Run the join command only on the worker nodes as a root user
 
NOTE:
But the token expires after 24 hours so use this below command to re-generate the token

 try re-running the below commands to reset and initialise the kubeadm
kubeadm reset
Kubeadm init

And if it has passed 5 mins means it has delayed so use this command to get both the token and SHA
kubeadm token create  - -print-join-command
kubeadm token list        To verify 


     
     
     
 OPTIONAL    
    Configure CNI with calico network
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


Kube init  —pod-network-cidr=192.168.0.0/16    (Optional)      Initialise the kubeadm with calico Network 

Kube init  —pod-network-cidr=10.30.0.0/12    (Optional)      Initialise the kubeadm with Weave Network 

  Configure CNI with weave network
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

 Run the join command only on the worker nodes as a root user 

Make a note of the kubeadm join command form the master for the token generated and keep the token safe 
Then copy and paste the join command to all the worker nodes to join the cluster as root user.





Commands using kubectl with the cluster 
Kubectl get pods 
Kubectl get deployment  <PodName> -o wide 
Kubectl get nodes
Kubectl get service <serviceName>
Kubectl edit svc <ServiceName>
Kubectl get pods —all-namespaces
Kubectl describe pod <podName>
Kubectl delete deployment <DepolymentName>

