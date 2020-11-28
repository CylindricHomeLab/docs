---
title: 'HomeLab Networking'
taxonomy:
    category:
        - homelab
    tag:
        - homelab
---

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

