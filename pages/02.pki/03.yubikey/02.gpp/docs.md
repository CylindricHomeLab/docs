---
title: 'Yubikey Setup for GPG'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - gpg
        - yubikey
---

# Using GPG with Git and Yubikey

https://andrewmatveychuk.com/how-to-sign-you-commits-with-gpg-git-and-yubikey/

## Getting Started

1. Install [GPG4Win](https://gpg4win.org/download.html)
1. Install the [YubiKey Manager](https://www.yubico.com/support/download/yubikey-manager/)
1. Install the [YubiKey SmartCard Minidriver](https://www.yubico.com/support/download/smart-card-drivers-tools/)

## GPG Setup

First create a temporary working directory and set some secure configuration options for GPG:

```powershell
$env:GNUPGHOME='C:\Temp\gpg'
mkdir $env:GNUPGHOME
cd $env:GNUPGHOME
Invoke-WebRequest -URI https://raw.githubusercontent.com/drduh/config/master/gpg.conf -OutFile gpg.conf
```

## GPG Master Key

The first key to generate is the master key. This should be stored somewhere offline and safe. It is only needed to create new subkeys, so doesn't need to be conveniently available.

Start by having a secure passphrase to access the master key. This can be generated or something you know, but it should be very secure.

```powershell
C:\Temp\gpg> gpg --gen-random --armor 0 24
p7SX3VztS6RE3+yxBpykcCNpWcNb1m6t
C:\Temp\gpg> 
```

Now generate a new master key:

```text
gpg --expert --full-generate-key
```
* Select key type `(8) RSA`
* Disable signing with `(S) Toggle the sign capability`
* Disable encryption with `(E) Toggle the encrypt capability`
* Select `(Q) Finished`
* Enter the largest usable key size `4096`
* Set to never expire `0`
* Accept the confirmation and enter the identity details

We want to keep the key ID for later use, so store it in a variable:

```
$KEYID="0x65F388C266B4F021"
```

## GPG Signing, Encryption and Authentication Sub-Keys

Next we have to add some sub-keys for signing and encryption etc, so log into the keyring to begin adding them:
```
C:\Temp\gpg> gpg --expert --edit-key $KEYID
Secret key is available
...
gpg>
```

First add a signing key:
```
gpg> addkey
```
* Select key type `(4) RSA (sign only)`
* Enter the largest usable key size `4096`
* Set to expire in 1 year `1y`
* Accept the confirmation to create the key

Repeat the `addkey` process to add an encryption key in the same way, but this time choose `(6) RSA (encrypt only)`.

Repeat the `addkey` process to add an authentication key in the same way, but:

* Select key type `(8) RSA`
* Disable signing with `(S) Toggle the sign capability`
* Disable encryption with `(E) Toggle the encrypt capability`
* Enable authentication with `(A) Toggle the authenticate capability`
* Select `(Q) Finished`
* Enter the largest usable key size `4096`
* Set to never expire `0`
* Accept the confirmation and enter the identity details

Finally save all the keys to the keyring:
```
gpg> save
C:\Temp\gpg>
```

## Additional Identity for Git
```
C:\Temp\gpg> gpg --expert --edit-key $KEYID
Secret key is available
...
gpg> adduid
gpg> trust
gpg> uid
[ultimate] (1). Cylindric <mark@hanfordonline.co.uk>

gpg> uid 2
[ultimate] (1). Cylindric <mark@hanfordonline.co.uk>
[ultimate] (2)* Mark Hanford <mark@hanfordonline.co.uk>

gpg> primary
[ultimate] (1)  Cylindric <mark@hanfordonline.co.uk>
[ultimate] (2)* Mark Hanford <mark@hanfordonline.co.uk>

gpg> save
C:\Temp\gpg>
```

## Verification
```
C:\Temp\gpg> gpg -K
-----------------------------
sec   rsa4096/0x22FC88C2B6B4F011 2021-07-24 [C]
      Key fingerprint = EA14 073E 363E 13E2 6E08  460A 22FC 88C2 B6B4 F011
uid                   [ultimate] Mark Hanford <mark@hanfordonline.co.uk>
uid                   [ultimate] Cylindric <mark@hanfordonline.co.uk>
ssb   rsa4096/0xC03E98899250D6A4 2021-07-24 [S] [expires: 2022-07-24]
ssb   rsa4096/0xE283E5E8939DBE62 2021-07-24 [E] [expires: 2022-07-24]
ssb   rsa4096/0xE9BEA82296C7EB05 2021-07-24 [A] [expires: 2022-07-24]
```
There should be two identities and three sub-keys with a single type each, `[S]`. `[E]` and `[A]`.

## Export

Export the master key and the subkeys

```
C:\Temp\gpg> gpg -o "G:\Keys\gpg\mastersub.gpg" --armor --export-secret-keys $KEYID
C:\Temp\gpg> gpg -o "G:\Keys\gpg\sub.gpg" --armor --export-secret-subkeys $KEYID
```

## Revocation Certification

Generate a revocation cert in case the keys ever need to be revoked:
```
C:\Temp\gpg> gpg --output $env:GNUPGHOME/revoke.asc --gen-revoke $KEYID
```
Copy this revocation key somewhere safe.

## Backup
The whole master keyring and config should be backed up somewhere safe and offline. Once they are moved to the YubiKey they cannot be recovered.

## Export Public Key
```
C:\Temp\gpg> gpg -o "C:\somewhere\pubkey.gpg" --armor --export $KEYID
C:\Temp\gpg> gpg --keyserver pgp.mit.edu --send-key $KEYID
C:\Temp\gpg> gpg --keyserver hkps://keyserver.ubuntu.com:443 --send-key $KEYID
```

## Transfer Keys to the YubiKey
```
C:\Temp\gpg> gpg --edit-key $KEYID
```

First export the Signing key:
```
gpg> key 1
gpg> keytocard
Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1
gpg> key 1
```

Then the Encryption key:
```
gpg> key 2
gpg> keytocard
Please select where to store the key:
   (2) Encryption key
Your selection? 2
gpg> key 2
```

Then the Authentication key:
```
gpg> key 3
gpg> keytocard
Please select where to store the key:
   (3) Authentication key
Your selection? 3
gpg> key 3
```

Then save:
```
gpg> save
C:\Temp\gpg>
```

Verify that the keys are all moved to the YubiKey as indicated by the `ssb>` headings:
```
C:\Temp\gpg> gpg -K
C:/Temp/gpg/pubring.kbx
-----------------------------
sec   rsa4096/0x22FC88C2B6B4F011 2021-07-24 [C]
      Key fingerprint = EA14 073E 363E 13E2 6E08  460A 22FC 88C2 B6B4 F011
uid                   [ultimate] Mark Hanford <mark@hanfordonline.co.uk>
uid                   [ultimate] Cylindric <mark@hanfordonline.co.uk>
ssb>  rsa4096/0xC03E98899250D6A4 2021-07-24 [S] [expires: 2022-07-24]
ssb>  rsa4096/0xE283E5E8939DBE62 2021-07-24 [E] [expires: 2022-07-24]
ssb>  rsa4096/0xE9BEA82296C7EB05 2021-07-24 [A] [expires: 2022-07-24]
```

## Cleanup

* Securely purge the entire `$GNUPGHOME` directory.

## Client Setup

### Option 1 - From a copy of the public key file

```
gpg --import pubkey.gpg
```

### Option 2 - From a public key server

First, find the key id from the keyserver by [searching for it](http://pgp.mit.edu)

Import the public key from the local copy or the public keyserver:
```
gpg --keyserver pgp.mit.edu --recv 0x22FC88C2B6B4F011
```

The keyservers often seem to have problems, especially on the API interface, in which case it might be necessary to try alternatives or use the interactive download if there is one.

### Finally - Trust the key

Trust the key to level `5`:
```
gpg --edit-key 0x22FC88C2B6B4F011
...
gpg> trust
...
gpg> quit
```

