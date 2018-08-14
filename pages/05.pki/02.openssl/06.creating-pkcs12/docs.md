---
title: 'Creating a PKCS12 Certificate'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

How to create a single PKCS12 certificate containing a cert, the key and the CA certs...

===

```sh
openssl pkcs12 -export  -inkey newcert-key.key -in newcert.crt -certfile ca_chain.pem -out newcert.pfx
```
