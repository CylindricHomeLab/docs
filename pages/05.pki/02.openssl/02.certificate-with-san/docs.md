---
title: 'Certificate With San'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

How to create a certificate with SAN fields.

===

# Create a Request Config

Create a cert.cnf request template. Note that because this has `prompt=no` in
it, the values will come from the config and not interactively.

```text
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
prompt             = no

[ req_distinguished_name ]
countryName         = GB
stateOrProvinceName = Surrey
localityName        = Camberley
organizationName    = CylCorp
commonName          = www.cylindric.net

[ req_ext ]
subjectAltName = @alt_names

[alt_names]
DNS.1   = www.cylindric.net
DNS.2   = host1.uk.lon.cylindric.net
DNS.3   = ftp.cylindric.net
```

# Create a CSR

Create a CSR and key from it

```sh
openssl req -out sslcert.csr -newkey rsa:2048 -nodes -keyout private.key -config cert.cnf
```
