---
title: 'Basic Client Cert'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

How to create client certificates with CFSSL.

===

This article carries on from the CA setup article, so the setup from that is assumed to be in place.

We'll refer to the base directory as `$BASE` everywhere, just to make it clear where each step is looking.


# Client Certificates

## Create a Config
It is possible to just let the `openssl req` prompt for these details, but I like to pre-configure them for easy re-use. It also allows you to define the Subject Alternative Names, which is now pretty much mandatory.

Create `$BASE/certificate-configs/www.cylindric.net.cnf`:

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
DNS.1 = www.cylindric.net
IP.1  = 104.28.15.24
```

## Create a Key

```sh
cd $BASE

openssl genrsa \
        -out certs/www.cylindric.net.key.pem 2048

chmod 400 certs/www.cylindric.net.key.pem
```

## Create a signing-request
```sh
cd $BASE

openssl req \
        -new \
        -sha256 \
        -config certificate-configs/www.cylindric.net.cnf \
        -key certs/www.cylindric.net.key.pem \
        -out intermediate-ca/csr/www.cylindric.net.csr.pem
```

## Sign the certificate
```sh
cd $BASE/intermediate-ca

openssl ca \
        -batch \
        -config openssl.cnf \
        -notext \
        -in csr/www.cylindric.net.csr.pem \
        -passin pass:intermediatepass \
        -out ../certs/www.cylindric.net.crt.pem

rm -f csr/www.cylindric.net.csr.pem
```

## Update the CRL
```sh
cd $BASE/intermediate-ca
openssl ca \
        -config openssl.cnf \
        -gencrl \
        -keyfile private/intermediate.key.pem \
        -passin pass:intermediatepass \
        -cert certs/intermediate.cert.pem \
        -out crl/intermediate.crl.pem

openssl crl \
        -inform PEM \
        -in crl/intermediate.crl.pem \
        -outform DER \
        -out crl/intermediate.crl
```

## Check
Check the certificate was created correctly:
```sh
openssl verify -CAfile certs/ca-chain.cert.pem certs/www.cylindric.net.crt.pem
```

Verify with the CRL too:
```sh
openssl verify \
    -crl_check \
    -CAfile intermediate-ca/crl/intermediate.crl.pem \
    certs/www.cylindric.net.crt.pem
```


## Certificate including CA chain

Certificates should be combined in this order:

1. Primary certificate
2. Intermediate certificate
3. Root certificate