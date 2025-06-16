# k8s-multinode-kubeadm
k8s-multinode-kubeadm for local setup

Container Runtime:
cri-o

Network: 
Calico

### To re-generate token to let the worker nodes join the master node.
kubeadm token create --print-join-command

### Labeling the nodes
kubectl label node k8s-worker2 node-role.kubernetes.io/worker=worker

### Metrics 
kubectl top nodes

kubectl top pods -n kube-system
