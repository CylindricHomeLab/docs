---
title: 'AWX'
taxonomy:
    category:
        - homelab
    tag:
        - homelab
        - awx
---

```bash
sudo dnf install -y epel-release
sudo dnf makecache

# Ansible
sudo dnf install -y ansible

# Docker
sudo su -
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum remove -y podman-manpages
dnf install -y docker-ce
systemctl enable --now docker
usermod -a -G docker mark
exit

# Docker-Compose
sudo pip3 install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# AWX
mkdir awx-test && cd awx-test
curl -O https://raw.githubusercontent.com/geerlingguy/awx-container/master/docker-compose.yml
docker-compose up -d
```
