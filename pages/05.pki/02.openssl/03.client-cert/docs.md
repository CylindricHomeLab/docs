---
title: 'Client Cert'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

How to create client certificates with CFSSL.

===

# Client Certificates

## Create a Key

```sh
    cd /root/ca
    openssl genrsa -out intermediate/private/www.cylindric.net.key.pem 2048
    chmod 400 intermediate/private/www.cylindric.net.key.pem
```

## Create a Certificate

```sh
    openssl req -config intermediate/openssl.cnf \
        -key intermediate/private/www.cylindric.net.key.pem \
        -new -sha256 -out intermediate/csr/www.cylindric.net.csr.pem

    openssl ca -config intermediate/openssl.cnf \
          -extensions server_cert -days 375 -notext -md sha256 \
          -in intermediate/csr/www.cylindric.net.csr.pem \
          -out intermediate/certs/www.cylindric.net.cert.pem

    chmod 444 intermediate/certs/www.cylindric.net.cert.pem
```
