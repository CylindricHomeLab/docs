---
title: 'Yubikey Notes'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - yubikey
---

Various stuff about Yubikey and GPG.

===

Install [GPG4Win](https://www.gpg4win.org/), I'm using v3.1.13




## Create the new key ring
[Yubico docs](https://support.yubico.com/hc/en-us/articles/360013790259-Using-Your-YubiKey-with-OpenPGP)

Create a new key, select "RSA and RSA", 4096-bit for both key lengths, no expiry:

```text
gpg --expert --full-gen-key
```

Make a note of the key fingerprint, e.g. 24615ADD763DEA90
Remember the passphrase entered here.

## Add a new authentication key

```text
gpg --expert --edit-key 24615ADD763DEA90
```

Add the authentication key with `addkey`, chose option 8 "RSA (set your own capabilities)", then use `s`, `e` and `a` to toggle Sign and Encrypt off, and Authenticate on.

## Add a new encryption key

```text
gpg --expert --edit-key 24615ADD763DEA90
```

Add the authentication key with `addkey`, chose option 8 "RSA (set your own capabilities)", then use `s`, `e` and `a` to toggle Sign and Authentication off, and Encrypt on.

## Add a new signing key

```text
gpg --expert --edit-key 24615ADD763DEA90
```

Add the authentication key with `addkey`, chose option 8 "RSA (set your own capabilities)", then use `s`, `e` and `a` to toggle Encrypt and Authentication off, and Signing on.

## Backup the key to an ascii file

```text
gpg --export-secret-key --armor 24615ADD763DEA90 > secret_key_file.asc
```

## Import the key to the Yubikey

Plug in the Yubikey.

```text
gpg --edit-key 24615ADD763DEA90
```

```text
toggle
keytocard
y
1
```

Choose the `(1) Signature key` and enter the passphrase from the first steps here.
The Admin Pin is the Yubikey Admin PIN, by default 12345678.


Now select Key 1

```text
key 1
keytocard
```

Choose the `(2) Encryption key` and enter the passphrase from the first steps here.
The Admin Pin is the Yubikey Admin PIN, by default 12345678.

Now `save` and `quit`
