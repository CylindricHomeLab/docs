---
title: 'Resetting the Yubikey'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - yubikey
---

# Resetting the Yubikey

1. Install the [YubiKey Manager](https://www.yubico.com/support/download/yubikey-manager/)
1. Install the [YubiKey SmartCard Minidriver](https://www.yubico.com/support/download/smart-card-drivers-tools/)

Add the path to YubiKey to the Path

```powershell
$env:Path += ";C:\Program files\Yubico\YubiKey Manager\"
ykman
```

With a key inserted, it should show up with the `list` command:

```powershell
C:\Temp> ykman list
YubiKey 5 NFC (5.2.7) [OTP+FIDO+CCID] Serial: 41274350
C:\Temp>
```

### Starting again
To start from scratch, reset the PIV module on the Yubikey:

```text
C:\Temp> ykman piv reset
WARNING! This will delete all stored PIV data and restore factory settings. Proceed? [y/N]: y
Resetting PIV data...
Success! All PIV data have been cleared from the YubiKey.
Your YubiKey now has the default PIN, PUK and Management Key:
        PIN:    123456
        PUK:    12345678
        Management Key: 010203040506070801020304050607080102030405060708
C:\Temp>
```

The PIN and PUK be set to something new and personal:

```text
C:\Temp> ykman piv access change-puk
Enter the current PUK:
Enter the new PUK:
Repeat for confirmation:
New PUK set.
C:\Temp> ykman piv access change-pin
Enter the current PIN:
Enter the new PIN:
Repeat for confirmation:
New PIN set.
C:\Temp>
```

Store these somewhere safe and secure, such as an offline KeePass or written on a piece of paper.
