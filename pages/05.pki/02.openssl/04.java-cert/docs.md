---
title: 'Java Keystore from certificate'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
        - java
---

How to create a Java KeyStore from a certificate

===

# Create a Request Config

First we need to create a config file we'll use to request the certificate. This
can have anything in it that the certificate needs to have, such as SAN entries.

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
commonName          = java-application.cylindric.net

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = java-application.cylindric.net
DNS.2 = servername.cylindric.net
IP.1  = 192.168.0.10
```

# Create a CSR

Create a CSR and key from it

```sh
openssl req -out request.csr -newkey rsa:2048 -nodes -keyout private.key -config cert.cnf
```

# Create a certificate

Use the above CSR to generate a certificate, using whatever PKI infrastructure
there is. I'll assume this is called "certificate.crt". You'll also need the CA
certificate chain, which we'll call "ca.crt".

# Convert the private key

OpenSSL seems to create the private key as a PKCS8 key, and not a PKCS1 key as
expected by keytool, so convert it

```sh
openssl rsa -in private.key -out private.key
```

# Create a PKCS12 certificate

Bundle the certificate, the key and the CA certificate chain into a single
PKCS12 container. Make sure to give it a password, traditionally this can just
be "password", but this obviously depends on your requirements.

```sh
openssl pkcs12 -export -inkey private.key -in certificate.crt -out certificate.p12 -CAfile ca.crt
```

# Create the KeyStore

Now create a new Java Keystore, inputting the PKCS12 password at the prompt, and
setting the KeyStore password to be "password" too. In the second command we
also perform the action recommended at the end of the first, which is to convert
it from the proprietary format to PKCS12. I'm not sure if this means all this
process is irrelevant, and we could have just used the PKCS12 cert as it was...

```sh
keytool -importkeystore -destkeystore keystore.jks -deststoretype JKS -srckeystore certificate.p12 -srcstoretype pkcs12 -storepass password
keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.jks -deststoretype pkcs12
```

Now you have a keystore.jks that can be used by Java stuff.
