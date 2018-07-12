---
title: 'Automatic Deployment'
taxonomy:
    category:
        - Tech
---

Various tips about the automatic deployment process...

===

# Automatic Answer File
Things to do with the autounattend.xml file...

## Setting the Installed OS Edition
The Windows Server 2016 installation ISO comes with various editions of Windows on it - Standard, Core, Data Centre, etc.
There is a `component` element with the name "Microsoft-Windows-Setup" that contains various target disk imaging options. One of these is "ImageInstall` and determines which version of the OS is deployed.

```
<InstallFrom>
  <MetaData wcm:action="add">
    <Key>/IMAGE/NAME </Key>
    <Value>Windows Server 2016 SERVERSTANDARD</Value>
  </MetaData>
</InstallFrom>
```

Note the space after the Key `/IMAGE/NAME ` - it is important.
Valid options for `<Value>` are:

* Windows Server 2016 SERVERSTANDARD
* Windows Server 2016 SERVERSTANDARDACORE
* Windows Server 2016 SERVERDATACENTER
