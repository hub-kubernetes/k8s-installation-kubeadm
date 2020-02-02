# k8s-installation-kubeadm

##  Pre-requisite 

*     Master node - 2 cpu x 2 GB memory
*     Worker node - 1 cpu x 2 GB memory

## Pre-installation steps

###  The below steps will be performed on both master and worker node 

*  1.  Turn of Swap

`   apt-get update`

`   swapoff -a ` 

*  2.  Comment swap FS from /etc/fstab 

`   vi /etc/fstab`

`   Comment any line that has swap written` 

*  3.  Edit /etc/hostname and edit hostname to match the host of your choice 

*  4.  Get private ip address of all hosts 

`   ip addr ` 

*  5.  Edit /etc/hosts to add hostname and IP address on all nodes 

`   vi /etc/hosts ` 

~~~
    kmaster 192.168.0.2  -- private IP address from previous step
    knode1  192.168.0.3  -- provate IP address from previous step 
~~~


## Installation Procedure 

### Install kubelet kubeadm and kubectl 

* **Ubuntu**

```
apt-get update && apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

apt-get install -y kubelet kubeadm kubectl

apt-mark hold kubelet kubeadm kubectl

```

* **Centos**

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

setenforce 0

sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet

```

### Install Docker 

* **Ubuntu**

```
sudo apt-get update

sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

* **Centos**

```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install -y docker-ce docker-ce-cli containerd.io --nobest

systemctl start docker

```


### Configure Cluster using kubeadm

**The below steps will be performed** __**ONLY ON MASTER NODE**__

* Get the IP address of master

```
ip addr
```


* Initialize the cluster

```
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address={{IP_ADDR_MASTER}}

```

* Your output should look like below - 

```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.138.0.2:6443 --token 3ccgnq.2owa1scoiqqoqhdq \
    --discovery-token-ca-cert-hash sha256:04ff7a9148ae02db8a4be70f016fe66d6d5870ea09641e48f6a7db54747e1acd 

```

Preserve the above output as it contains the token required for node configuration. 

* Copy over the configuration files

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```


* Install Networking component (CNI)

```
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml

```



* Join the worker node 


The below steps are to be run __**only on the worker node**__ 

The output of the kubeadm init command will provide the kubeadm join as its output. Run the kubeadm join command on the worker nodes. 



```
kubeadm join 10.138.0.2:6443 --token 3ccgnq.2owa1scoiqqoqhdq \
    --discovery-token-ca-cert-hash sha256:04ff7a9148ae02db8a4be70f016fe66d6d5870ea09641e48f6a7db54747e1acd 

```

The output will be as below 

```
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```





