# -*- mode: ruby -*-
# vi: set ft=ruby :

# Nodes list
nodes = [
  {
    :name => "k8s-master",
    :type => "master",
    :eth1 => "172.16.100.10"
  },
  {
    :name => "k8s-node-0",
    :type => "node",
    :eth1 => "172.16.100.20"
  },
  {
    :name => "k8s-node-1",
    :type => "node",
    :eth1 => "172.16.100.30"
  }
]

# Script to install k8s using kubeadm with docker as container runtime interface
$configureVM = <<-'SCRIPT'

echo "configure /etc/hosts file..."
cat <<EOF >> /etc/hosts
172.16.100.10 k8s-master
172.16.100.20 k8s-node-0
172.16.100.30 k8s-node-1
EOF

echo "set vagrant user's password..."
echo 'vagrant:vagrant' | chapsswd

echo "disable swap..."
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo "let iptables see bridged traffic..."
cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

echo "install dependencies..."
apt update
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

echo "set up docker repository..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "install docker..."
apt update
apt install -y docker-ce docker-ce-cli containerd.io

echo "configure docker daemon..."
mkdir -p /etc/docker
cat <<EOF | tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

echo "restart docker and enable on boot..."
systemctl enable docker
systemctl daemon-reload
systemctl restart docker

echo "verify docker engine..."
docker run hello-world

echo "add vagrant user in docker group..."
usermod -aG docker vagrant

echo "set up k8s repository..."
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo \
"deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" \
| tee /etc/apt/sources.list.d/kubernetes.list

echo "install kubeadm, kubelet, kubectl..."
apt update
apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

SCRIPT

# Script to configure K8s master node
$configureMaster = <<-'SCRIPT'

echo "configuring master node..."
IP_ADDR=$(ifconfig enp0s8 | grep netmask | awk '{print $2}')
HOST_NAME=$(hostname -s)

echo "initialize kubeadm..."
kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=192.168.0.0/16

echo "configure kubectl..."
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
cp -i /etc/kubernetes/admin.conf /vagrant/
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

echo "install calico as the pod network add-on..."
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

echo "create kubeadm token..."
kubeadm token create --print-join-command > /tmp/kubeadm_join_cmd.sh

echo "enable passwordless authentication in SSH..."
sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
service sshd restart

SCRIPT

# Script to configure K8s worker node
$configureNode = <<-'SCRIPT'

echo "configuring worker node..."
apt install -y sshpass
sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@k8s-master:/tmp/kubeadm_join_cmd.sh /tmp/
bash /tmp/kubeadm_join_cmd.sh
SCRIPT

Vagrant.configure("2") do |config|

  nodes.each do |node|

    config.vm.define node[:name] do |config|

      config.vm.box = 'ubuntu/bionic64'
      config.vm.box_version = '20210407.0.0'
      config.vm.hostname = node[:name]
      config.vm.network :private_network, ip: node[:eth1]

      config.vm.provider "virtualbox" do |v|
        v.name = node[:name]
        v.memory = 2048
        v.cpus = 2
      end

      config.vm.provision "shell", inline: $configureVM

      if node[:type] == "master"
          config.vm.provision "shell", inline: $configureMaster
      else
          config.vm.provision "shell", inline: $configureNode
      end

    end

  end

end 