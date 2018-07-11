---
title: OpenSSL Setup
taxonomy:
    category:
        - Tech
    tag:
        - pki
        - openssl
theme: knowledge-base-cyl
blog_url: /blog
show_sidebar: true
show_breadcrumbs: true
show_pagination: true
---

General setup information for OpenSSL.

===

# SSL CA
## Pre-Requisites

```text
mkdir ./ca
cd ./ca
mkdir certs crl newcerts private intermediate
chmod 700 private
touch index.txt
echo 1000 > serial
```

## Set the Main Configuration
Create a file called `openssl.cnf` with the following:
```
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = /root/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem

crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict


[ policy_strict ]
countryName            = match
stateOrProvinceName    = match
localityName           = optional
organizationName       = match
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ policy_loose ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ req ]
prompt                 = no
string_mask            = utf8only
default_bits           = 2048
default_md             = sha256
distinguished_name     = req_distinguished_name
x509_extensions        = v3_ca

[ req_distinguished_name ]
countryName            = GB
stateOrProvinceName    = Surrey
localityName           = Camberley
0.organizationName     = CAAS
organizationalUnitName = Security
commonName             = CAAS Root CA

[ v3_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:true
keyUsage               = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:true, pathlen:0
keyUsage               = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
basicConstraints       = CA:FALSE
nsCertType             = client, email
nsComment              = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid, issuer
keyUsage               = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage       = clientAuth, emailProtection

[ server_cert ]
basicConstraints       = CA:FALSE
nsCertType             = server
nsComment              = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid, issuer:always
keyUsage               = critical, digitalSignature, keyEncipherment
extendedKeyUsage       = serverAuth

[ crl_ext ]
authorityKeyIdentifier = keyid:always

[ ocsp ]
basicConstraints       = CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid, issuer
keyUsage               = critical, digitalSignature
extendedKeyUsage       = critical, OCSPSigning
```
## Create the Root Key
```text
openssl genrsa -aes256 -out private/ca.key.pem 4096
chmod 400 private/ca.key.pem
```

## Create the CA Certificate
```text
openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -extensions v3_ca -out certs/ca.cert.pem
chmod 444 certs/ca.cert.pem
```

# SSL Intermediate CA
## Configure the Intermediate CA
```
cd intermediate
mkdir certs crl csr newcerts private
chmod 700 private
echo 1000 > serial
echo 1000 > crlnumber
touch index.txt
cp ../openssl.cnf .
```

Edit the Intermediate's `openssl.cnf` and change the path information:
```
dir               = /root/ca/intermediate
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem
crl               = $dir/crl/intermediate.crl.pem
policy            = policy_loose
```

## Create the Intermediate Certificate
```
cd /root/ca

openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
chmod 400 intermediate/private/intermediate.key.pem

openssl req -config intermediate/openssl.cnf -new -sha256 \
      -key intermediate/private/intermediate.key.pem \
      -out intermediate/csr/intermediate.csr.pem

openssl ca -config openssl.cnf -extensions v3_intermediate_ca \
      -days 3650 -notext -md sha256 \
      -in intermediate/csr/intermediate.csr.pem \
      -out intermediate/certs/intermediate.cert.pem

chmod 444 intermediate/certs/intermediate.cert.pem
```

## Create a Certificate Chain
```
cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
chmod 444 intermediate/certs/ca-chain.cert.pem
```

https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html
~~https://www.semurity.com/blog-post/setup-certificate-authority-ca-using-openssl/~~