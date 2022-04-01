---
title: 'My Setup'
taxonomy:
    category:
        - mikrotik
    tag:
        - mikrotik
        - dns
---

Some notes on setting up DNS.

===

# Static DNS Records

```
# Core Networking
ip dns static add name=vpn.home.cylindric.net address=172.29.14.1

# Home Automation
ip dns static add name=hassio.home.cylindric.net address=172.29.14.201
ip dns static add name=mqtt.home.cylindric.net address=172.29.14.201
ip dns static add name=influx.home.cylindric.net address=172.29.14.201
ip dns static add name=grafana.home.cylindric.net address=172.29.14.201

# Workshop Home Automation
ip dns static add name=light.workshop.home.cylindric.net address=172.29.14.202
ip dns static add name=vac.workshop.home.cylindric.net address=172.29.14.203
ip dns static add name=power.workshop.home.cylindric.net address=172.29.14.204
ip dns static add name=heater.workshop.home.cylindric.net address=172.29.14.205
ip dns static add name=spare.workshop.home.cylindric.net address=172.29.14.206
```

# Backup WAN

I have added a a [ZTE MF833U1 USB LTE adapter](https://www.amazon.co.uk/gp/product/B08R69YGLV) directly to the Mikrotik's USB port.

* The device shows up under `System > Resources > USB`  as a rather shoddy device with a Vendor ID of `DEMO,Incorporated`, a Name of `DEMO Mobile Broadband` and a Serial Number `1234567890ABCDEF`.

* It automatically adds an interface called `lte1`.

* It automatically shows in `IP > Adresses` with a local address `192.168.0.178`.

## Setup

1. Set more useful name for interfaces
   ```mikrotik
   /interface ethernet
   set [ find default-name=ether1 ] name=ether1-wan
   set [ find default-name=ether2 ] name=ether1-srv
   set [ find default-name=ether3 ] name=ether1-lan
   ```
2. Enable internet detection
   ```mikrotik
   /interface detect-internet
   set detect-interface-list=all
   set internet-interface-list=all
   set lan-interface-list=all
   set wan-interface-list=all
   ```
3. Set masquerading on the WAN interfaces
   ```mikrotik
   /ip firewall nat
   add chain=srcnat action=masquerade out-interface=ether1-wan comment="WAN failover"
   add chain=srcnat action=masquerade out-interface=lte1 comment="WAN failover"
   ```