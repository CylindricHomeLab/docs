---
title: 'Kubernetes Setup'
taxonomy:
    category:
        - Tech
    tag:
        - kubernetes
        - containers
theme: knowledge-base-cyl
---

Some setup notes for Kubernetes.

===

Build 3 servers to act as nodes.

Install `kubelet`, `kubeadm` and `kubectl` on them all:

    #!/bin/sh
    # https://medium.com/@KevinHoffman/building-a-kubernetes-cluster-in-virtualbox-with-ubuntu-22cd338846dd
    
    yum update -y
    yum install -y docker
    systemctl enable docker && systemctl start docker
    
    # kubaadm doesn't like swap.
    swapoff -a
    sed -e '/swap/ s/^#*/#/' -i /etc/fstab
    mount -a
    
    # fixes [WARNING FileExisting-crictl]
    yum install -y golang git
    go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
    
    # fixes [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system
    
    
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
    yum install -y kubelet kubeadm kubectl
    systemctl enable kubelet && systemctl start kubelet

On the first node, create a new Kubernetes cluster. Make a note of the `TOKEN` used, as that will need to be the same on all nodes.

    HOSTNAME=`hostname --short`
    TOKEN=6rt019.qf7ilzd6tsawf7t6
    MASTERNODE_ADDRESS=192.168.50.11
    POD_SUBNET=182.168.50.0/24
    
    kubeadm init --token=$TOKEN --apiserver-advertise-address=$MASTERNODE_ADDRESS--pod-network-cidr=$POD_SUBNET
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config
    
    # Install the CNI plugin Calico
    kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml
    
    # Install dashboard - https://github.com/kubernetes/dashboard/wiki/Creating-sample-user
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    

On all subsequent nodes, simply join the cluster:

    TOKEN=6rt019.qf7ilzd6tsawf7t6
    MASTERNODE_ADDRESS=192.168.50.11
    
    kubeadm join $MASTERNODE_ADDRESS:6443 --token $TOKEN --discovery-token-unsafe-skip-ca-verification

- [https://medium.com/@KevinHoffman/building-a-kubernetes-cluster-in-virtualbox-with-ubuntu-22cd338846dd](https://medium.com/@KevinHoffman/building-a-kubernetes-cluster-in-virtualbox-with-ubuntu-22cd338846dd)
- [https://github.com/kubernetes/dashboard/wiki/Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)

The cluster can be verified with:

    kubectl cluster-info

Start the proxy on the master node:

    kubectl proxy