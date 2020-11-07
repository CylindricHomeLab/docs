---
title: 'VPN Setup'
taxonomy:
    category:
        - mikrotik
    tag:
        - mikrotik
        - vpn
---

Some notes on setting up VPN.

===

# Method from Medo64

https://www.medo64.com/2016/12/simple-openvpn-server-on-mikrotik/

```
/certificate
add name=ca-template common-name=home.cylindric.net days-valid=3650 key-size=2048 key-usage=crl-sign,key-cert-sign
add name=server-template common-name=*.home.cylindric.net days-valid=3650 key-size=2048 key-usage=digital-signature,key-encipherment,tls-server
add name=client-template common-name=client.home.cylindric.net days-valid=3650 key-size=2048 key-usage=tls-client
```

```
/certificate
sign ca-template name=ca-certificate
sign server-template name=server-certificate ca=ca-certificate
sign client-template name=client-certificate ca=ca-certificate
```

```
/certificate
export-certificate ca-certificate export-passphrase="" type=pem
export-certificate client-certificate export-passphrase=12345678 type=pkcs12
```

```
/ip
pool add name="vpn-pool" ranges=192.168.8.10-192.168.8.99
```

```
/ppp
profile add name="vpn-profile" use-encryption=yes local-address=192.168.8.250 dns-server=192.168.8.250 remote-address=vpn-pool
secret add name=user profile=vpn-profile password=password
```

```
/interface ovpn-server server
set default-profile=vpn-profile certificate=server-certificate require-client-certificate=yes auth=sha1 cipher=aes128,aes192,aes256 enabled=yes
```

```
/ip firewall filter
add chain=input protocol=tcp dst-port=1194 action=accept place-before=0 comment="Allow OpenVPN"
```

# Method from the Wiki

Some certificate requirements should be met to connect various devices to the server:

* Common name should contain IP or DNS name of the server;
* SAN (subject alternative name) should have IP or DNS of the server;
* EKU (extended key usage) tls-server and tls-client are required.

```
/certificate

add common-name=ca name=ca
sign ca ca-crl-host=home.cylindric.net

add common-name=vpn.home.cylindric.net subject-alt-name=DNS:vpn.home.cylindric.net key-usage=tls-server name=server1
sign server1 ca=ca
```

Since that the policy template must be adjusted to allow only specific network policies, it is advised to create a separate policy group and template. For compatibility, a new proposal is created with PFS group set to none.

```
/ip pool add name=rw-pool ranges=192.168.77.2-192.168.77.254

/ip ipsec proposal
add name=rw-proposal pfs-group=none

/ip ipsec mode-conf
add name=rw-conf \
    system-dns=no \
    static-dns=172.29.14.1 \
    address-pool=rw-pool \
    address-prefix=32

/ip ipsec policy
group add name=rw-policies
add template=yes \
    dst-address=192.168.77.0/24 \
    group=rw-policies \
    proposal=rw-proposal

/ip ipsec peer
add auth-method=rsa-signature \
    certificate=server1 \
    generate-policy=port-strict \
    mode-config=rw-conf \
    passive=yes \
    remote-certificate=none \
    exchange-mode=ike2 \
    policy-template-group=rw-policies
```

Generate a new certificate for the client and sign it with previously created CA. Export the client certificate in PKCS12 format.
```
/certificate
add common-name=RouterOS_client name=RouterOS_client key-usage=tls-client
sign RouterOS_client ca=ca 

export-certificate ca type=pem 
export-certificate RouterOS_client export-passphrase=1234567890 type=pkcs12 
```

Import those certs to the client.

Now we can create the peer configuration.

```
/ip ipsec peer
add address=vpn.home.cylindric.net auth-method=rsa-signature certificate=cert_export_RouterOS_client.p12_0 mode-config=request-only exchange-mode=ike2 generate-policy=port-strict
```

