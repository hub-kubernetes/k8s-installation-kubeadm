# k8s-installation-kubeadm

##  Pre-requisite 

*     Master node - 2 cpu x 2 GB memory
*     Worker node - 1 cpu x 1 GB memory

##  The below steps will be performed on both master and worker node 

##  1.  Turn of Swap

`   apt-get update`

##  2.  Comment swap FS from /etc/fstab 

`   vi /etc/fstab`

`   Comment any line that has swap written` 

##  3.  Edit /etc/hostname and edit hostname to match the host of your choice 

##  4.  Get private ip address of all hosts 

`   ip addr ` 

##  5.  Edit /etc/hosts to add hostname and IP address on all nodes 

`   vi /etc/hosts ` 

~~~
    kmaster 192.168.0.2  -- private IP address from previous step
    knode1  192.168.0.3  -- provate IP address from previous step 
~~~

##  6.  Install openssh server if not available

`   sudo apt-get install openssh-server ` 

##  7.  Install curl and apt-transport-http utilities 

`   apt-get update && apt-get install -y apt-transport-https curl `

##  8.  Add gpg keys of cloud packages 

`   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - `

##  9.  Add kubernetes repository 

~~~
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF
~~~

##  10. Install docker, kubeadm, kubelet, kubectl 

`   sudo -i ` 

`   apt-get update && apt-get install -y docker.io kubelet kubeadm kubectl ` 

##  Add cgroup driver to kubeadm.conf

`   vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf ` 

`   Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs" ` 


#   The below steps should be run only on master 

##  1.  Run kubeadm init 

`   kubeadm init --apiserver-advertise-address=<ip-address-MASTER-vm> --pod-network-cidr=192.168.0.0/16 ` 

##  2.  Preserve the output of the previous command 

~~~
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

##  3.  Run kubeadm init on worker nodes as per instructions 

##  4.  Install Calico CNI
~~~

    kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
    kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

~~~









