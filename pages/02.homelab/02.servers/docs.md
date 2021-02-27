---
title: 'HomeLab Servers'
taxonomy:
    category:
        - homelab
    tag:
        - homelab
---

# Domain Controllers

## dom01

Use `sconfig` to set basic parameters such as hostname, IP address and default gateway.

```powershell
Add-WindowsFeature AD-Domain-Services

Import-Module ADDSDeployment

Install-ADDSForest `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -DomainMode "WinThreshold" `
    -DomainName "home.cylindric.net" `
    -DomainNetbiosName "home" `
    -ForestMode "WinThreshold" `
    -InstallDns:$true `
    -LogPath "C:\Windows\NTDS" `
    -NoRebootOnCompletion:$false `
    -SysvolPath "C:\Windows\SYSVOL" `
    -Force:$true
```

## Dom02

Use `sconfig` to set basic parameters such as hostname, IP address and default gateway. Set DNS server to be the first DC's IP.

```powershell
Add-WindowsFeature AD-Domain-Services

Import-Module ADDSDeployment

Install-ADDSDomainController -InstallDns -Credential (Get-Credential "HOME\Administrator") -DomainName "home.cylindric.net"
```

## Domain Prep

### DC Auto-start

```bash
export APINODE=prox1.home.cylindric.net
export TARGETNODE=prox1
export PROXUSER=your_username
export PROXPASS=your_password

# get the CA root certificate
curl --insecure --silent --cookie "$(<cookie)" https://$APINODE:8006/api2/json/nodes/prox1/certificates/info \
    | jq --raw-output '.data[] | select(.filename == "pve-root-ca.pem").pem' \
    > $APINODE.crt

sudo cp $APINODE.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# future requests shouldn't need "insecure" any more.

curl --silent --data "username=$PROXUSER@pam&password=$PROXPASS" \
 https://$APINODE:8006/api2/json/access/ticket\
| jq --raw-output '.data.ticket' | sed 's/^/PVEAuthCookie=/' > cookie

curl --silent --data "username=$PROXUSER@pam&password=$PROXPASS" \
 https://$APINODE:8006/api2/json/access/ticket \
| jq --raw-output '.data.CSRFPreventionToken' | sed 's/^/CSRFPreventionToken:/' > csrftoken

curl -s --cookie "$(<cookie)" https://$APINODE:8006/api2/json/nodes/$TARGETNODE/status | jq '.'

# Set autoboot for DOM01 and DOM02
VMID=$(curl --silent --cookie "$(<cookie)" https://$APINODE:8006/api2/json/cluster/resources | jq --raw-output -c '.data[] | select(.name == "dom01").vmid')
VMNODE=$(curl --silent --cookie "$(<cookie)" https://$APINODE:8006/api2/json/cluster/resources | jq --raw-output -c '.data[] | select(.name == "dom01").node')
curl -s -H "$(<csrftoken)" --cookie "$(<cookie)" -X POST -d onboot=1 "https://$APINODE:8006/api2/json/nodes/$VMNODE/qemu/$VMID/config"

VMID=$(curl --silent --cookie "$(<cookie)" https://$APINODE:8006/api2/json/cluster/resources | jq --raw-output -c '.data[] | select(.name == "dom02").vmid')
VMNODE=$(curl --silent --cookie "$(<cookie)" https://$APINODE:8006/api2/json/cluster/resources | jq --raw-output -c '.data[] | select(.name == "dom02").node')
curl -s -H "$(<csrftoken)" --cookie "$(<cookie)" -X POST -d onboot=1 "https://$APINODE:8006/api2/json/nodes/$VMNODE/qemu/$VMID/config"
```

### Core AD

```powershell
New-ADOrganizationalUnit -Path "DC=home,DC=cylindric,DC=net" -Name "Cylindric" 
New-ADOrganizationalUnit -Path "OU=Cylindric,DC=home,DC=cylindric,DC=net" -Name "Computers"
New-ADOrganizationalUnit -Path "OU=Cylindric,DC=home,DC=cylindric,DC=net" -Name "Users"
New-ADOrganizationalUnit -Path "OU=Cylindric,DC=home,DC=cylindric,DC=net" -Name "Groups"
```

### Core DNS

```powershell
$Dom = "dom01"
$Zone = "home.cylindric.net"
Add-DnsServerPrimaryZone -ComputerName $Dom -NetworkID "172.29.14.0/24" -ReplicationScope "Forest"
Add-DnsServerResourceRecordA -ComputerName $Dom -Name "prox1" -ZoneName $Zone -AllowUpdateAny -CreatePtr -IPv4Address "172.29.14.153"
Add-DnsServerResourceRecordA -ComputerName $Dom -Name "prox2" -ZoneName $Zone -AllowUpdateAny -CreatePtr -IPv4Address "172.29.14.155"
Add-DnsServerResourceRecordA -ComputerName $Dom -Name "router" -ZoneName $Zone -AllowUpdateAny -CreatePtr -IPv4Address "172.29.14.1"
Add-DnsServerResourceRecordA -ComputerName $Dom -Name "hassio" -ZoneName $Zone -AllowUpdateAny -CreatePtr -IPv4Address "172.29.14.116"
Add-DnsServerResourceRecordA -ComputerName $Dom -Name "octopi" -ZoneName $Zone -AllowUpdateAny -CreatePtr -IPv4Address "172.29.14.152"
Add-DnsServerResourceRecordCName -ComputerName $Dom -Name "prox" -HostNameAlias "prox1.home.cylindric.net" -ZoneName $Zone
Add-DnsServerResourceRecordCName -ComputerName $Dom -Name "mqtt" -HostNameAlias "hassio.home.cylindric.net" -ZoneName $Zone
Add-DnsServerResourceRecordCName -ComputerName $Dom -Name "sabnzbd" -HostNameAlias "megavecii.home.cylindric.net" -ZoneName $Zone
Add-DnsServerResourceRecordCName -ComputerName $Dom -Name "sonarr" -HostNameAlias "megavecii.home.cylindric.net" -ZoneName $Zone
Add-DnsServerResourceRecordCName -ComputerName $Dom -Name "radarr" -HostNameAlias "megavecii.home.cylindric.net" -ZoneName $Zone
```

## Windows Servers

For manual builds, simply use `sconfig` to setup the basic networking and hostnames etc.

```powershell
Add-Computer -DomainName home.cylindric.net -Restart -NewName dom03
```

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