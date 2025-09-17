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

### Working with multiple contexts

KUBECONFIG=config-devk8s:config-prodk8s kubectl config view --merge --flatten > ~/.kube/config

kubectl config get-contexts

or use kubectx my-context-name

kubectl config rename-context kubernetes-admin@kubernetes local-k8s-cluster

kubectl config set-context local-k8s-cluster --cluster=kubernetes --user=kubernetes@admin --namespace=dev


### Install Calico network plugin

remove default nw plugin 

```
kubectl delete -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
kubectl delete namespace kube-flannel
```

cleanup the nw interfaces

```
sudo ip link delete flannel.1 2>/dev/null || true
sudo ip link delete cni0 2>/dev/null || true
sudo rm -rf /etc/cni/net.d/*.conflist 2>/dev/null || true
sudo systemctl restart cri-docker
sudo systemctl restart docker
```
### Install Calion now

Install Calico operator
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```
Configure Calico
```
cat <<EOF | kubectl apply -f -
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
EOF
```
Check Calico pods
```
kubectl get pods -n calico-system
```
Check nodes are ready
```
kubectl get nodes
```
Test with sample deployment
```
kubectl create deployment test-nginx --image=nginx --replicas=2
kubectl get pods -o wide
kubectl delete deployment test-nginx
```
### Now, Lets say, I want to uninstall Calico. Here you

```
kubectl delete installation default
kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```
Cleaning up the resources

Delete namespaces
```
kubectl delete namespace calico-system
kubectl delete namespace tigera-operator
```
Remove CRDs
```
kubectl delete crd $(kubectl get crd | grep -E '(calico|tigera)' | awk '{print $1}')
```

Run on all nodes

```
sudo ip link delete tunl0 2>/dev/null || true
sudo rm -rf /etc/cni/net.d/10-calico.conflist
sudo rm -rf /var/lib/calico
sudo systemctl restart cri-docker
sudo systemctl restart docker
sudo systemctl restart kubelet
```

### Finally, verify removal

Check no Calico pods remain
```
kubectl get pods -A | grep calico
```
Check no Calico namespaces
```
kubectl get namespaces | grep -E '(calico|tigera)'
```
Check nodes status
```
kubectl get nodes
```




