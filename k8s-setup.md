# k8s master setup
```console
swapoff -a
sysctl -w net.ipv4.ip_forward=1

apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update

apt-get install -y kubelet kubeadm kubernetes-cni

kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```
# if you want single node mode (runing apps in master node)
```console
kubectl taint nodes --all node-role.kubernetes.io/master-
```
done!

# multi node setup
on worker install k8s :
```console
swapoff -a
sysctl -w net.ipv4.ip_forward=1

apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update

apt-get install -y kubelet kubeadm kubernetes-cni
```

on master
```console
kubeadm token create --print-join-command
```

come back to worker paste command Ex
```console
kubeadm join 192.168.56.110:6443 --token hp9b0k.1g9tqz8vkf78ucwf     --discovery-token-ca-cert-hash 
```