```
/ip firewall address-list
    add address=192.168.77.0/24 list=vpn

    /ip firewall filter
    add action=accept \
        chain=forward \
        log=yes \
        log-prefix=vpn-in \
        src-address-list=vpn \
        comment="IPSec Forward In" \
        place-before=0

    /ip firewall filter
    add action=accept \
        chain=input \
        log=yes \
        log-prefix=vpn-in \
        src-address-list=vpn \
        comment="IPSec Forward Input" \
        place-before=0
```


# Method from Wyzzicom

Create the server certificate, and sign it:
```
/certificate
add name=vpn \
    common-name=vpn.home.cylindric.net \
    subject-alt-name=DNS:vpn.home.cylindric.net \
    key-usage=digital-signature,tls-client,tls-server,ipsec-tunnel,ipsec-user,ipsec-end-system \
    days-valid=3650 \
    key-size=2048 \
    country=GB \
    state=Surrey \
    locality=Camberley \
    organization=CylCorp \
    unit="Services"

sign vpn ca=CylHomeCA name=vpn
set vpn trusted=yes


We need a certificate to authenticate clients:
```
/certificate
add name=client-template  \
    common-name=mark  \
    subject-alt-name=DNS:mark \
    key-usage=digital-signature,tls-client,tls-server,ipsec-tunnel,ipsec-user,ipsec-end-system \
    days-valid=3650  \
    key-size=2048  \
    country=GB \
    state=Surrey \
    locality=Camberley \
    organization=CylCorp \
    unit="Services" 

sign client-template ca=CylHomeCA name=mark
```

The client certificate needs to be distributed to the clients, so bundle it up as a PKCS12 certificate. First we export it:
```
/certificate export-certificate CylHomeCA
/certificate export-certificate client1 export-passphrase=XXXXXXXXXXXX
```

Transfer those files from the MikroTik to a computer that has OpenSSL tools installed on it, and convert them to a single PKCS12 file. There should be three files called cert_export_something. When prompted, enter the password used just above, and then a new password for the p12 file.

```
openssl pkcs12 -export -in cert_export_client1.crt -inkey cert_export_client1.key -certfile cert_export_CylHomeCA.crt -name client1 -out client1.p12
```

Upload the client1.p12 and the cert_export_CylHomeCA.crt to the client device, and install them. How you do that varies depending on the OS.

# Networking

Create a pool of IP addresses that we can use for the clients:

```
/ip pool add name=pool_vpn ranges=192.168.1.2-192.168.1.10  
```

# IPSec

Create a mode config that uses the pool we created earlier:
```
/ip ipsec mode-config
    add address-pool=pool_vpn \
    address-prefix-length=32 \
    name=vpn \
    split-include=172.16.0.0/12 \
    system-dns=no
```

Next we create a peer:
```
/ip ipsec peer
 add address=0.0.0.0/0 \
    auth-method=rsa-signature \
    certificate=router \
    dh-group=modp2048 \
    enc-algorithm=aes-256 \
    exchange-mode=ike2 \
    generate-policy=port-strict \
    hash-algorithm=sha256 \
    mode-config=vpn \
    passive=yes
```

And a proposal:
```
/ip ipsec proposal
    set [ find default=yes ] \
    auth-algorithms=sha256 \
    pfs-group=none \
    enc-algorithms=aes-256-cbc
```

And a policy:
```
/ip ipsec policy
set 0 \
    dst-address=192.168.1.0/24 \
    src-address=0.0.0.0/0 
```

# Firewall

The firewall needs to be told to accept incoming IPSec traffic:
```
/ip firewall filter

add action=accept \
    chain=input \
    dst-port=500,4500 \
    in-interface=ether1 \
    protocol=udp \
    comment="IPSec"

add action=accept \
    chain=input \
    in-interface=ether1 \
    protocol=ipsec-esp \
    comment="IPSec"
```


# Sources

https://www.wizzycom.net/mikrotik-pure-ipsec-vpn-and-android-device-as-client/, and just modified for my specific needs.

https://wiki.mikrotik.com/wiki/Manual:IP/IPsec#Road_Warrior_setup_using_IKEv2_with_RSA_authentication
