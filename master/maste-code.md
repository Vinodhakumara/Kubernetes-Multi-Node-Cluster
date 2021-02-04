`yum install docker -y`

`systemctl enable docker --now`

`cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo`
>**[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
**

`yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`

`systemctl enable --now kubelet`

`kubeadm config images pull`

`kubeadm init --pod-network-cidr=10.240.0.0/16`

`docker info | grep -i cgroup`

`systemctl restart docker`

`cat  /etc/docker/daemon.json`
>{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
`docker info | grep -i cgroup`

`yum install iproute-tc`

`kubeadm init --pod-network-cidr=10.240.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem`

>`mkdir -p $HOME/.kube`
`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
`sudo chown $(id -u):$(id -g) $HOME/.kube/config`

`kubectl get pods --all-namespaces`

`echo 3 > /proc/sys/vm/drop_caches`

` kubeadm token create  --print-join-command`

`kubectl apply  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

`kubectl get pods --all-namespaces`

`kubectl get nodes`

<h2>TroubleShooting of CoreDNS pods:</h2>

<h3>At master node:<h3>

`kubectl get pods --all-namespaces`
`kubectl logs coredns-74ff55c5b-c45jz -n kube-system`

##After checking the following:
Flannel interfaces existed on each node
The routes on each table were checked to ensure each other servers subnet was present and I could ping flannel interfaces on other servers
`Kublet was running with the parameter –network-plugin=cni`
`kube-controller-manager` was running with the parameters `–allocate-node-cidrs=true` and  `–cluster-cidr`
`ps -aux | grep  kube-controller-manager  | grep cidr`
>root      8048  0.8  6.2 816460 63408 ?        Ssl  16:59   1:05 kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf --bind-address=127.0.0.1 --client-ca-file=/etc/kubernetes/pki/ca.crt --cluster-cidr=10.240.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt --cluster-signing-key-file=/etc/kubernetes/pki/ca.key --controllers=*,bootstrapsigner,tokencleaner 

