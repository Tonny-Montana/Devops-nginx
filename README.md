
---------------------------------------------------------------------------------------------------------------
apt install bash-completion -y
echo  source /etc/profile.d/bash_completion.sh >>  /root/.bashrc
---------------------------------------------------------------------------------------------------------------

vim /etc/environment

---------------------------------------------------------------------------------------------------------------

http_proxy="http://172.19.22.11:9091"
https_proxy="http://172.19.22.11:9091"
no_proxy=".svc,.default,.local,.cluster.local,localhost,127.0.0.0/8,10.1.16.0/24,10.0.0.0/8,.domain.com,10.96.0.0/12,192.168.0.0/16,172.19.22.0/24"

"localhost,*.svc,*.cluster.local,*.local,*.default,127.0.0.1,10.96.0.0/12,192.168.0.0/16,10.0.0.0/8,172.19.22.0/24"
exit
sudo su

---------------------------------------------------------------------------------------------------------------

sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

---------------------------------------------------------------------------------------------------------------

sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

---------------------------------------------------------------------------------------------------------------

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system


---------------------------------------------------------------------------------------------------------------

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64vim ] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update -y
sudo apt install -y containerd.io

---------------------------------------------------------------------------------------------------------------

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

---------------------------------------------------------------------------------------------------------------

sudo vim /etc/containerd/config.toml

---------------------------------------------------------------------------------------------------------------

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true

---------------------------------------------------------------------------------------------------------------

sudo mkdir -p /etc/systemd/system/containerd.service.d
sudo vim /etc/systemd/system/containerd.service.d/http-proxy.conf

---------------------------------------------------------------------------------------------------------------

[Service]
Environment="HTTP_PROXY=http://172.19.22.11:9091"
Environment="HTTPS_PROXY=http://172.19.22.11:9091"
Environment="NO_PROXY=.svc,.default,.local,.cluster.local,localhost,127.0.0.0/8,10.1.16.0/24,10.0.0.0/8,.domain.com,10.96.0.0/12,192.168.0.0/16,172.19.22.0/24"


---------------------------------------------------------------------------------------------------------------

sudo systemctl daemon-reload
sudo systemctl restart containerd

---------------------------------------------------------------------------------------------------------------


apt update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

---------------------------------------------------------------------------------------------------------------

# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


---------------------------------------------------------------------------------------------------------------


# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


---------------------------------------------------------------------------------------------------------------

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

---------------------------------------------------------------------------------------------------------------


sudo systemctl enable --now kubelet

sudo systemctl enable kubelet

sudo kubeadm config images pull







