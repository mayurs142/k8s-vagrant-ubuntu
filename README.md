# vagrant-k8s-cluster

Vagrant script for setting up a 3-node kubernetes cluster using kubeadm on ubuntu nodes

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
