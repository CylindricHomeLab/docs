---
title: 'OpenSSL CA Setup'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

General setup information for OpenSSL.

===

This example creates a Root Certificate Authority, an intermediate Signing Authority and then a sample certificate. In practice, the Root and Intermediate would be on different servers, with the Root possibly even locked away somewhere nice and secure.

Everything is created here in a single base directory, with some sub-directories for each CA and then certificates and configs etc.

The private keys should be password protected, and in these examples I use "rootpass" and "intermediatepass".

```text
.
├── certificate-configs
├── certs
├── intermediate-ca
└── root-ca
```

We'll refer to the base directory as `$BASE` everywhere, just to make it clear where each step is looking.

# SSL CA
## Pre-Requisites

```sh
mkdir -p $BASE/certs
mkdir -p $BASE/root-ca
cd $BASE/root-ca
mkdir certs crl newcerts private
chmod 700 private
touch index.txt
echo unique_subject = yes > index.txt.attr
echo 1000 > serial
echo 1000 > crlnumber
``` 


## Set the Main Configuration
Create a file called `$BASE/root-ca/openssl.cnf` with the following:
```text
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = .
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand
private_key       = $dir/private/root.key.pem
certificate       = $dir/certs/root.cert.pem
crlnumber         = $dir/crlnumber
crl               = $dir/crl/root.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 730

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
0.organizationName     = CylCorp
organizationalUnitName = Security
commonName             = CylCorp Root CA

[ v3_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:true
keyUsage               = critical, digitalSignature, cRLSign, keyCertSign
extendedKeyUsage       = serverAuth
crlDistributionPoints  = @crl_section
authorityInfoAccess    = @ocsp_section

[ v3_intermediate_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:true, pathlen:0
keyUsage               = critical, digitalSignature, cRLSign, keyCertSign
authorityInfoAccess    = @ocsp_section
crlDistributionPoints  = @crl_section

[ crl_ext ]
authorityKeyIdentifier = keyid:always
issuerAltName          = issuer:copy 

[ ocsp ]
basicConstraints       = CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid, issuer
keyUsage               = critical, digitalSignature
extendedKeyUsage       = critical, OCSPSigning

[alt_names]
DNS.0 = Sandbox Intermidiate CA 1
DNS.1 = Sandbox CA Intermidiate 1

[crl_section]
URI.0 = http://pki.cylindric.net/CylCorpRoot.crl
URI.1 = http://pki.cylindric.net/SandboxRoot.crl

[ocsp_section]
caIssuers;URI.0 = http://pki.cylindric.net/CylCorpRoot.crt
caIssuers;URI.1 = http://pki.backup.cylindric.net/CylCorpRoot.crt
OCSP;URI.0 = http://pki.cylindric.net/ocsp/
OCSP;URI.1 = http://pki.backup.cylindric.net/ocsp/
```


## Create the Root Key
```sh
cd $BASE/root-ca

openssl genrsa \
        -aes256 \
        -out private/root.key.pem \
        -passout pass:rootpass 4096

chmod 400 private/root.key.pem
```

## Create the CA Certificate
```sh
cd $BASE/root-ca

openssl req \
        -config openssl.cnf \
        -key private/root.key.pem \
        -passin pass:rootpass \
        -new \
        -x509 \
        -days 7300 \
        -extensions v3_ca \
        -out certs/root.cert.pem

chmod 444 certs/root.cert.pem
```

# SSL Intermediate CA
## Configure the Intermediate CA
```sh
cd $BASE
mkdir -p $BASE/intermediate-ca
cd $BASE/intermediate-ca
mkdir certs crl csr newcerts private
chmod 700 private
echo 1000 > serial
echo 1000 > crlnumber
touch index.txt
echo unique_subject = yes > index.txt.attr
```

Create a file called `$BASE/intermediate-ca/openssl.cnf` with the following:
```text
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = .
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 365
default_md        = sha256
name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

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
0.organizationName     = CylCorp
organizationalUnitName = Security
commonName             = CylCorp Intermediate CA

[ v3_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:true, pathlen:0
keyUsage               = critical, digitalSignature, cRLSign, keyCertSign

[ crl_ext ]
authorityKeyIdentifier = keyid:always

[crl_section]
URI.0 = http://pki.cylindric.net/CylCorpIntermediate.crl
URI.1 = http://pki.cylindric.net/CylCorpIntermediate.crl

[ocsp_section]
caIssuers;URI.0 = http://pki.cylindric.net/CylCorpIntermediate.crt
caIssuers;URI.1 = http://pki.backup.cylindric.net/CylCorpIntermediate.crt
OCSP;URI.0 = http://pki.cylindric.net/ocsp/
OCSP;URI.1 = http://pki.backup.cylindric.net/ocsp/
```

## Create the Intermediate Certificate
```sh
cd $BASE/intermediate-ca

openssl genrsa \
        -aes256 \
        -out private/intermediate.key.pem \
        -passout pass:intermediatepass 4096

chmod 400 private/intermediate.key.pem
```

## Create the Intermediate CA Request
```sh
cd $BASE/intermediate-ca

openssl req \
        -config openssl.cnf \
        -new -sha256 \
        -key private/intermediate.key.pem \
        -passin pass:intermediatepass \
        -out csr/intermediate.csr.pem
```

## Create the Intermediate CA certificate
This certificate is signed by the Root CA, so note that we're in root-ca now...

```sh
cd $BASE/root-ca

openssl ca \
        -batch \
        -config openssl.cnf \
        -extensions v3_intermediate_ca \
        -days 3650 \
        -notext \
        -in ../intermediate-ca/csr/intermediate.csr.pem \
        -passin pass:rootpass \
        -out ../intermediate-ca/certs/intermediate.cert.pem

rm -f ../intermediate-ca/csr/intermediate.csr.pem

chmod 444 ../intermediate-ca/certs/intermediate.cert.pem
```

## Update the CRL
It's important to make sure the CRL is up to date after every addition or revokation of a certificate.

```sh
cd $BASE/root-ca

openssl ca \
        -config openssl.cnf \
        -gencrl \
        -passin pass:rootpass \
        -cert certs/root.cert.pem \
        -out crl/root.crl.pem

openssl crl \
        -inform PEM \
        -in crl/root.crl.pem \
        -outform DER \
        -out crl/root.crl

cd $BASE/intermediate-ca

openssl ca \
        -config openssl.cnf \
        -gencrl \
        -passin pass:intermediatepass \
        -cert certs/intermediate.cert.pem \
        -out crl/intermediate.crl.pem

openssl crl \
        -inform PEM \
        -in crl/intermediate.crl.pem \
        -outform DER \
        -out crl/intermediate.crl

```

## Create a Certificate Chain
```sh
cd $BASE

cat intermediate-ca/certs/intermediate.cert.pem root-ca/certs/root.cert.pem > certs/ca-chain.cert.pem
chmod 444 certs/ca-chain.cert.pem
```

https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html
~~https://www.semurity.com/blog-post/setup-certificate-authority-ca-using-openssl/~~
