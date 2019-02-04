---
title: 'EJBCA Notes'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - ejbca
---

Various stuff about EJBCA.

===


# Create a new Root CA

```sh
bin/ejbca.sh ca init \
    --caname SandboxRootCA \
    --dn "C=GB,O=CylindricCorp,CN=CylindricCorp Root CA" \
    --tokenType soft \
    --keyspec 4096 \
    --keytype RSA \
    -v 365 \
    --policy 2.5.29.32.0 \
    -s SHA256WithRSA \
    --tokenPass null
```

# Create an Intermediate Signing CA

```sh
ROOTCA=$(bin/ejbca.sh ca info SandboxRootCA|grep "CA ID"|tr -dc '0-9')

bin/ejbca.sh ca init \
    --caname SandboxIssuingCA \
    --dn "C=GB,O=CylindricCorp,CN=CylindricCorp Issuing CA" \
    --tokenType soft \
    --keyspec 4096 \
    --keytype RSA \
    -v 365 \
    --policy 2.5.29.32.0 \
    -s SHA256WithRSA \
    --tokenPass null \
    --signedby $ROOTCA
```