# Multi Node K8S-Cluster-On-AWS
In This doc you will be seeing how we can create a `Multi Node Kubernetes` cluster on AWS. You need to launch the instances on AWS. Here i have launched `t2.micro` which is under free tier so it will not charge. Now as you can see in the below images that i have launched 1 isntance as `Master Node` and `Salve Node`.

![bn](https://github.com/Vinodhakumara/Kubernetes-Multi-Node-Cluster/blob/master/slave/Screenshot%202021-02-05%2000.10.08.png) 

Now as the instances are launched then we can now configure the kubernetes setup in both of the instances. we will go one at a time. first let's ready the master with [Kubernetes](https://kubernetes.io/).

> ## Master Setup
>Install Docker as I am using `Amazon Linux` ami so install -y kubelet kubeadm kubectl --disableexcludes=kubernetes. you can install `yum install docker`![](master/Install%20Dorcker%20on%20master.png)
After installing docker you can check the status and make the docker enable by `systemctl enable docker --now`. So, every time when we start the OS it will be already in the started state .
![](master/Enable%20Docker%20n%20start.png)
*Now we can proceed with Installing kubeadm kubelet and kubectl*
1.`kubelet`: Kubelet is a primary **node agent** that runs on each node. It can register the node with the API server using hostname,flags,etc
2.`kubeadm`: Kubeadm is a tool built to provides us to connect to a cluster.

>You can install kubectl of clients by installing kubeadm using `yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`. ![](master/Install%20Kubesdm%2Cctl%2Clet%20etc.png)
Enable kubelet by using `systemctl enable kubelet`
![](master/Enable%20kubelet.png)
As we can see in the above image, the kubelet is trying to activate not able to start.

>If now we try to install we face troubleshoot ![](master/kubeladm%20init%20-%20Troubleshoot.png)
Now we needs to pull images of different types of resources from *kubeadm*. This will pull required images and lunchs some pods using `kubeadm config images pull`.
![](master/Pull%20config%20kube%20images.png)
Kuberenets has to run differmt process behind the scene. So, to install these resources it uses containers and in the containers we will be having the required resources eg.

1.kube-apiserver:v1.20.2

2.kube-controller-manager:v1.20.2

3.kube-scheduler:v1.20.2

4.kube-proxy:v1.20.2

5.pause:3.2

6.etcd:3.4.13-0

7.coredns:1.7.0


Now we need to change a driver default docker uses `cgroup` need to change to `systemd` by `cat > /etc/docker/daemon.json` inside that add these code `{ "exec-opts": ["native.cgroupdriver=systemd"] }`.After changing the driver you can now restart the docker using `systemctl restart docker` so that what we have done changes in the daemon.json file that should be loaded. Now after restarting if you check the driver `docker info | grep Driver` then you will find that the driver is now updated from cgroupfs to systemd. 
![](master/Change%20cgroup%20to%20systemd.png)

Now we can proceed further with installing `yum install iproute-tc` this will require to set the routing path.
![](master/Install%20IProute-tc.png)
Now we need to set the bridge routing to enable(1) using `echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables`
After that we are set toinitialize the the master with respective pod-network-cidr. You can initialize using `kubeadm init --pod-network-cidr=10.240.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem`

**--pod-network-cidr=10.240.0.0/16**: containers will be launched within this range of network
**--ignore-preflight-errors=NumCPU**: As we are using t2.micro i.e 1GB ram and 1 CPU so to ignore the errors of CPU we using this command
**--ignore-preflight-errors=Mem**: We have only 1Gb Ram so to ignore errors on mem we using this command
![](master/bridge%20iptable%20to%201%20and%20run%20kubeadm%20init.png)

After intializing master it will give you command to run. Below are the commads tha you need to run in the nodes(os) you wanted to make them salves.

>`mkdir -p $HOME/.kube`
`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
`sudo chown $(id -u):$(id -g) $HOME/.kube/config`

Once we run those command Kubernetes creates `.kube` from the above first command and inside that we need to create `config` file or configuration file ,Second commadn is to change owner permisio of `config` file inside `.kube`.
![](master/Copy%20from%20admin%20to%20kube%20config.png)

> ## Slave setup

The same process you need to while creating `slave1`. So first we need to install docker. `yum install docker`. ![](slave/slave-%20Docker%20Install.png)

after installing docker we need to make docker enable. `systemctl enable docker --now`.
After that we need to create a repository of kubernetes vi /etc/yum.repos.d/kubernetes.repo and write the command in that file

`[kubernetes] name=Kubernetes baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch enabled=1 gpgcheck=1 repo_gpgcheck=1 gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg exclude=kubelet kubeadm kubectl`
After saving this you can check the kubernetes repository is configured or not using `yum repolist`.
![](slave/slave-Enable%20docker%20%2Cconfig%20yum%20kube.png)

Now we can proceed to install kubeadm. ensure kubeadm will install kubectl and kubelet also so now need to worry while installing kubectl and kubelet. you can install kubecadm using `yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`. Even in slave we need to install kubeadm bcouse that helps to join to master.
![](slave/slave-install%20kubelet%2Cctl%2Cadm%20etc.png)

Here we are enabling Kubelet using `systemctl enable kubelet --now` and pulling required images using `kubelet config images pull`
![](slave/slave-enable%20kubelet%20and%20pull%20image%20kubelet.png)

As you can see the images pulled by kubeadm.


1.kube-apiserver:v1.20.2

2.kube-controller-manager:v1.20.2

3.kube-scheduler:v1.20.2

4.kube-proxy:v1.20.2

5.pause:3.2

6.etcd:3.4.13-0

7.coredns:1.7.0


Now we need to change a driver default docker uses `cgroup` need to change to `systemd` by `cat > /etc/docker/daemon.json` inside that add these code `{ "exec-opts": ["native.cgroupdriver=systemd"] }`.After changing the driver you can now restart the docker using `systemctl restart docker` so that what we have done changes in the daemon.json file that should be loaded. Now after restarting if you check the driver `docker info | grep Driver` then you will find that the driver is now updated from cgroupfs to systemd. 
![](slave/slave-cgroup%20to%20systemd.png)

after that you need to restart docker so make the chnages successful. `systemctl restart docker`

After that we need to install iproute-tc software. this software is responsible to make the routings inside the master salve setup. To install `yum install iproute-tc`.
![](slave/slave-install%20iptable.png)

Now we need to chamge the iptable using this command `echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables`.
![](slave/slave-enable%20bridge%20iptable.png)

Take the token from *master* to enter into a *slave system* and to make a connection (key).
![](master/get%20token%20from%20master.png)


Now you can proceed with joining the slave with master. You can use the token given while initializing master. This tokens is given at one time m=by the master but you can create new token if you forget to save this tokens.
![](slave/slave-join%20to%20master.png)

As you can see we have master and slave node as well. To make this ready we need to make the *overlay connection* between master and slave. we need to use flannel plugin. flannel is a plugin that gives a facility of *overlay network*. Using `https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
` and then run `kubectl get pods --all-namespaces`.
>Display Nodes(Slave node,master node) details. ![](master/master-show%20nodes.png)
now we can see all nodes are in ready state.

So let's lauch a pod in the nodes. Here i am cretaing deployment just for testing. `kubectl create deployments myd --image=vinod2681997/myweb` this image has preconfigured with `apache webserver`.
![](master/master-create%20deployment.png)

As you can see that the deployment's pods is launched inside the slave1 node and its working fine. So now lets check the application running inside the pods by exposing the respective pod. you can expose deployments using kubectl expose deployments myd --type=NodePort --port=80. Now the deployment's pods will be exposed to a outside world.
![](master/master-expose%20deploy.png)

>Take Slave public `IP from AWS Instance` and `PORT` number of Slave container service. ![](master/master-take%20port%20number%20of%20slave.png)

As you can see the deployment's pods is now exposed on the port 31893. So using the public ip address of the slave1 node with port 31893 you see the application running on node slave1.
![](slave/slave-final%20output.png)
