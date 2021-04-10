# vagrant-k8s-cluster

Vagrant script for setting up a 3-node kubernetes cluster using kubeadm on ubuntu nodes

## prerequisities

1. [VirtualBox][1]
2. [Vagrant][2]

## how to use

```bash
# clone the repo
git clone https://github.com/mayurs142/vagrant-k8s-cluster.git

# change directory
cd vagrant-k8s-cluster

# vagrant up
vagrant up
```

## managing multiple kubeconfig and contexts

```bash
# Merge 2 k8s config files into one
KUBECONFIG=~/.kube/config:admin.conf kubectl config view --merge --flatten > /tmp/config

# Move the merged config to the default kubeconfig
mv /tmp/config ~/.kube/config

# List contexts
kubectl config get-contexts

# Switch between contexts
kubectl config use kubernetes-admin@kubernetes

# Delete contexts from kubeconfig
kubectl config delete-context kubernetes-admin@kubernetes
```

[1]: https://www.virtualbox.org/wiki/Downloads
[2]: https://www.vagrantup.com/downloads
