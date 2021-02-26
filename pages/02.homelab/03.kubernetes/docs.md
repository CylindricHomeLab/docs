---
title: 'HomeLab Kubernetes'
taxonomy:
    category:
        - homelab
    tag:
        - homelab
        - kubernetes
---

## Pre-Reqs

* One master node
* * `kube01` - 172.29.14.13
* Two worker nodes
* * `kube02` - 172.29.14.14
* * `kube03` - 172.29.14.15

## All Nodes

### OS Config

```bash
sudo su -

setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Docker Install

```bash
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum remove -y podman-manpages
dnf install -y docker-ce
systemctl enable --now docker
```

### KubeAdm Install

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
dnf install -y kubeadm 
systemctl enable --now kubelet
```

## Master Node

### Kube Master Setup

```bash
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10251/tcp
firewall-cmd --permanent --add-port=10252/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --reload
```

```bash
kubeadm init
```

Make a note of the master join command:

```text
kubeadm join 172.29.14.13:6443 --token 0zuyh2.fhhp5osbpwjyfza5 --discovery-token-ca-cert-hash sha256:112eb1b2b5d28d56de76be9066e6f96acdff8103a889568a76c996508291e923
```

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify:

```bash
kubectl get nodes
```

### Pod Network

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

I had a problem with the Weave containers.

```bash
kubectl delete -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
curl -L git.io/weave -o /usr/local/bin/weave
chmod a+x /usr/local/bin/weave
weave reset
rm /opt/cni/bin/weave-*
weave setup
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

## Worker Nodes

Do all of the steps in the common section above.

```bash
firewall-cmd --permanent --add-port=6783/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --reload
```

```bash
kubeadm join 172.29.14.13:6443 --token 0zuyh2.fhhp5osbpwjyfza5 --discovery-token-ca-cert-hash sha256:112eb1b2b5d28d56de76be9066e6f96acdff8103a889568a76c996508291e923

curl -L git.io/weave -o /usr/local/bin/weave
chmod a+x /usr/local/bin/weave
rm /opt/cni/bin/weave-*
weave setup
```


Verify (on the master):

```bash
kubectl get nodes
```

## Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
kubectl proxy
```



## References

[TecMint - How to Install a Kubernetes Cluster on CentOS 8](https://www.tecmint.com/install-a-kubernetes-cluster-on-centos-8/)
[The Mori - Errors while deploying weave on kubernetes](https://crt.the-mori.com/2020-03-17-deploying-weave-kubernetes-cni-config)