<h1>Slave setup</h1>

`yum install docker -y`

`systemctl enable docker --now`

`cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo`
>[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl

`yum install -y  kubectl kubelet  kubeadm  --disableexcludes=kubernetes`
`systemctl enable kubelet --now`

`docker info | grep -i cgroup`
 >Cgroup Driver: cgroupfs

`cat  /etc/docker/daemon.json`
>{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

`systemctl restart docker`

`docker info | grep -i cgroup`
>Cgroup Driver: systemd

`yum install iproute-tc`

`cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf`
>net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

`sysctl --system`

`kubeadm join 172.31.40.209:6443 --token 3joamn.y8asi8hw2jb41xx5     --discovery-token-ca-cert-hash sha256:86dd3714b1168b8de0abcb63300a458baf838860ff9c6f53bd85707853ff4db7`

`systemctl status kubelet -l`
`echo 3 > /proc/sys/vm/drop_caches`
