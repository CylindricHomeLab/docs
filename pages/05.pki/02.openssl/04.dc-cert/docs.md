---
title: 'Domain Controller Certificate'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

How to create a certificate for Windows Domain Controllers.

===

AD servers needing a certificate for LDAPS need some extra permitted uses. Also,
they must have SAN entries to be accepted by various third-party products, such
as anything Java based. Therefore the key requirements are:

-   CN to match the server FQDN
-   SAN entries for
    -   FQDN - for direct connections
    -   Domain name - for DC discovery using the DNS domain
    -   Generic FQDN - optional, but common where using something not tied to the host is required; load balancing, etc.
    -   IP address - optional, but useful for connecting via LDAP when you don't have DNS resolution to the DC
-   Usages:
    -   client authentication
    -   server authentication
    -   key encipherment
    -   digital signature
    -   non-repudiation
    -   email protection
    -   Smart Card login

# Create a Request Config

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
commonName          = dc1.cylindric.net

[ req_ext ]
subjectAltName         = @alt_names
keyUsage               = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage       = critical, clientAuth, emailProtection, msSmartcardLogin
basicConstraints       = critical, CA:FALSE

[ alt_names ]
DNS.1 = dc1.cylindric.net
DNS.2 = cylindric.net
DNS.3 = ldap.cylindric.net
IP.1  = 192.168.0.10
```

# Create a CSR

Create a CSR and key from it

```sh
openssl req -out sslcert.csr -newkey rsa:2048 -nodes -keyout private.key -config cert.cnf
```
