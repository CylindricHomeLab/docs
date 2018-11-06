---
title: 'PKI Setup'
taxonomy:
    category:
        - mikrotik
    tag:
        - mikrotik
        - pki
---

Some notes on setting up DNS.

===

Don't forget, [Ctrl]+[V] toggles auto-complete, so if pasting causes duplicate characters, try pressing that and then pasting again with a right-click.

First we create a root CA certificate template. We'll use this to create the CA certificate.

```
/certificate
add name=ca-template \
    common-name=CylHomeCA \
    key-usage=key-cert-sign,crl-sign \
    days-valid=3650 \
    key-size=2048 \
    country=GB \
    state=Surrey \
    locality=Camberley \
    organization=CylCorp \
    unit="Certificate Authority"

sign ca-template name=CylHomeCA

set CylHomeCA trusted=yes
```

Create the root certificate, and sign it:
```
/certificate
add name=server-template \
    common-name=router \
    days-valid=3650 \
    key-size=2048 \
    country=GB \
    state=Surrey \
    locality=Camberley \
    organization=CylCorp \
    unit="Services"

sign server-template ca=CylHomeCA name=router
set router trusted=yes
```
