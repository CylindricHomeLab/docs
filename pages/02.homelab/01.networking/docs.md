---
title: 'HomeLab Networking'
taxonomy:
    category:
        - homelab
    tag:
        - homelab
---

## Networks

### Subnets

* 172.16.0.0/12 = 172.16.0.0 - 172.31.255.255

| 172.16.0.0/24  | 
| ...            | 
| 172.29.12.0/24 | 
| 172.29.13.0/24 | hosts
| 172.29.14.0/24 | clients
| 172.29.15.0/24 | servers
| 172.29.16.0/24 | 
| ...            | 
| 172.29.31.0/24 | 




| 172.29.14.1   | router
| ...           | 
| 172.29.14.11  | dom01
| 172.29.14.12  | dom02
| 172.29.14.13  | jump01
| ...           | 
| 172.29.14.116 | hassio
| ...           | 
| 172.29.14.152 | octopi
| 172.29.14.153 | prox1
| 172.29.14.154 | 
| 172.29.14.155 | prox2

## Configure the MikoTik router

### DNS forwarding

This came thanks to an article by [Johann Fenech](https://blog.johannfenech.com/mikrotik-conditional-dns-forwarding/).
It sets up some firewall rules to forward DNS requests to homelab IPs to the AD-DNS server instead of having to duplicate them all manually on the router.

```text
/ip firewall layer7-protocol add name=home.cylindric.net regexp=home.cylindric.net
/ip firewall mangle add chain=prerouting dst-address=172.29.14.1 layer7-protocol=home.cylindric.net action=mark-connection new-connection-mark=home.cylindric.net-forward protocol=tcp dst-port=53
/ip firewall mangle add chain=prerouting dst-address=172.29.14.1 layer7-protocol=home.cylindric.net action=mark-connection new-connection-mark=home.cylindric.net-forward protocol=udp dst-port=53
/ip firewall nat add action=dst-nat chain=dstnat connection-mark=home.cylindric.net-forward to-addresses=172.29.14.11
/ip firewall nat add action=masquerade chain=srcnat connection-mark=home.cylindric.net-forward
```

