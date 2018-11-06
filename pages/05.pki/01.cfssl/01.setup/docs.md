---
title: 'CFSSL Setup'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - cfssl
---

General setup information for CFSSL.

===

# Pre-reqs

```sh
apt install -y golang
go get -u github.com/cloudflare/cfssl/cmd/...
```

# Root CA

Create a main config ./ca/ca-config.json:

```json
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "server": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "8760h"
      },
      "client": {
        "usages": [
          "digital signature",
          "email protection",
          "client auth"
        ],
        "expiry": "8760h"
      }
    }
  }
}
```

Create a CSR config ./ca/ca-csr.json:

```json
{
  "CN": "CylCerts Root CA",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "GB",
      "ST": "Surrey",
      "L": "Farnborough",
      "O": "CylCerts",
      "OU": "Security"
    }
  ]
}
```

Generate a certificate:

```sh
cfssl gencert -initca ./ca/ca-csr.json | cfssljson -bare ./ca/ca
```

# Intermediate CA

Create a CSR config ./intermediate/intermediate-csr.json:

```json
{
  "CN": "CylCerts Intermediate CA",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C":  "GB",
      "ST": "Surrey",
      "L":  "Farnborough",
      "O":  "CylCerts",
      "OU": "Security"
    }
  ],
  "ca": {
    "expiry": "42720h"
  }
}
```

Create the signing config intermediate/root_to_intermediate_ca.json:

```json
{
  "signing": {
    "default": {
      "usages": [
        "digital signature",
        "cert sign",
        "crl sign",
        "signing"
      ],
      "expiry": "87600h",
      "ca_constraint": {"is_ca": true, "max_path_len":0, "max_path_len_zero": true}
    }
  }
}
```

Create the intermediate certificate:

```sh
cfssl gencert -initca intermediate/intermediate-csr.json | cfssljson -bare intermediate/intermediate_ca
```

Sign the intermediate certificate:

```sh
cfssl sign -ca ca/ca.pem -ca-key ca/ca-key.pem -config intermediate/root_to_intermediate_ca.json intermediate/intermediate_ca.csr | cfssljson -bare intermediate/intermediate_ca
```

# Server Certificate

Create the CSR config newserver.csr.json:

```json
{
  "CN": "newserver.cylindric.net",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [
  {
    "C":  "GB",
    "ST": "Surrey",
    "L":  "Farnborough",
    "O":  "CylCerts",
    "OU": "Security"
  }
  ],
  "Hosts": ["newserver.cylindric.net"]
}
```

Create the signing config ./intermediate/intermediate_to_client_cert.json:

```json
{
  "signing": {
    "profiles": {
      "CA": {
        "usages": ["cert sign"],
        "expiry": "8760h"
      }
    },
    "default": {
      "usages": ["digital signature"],
      "expiry": "8760h"
    }
  }
}
```

Generate the certificate:

```sh
cfssl gencert -ca intermediate/intermediate_ca.pem -ca-key intermediate/intermediate_ca-key.pem -config intermediate/intermediate_to_client_cert.json newserver.cylindric.net.json | cfssljson -bare certs/newserver.cylindric.net
```

Clean up the old files:

```sh
rm *.csr *.json
```
