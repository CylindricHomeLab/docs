---
title: 'HomeLab Servers'
taxonomy:
    category:
        - homelab
    tag:
        - homelab
---

## General Prep

* Create appropriate admin groups in AD

## Windows Servers

For manual builds, simply use `sconfig` to setup the basic networking and hostnames etc.

## CentOS Servers

Elevate to root, this is all privileged stuff.

```bash
sudo su -
yum update -y
dnf makecache
```

Enable the Cockpit interface:

```bash
systemctl enable --now cockpit.socket
```

Hyper-V config

```bash
cat << EOF > /etc/udev/rules.d/100-balloon.rules
SUBSYSTEM=="memory", ACTION=="add", ATTR{state}="online"
EOF

yum install -y hyperv-daemons

echo none > /sys/block/sda/queue/scheduler
grub2-editenv – set "$(grub2-editenv – list | grep kernelopts) elevator=none"
grub2-editenv – list | grep kernelopts
```

### Domain Join

Join the server to the domain (don't copy'n'paste the whole lot, `join` prompts for a password):

```bash
dnf install -y realmd sssd oddjob oddjob-mkhomedir adcli samba-common samba-common-tools krb5-workstation authselect-compat
realm join home.cylindric.net -U Administrator
authselect select sssd
authselect select sssd with-mkhomedir
sed -i '/^services =.*/a default_domain_suffix = home.cylindric.net' /etc/sssd/sssd.conf
systemctl restart sssd
```

Now allow access to the appropriate user groups:

```bash
realm permit -g 'Domain Admins'
realm permit -g 'ExampleGroup'
```

Now allow appropriate groups to use `sudo`:

```bash
cat << EOF > /etc/sudoers.d/domain_admins
%Domain\ Admins@home.cylindric.net  ALL=(ALL) ALL
%ExampleGroup@home.cylindric.net  ALL=(ALL) ALL
EOF
```

## References

* [ComputingForGeeks](https://computingforgeeks.com/join-centos-rhel-system-to-active-directory-domain/)
* [Altaro](https://www.altaro.com/hyper-v/centos-linux-hyper-v/)