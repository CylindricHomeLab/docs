---
title: 'Converting to and from PKCS12 Certificates'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

How to convert to and from a single PKCS12 certificate containing a cert, the key and the CA certs...

===

Converting a private key, a certificate and a CA chain to a single PKCS12 certificate bundle:

```sh
openssl pkcs12 -export -inkey cert-key.pem -in cert.crt -certfile ca_chain.pem -out cert.p12
```

Converting a PKCS12 certificate into separate files:

```sh
openssl pkcs12 -in cert.p12 -out cert-key.pem -nocerts -nodes
openssl pkcs12 -in cert.p12 -out cert.pem -clcerts -nokeys
```