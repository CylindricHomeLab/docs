---
title: 'Certificate With San'
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

How to create a certificate with SAN fields.

===

# Create a Request Config
Create a cert.cnf request template

```text
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
prompt             = no

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
stateOrProvinceName         = State or Province Name (full name)
localityName               = Locality Name (eg, city)
organizationName           = Organization Name (eg, company)
commonName                 = Common Name (e.g. server FQDN or YOUR name)
[ req_ext ]
subjectAltName = @alt_names
[alt_names]
DNS.1   = bestflare.com
DNS.2   = usefulread.com
DNS.3   = chandank.com
```

# Create a CSR
Create a CSR and key from it


```text
openssl req -out sslcert.csr -newkey rsa:2048 -nodes -keyout private.key -config cert.cnf
```

