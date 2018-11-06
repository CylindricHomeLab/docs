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
