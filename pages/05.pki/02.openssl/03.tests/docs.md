---
title: 'Useful Tests'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - openssl
---

Here are some useful commands for testing, verifying and interrogating certificates.

===

Simple verification of a certificate

## Retrieve a Remote Certificate

This gets the details of a certificate from a remote server. The little `echo | ` prefix simply prevents the potential timeout as openssl tries to actually open a connection to the remote port.

```sh
echo | openssl s_client -showcerts -connect www.cylindric.net:443
```

If the server requires the use of SNI, for example a shared server or something behind a load-balancer etc, it may be necessary to add the servername parameter:

```sh
echo | openssl s_client -showcerts -connect www.cylindric.net:443 -server www.cylindric.net
```

Right at the top is the certificate chain, then the actual certificates returned.

## Extract a Certificate from s_client

This takes the output from the `openssl s_client` command and extracts the actual certificate, which can be saved and used for further testing, or piped straight into subsequent `openssl` commands.

```sh
echo | openssl s_client -connect www.cylindric.net:443 -servername www.cylindric.net 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certificate.crt
```

If you want the whole certificate chain instead of just the final leaf certificate, add `-showcerts` to the command:

```sh
echo | openssl s_client -showcerts -connect www.cylindric.net:443 -servername www.cylindric.net 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certificate.crt
```

This gets the intermediate certificates:

```sh
echo | openssl s_client -showcerts -connect www.cylindric.net:443 -servername www.cylindric.net 2>&1 | sed -n '/-----BEGIN/,/-----END/p' > chain.crt
```

## View the Chain of Trust

This simply prints the certificate chain for the provided cert. You'll probably need the certificate chain too.

```sh
openssl verify -show_chain -CAfile chain.crt certificate.crt 
```

# References

https://www.feistyduck.com/library/openssl-cookbook/online/ch-testing-with-openssl.html
https://akshayranganath.github.io/OCSP-Validation-With-Openssl/
