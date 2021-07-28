---
title: 'Git Setup for Commit Signing'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - gpg
        - yubikey
        - wsl
        - git
---

# Using Yubikeys to sign git commits

## Pre-Requisites

* The public key should be installed and trusted, as per [02](../02.gpg/docs.md)

## Configure Git

Find the ID of the GPG key to use:

```
gpg --list-keys
-----------------------------------------------
pub   rsa4096 2021-07-24 [C]
      EF7C2A657A623928FBFD1E762D3F69C229E9DA52
uid           [ultimate] Your Name <someone@somewhere.nice>

pub   rsa4096 2021-07-24 [C]
      EA14073E363E13426E02460A12FC86C2B6B4F011
uid           [ultimate] Your Name <someone@somewhere.nice>
sub   rsa4096 2021-07-24 [S] [expires: 2022-07-24]
sub   rsa4096 2021-07-24 [E] [expires: 2022-07-24]
sub   rsa4096 2021-07-24 [A] [expires: 2022-07-24]
```

Make a note of the long hex id above the three sub-keys (`EA14...`)

Configure git: 
```
git config --global user.email someone@somewhere.nice
git config --global user.name "Your Name"
git config --global user.signingkey EA14073E363E13426E02460A12FC86C2B6B4F011
git config --global commit.gpgsign true
```

It might be necessary to tell git to use the correct GPG executable if it's not on the path:
```
# For Powershell:
git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe"

# For WSL:
git config --global gpg.program "/usr/bin/gpg"
```

## Test

```bash
mkdir ~/gpgtest
cd ~/gpgtest
git init
touch README.md
git add README.md

# This should prompt to enter the PIN for the Yubikey:
git commit -m "Testing without the key"
```

## Troubleshooting

If the Yubikey is not already inserted when it's needed, it's possible that gpg will get stuck in a state where it's asking for a specific Yubikey to be inserted, and even inserting the key continues to report an error such as 

```
gpg: OpenPGP card not available: General error
```

or

```
error: gpg failed to sign the data
fatal: failed to write commit object
```

In this case, it might be necessary to reset the gpg agent on the host:

```
gpgconf.exe--kill gpg-agent
```
