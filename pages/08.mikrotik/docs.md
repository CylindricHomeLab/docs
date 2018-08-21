---
title: 'Basic Configuration'
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
ip dns static add name=hassio.home.cylindric.net address=172.29.14.201
ip dns static add name=mqtt.home.cylindric.net address=172.29.14.201
ip dns static add name=influx.home.cylindric.net address=172.29.14.201
ip dns static add name=grafana.home.cylindric.net address=172.29.14.201
ip dns static add name=light.workshop.home.cylindric.net address=172.29.14.202
ip dns static add name=vac.workshop.home.cylindric.net address=172.29.14.203
ip dns static add name=power.workshop.home.cylindric.net address=172.29.14.204
ip dns static add name=heater.workshop.home.cylindric.net address=172.29.14.205
ip dns static add name=spare.workshop.home.cylindric.net address=172.29.14.206
```

# OpenVPN

Create an IP pool for the VPN to use:
```
/ip pool add name=ovpn-pool ranges=172.29.15.100-172.29.15.199
```

Create a new PPP profile:
```
/ppp profile 
add change-tcp-mss=default comment="" local-address=172.29.15.1 \
name="CylVpn" only-one=default remote-address=ovpn-pool \
use-compression=default use-encryption=required
```

Create a new user:
```
/ppp secret 
add caller-id="" comment="" disabled=no limit-bytes-in=0 \
limit-bytes-out=0 name="mark" password="userepassword" \
routes="" service=ovpn
```

Create a new OpenVPN server using a previously-created certificate:
```
/interface ovpn-server server 
set auth=sha1,md5 certificate=vpn.home.cylindric.net \
cipher=blowfish128,aes128,aes192,aes256 default-profile=CylVpn \
enabled=yes keepalive-timeout=disabled max-mtu=1500 mode=ip netmask=29 \
port=1194 require-client-certificate=no
```

Make sure inbound traffic to the port is enabled:
```
/ip firewall filter 
add action=accept chain=input comment="OpenVPN" disabled=no dst-port=1194 protocol=tcp
```