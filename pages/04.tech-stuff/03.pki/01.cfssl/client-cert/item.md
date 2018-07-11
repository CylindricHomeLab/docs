---
title: Client Cert
taxonomy:
    category:
        - Tech
    tag:
        - pki
        - cfssl
theme: cylindric-kb
blog_url: /blog
show_sidebar: true
show_breadcrumbs: true
show_pagination: true
---

How to create client certificates with CFSSL.

===

# Client Certificates
## Create a Key
```
cd /root/ca

openssl genrsa -out intermediate/private/www.caas.local.key.pem 2048

chmod 400 intermediate/private/www.caas.local.key.pem
```

## Create a Certificate
```
openssl req -config intermediate/openssl.cnf \
    -key intermediate/private/www.caas.local.key.pem \
		-new -sha256 -out intermediate/csr/www.caas.local.csr.pem

openssl ca -config intermediate/openssl.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in intermediate/csr/www.caas.local.csr.pem \
      -out intermediate/certs/www.caas.local.cert.pem

chmod 444 intermediate/certs/www.caas.local.cert.pem
```