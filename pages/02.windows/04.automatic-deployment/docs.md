---
title: 'Automatic Deployment'
taxonomy:
    category:
        - Windows
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
* Windows Server 2016 SERVERDATACENTERCORE

## License Keys
For machines that will be later activated using KMS or ADBA, the product key should be left out.
```
<UserData>
  <ProductKey>
    <WillShowUI>OnError</WillShowUI>
  </ProductKey>
</UserData>
```
For machines using the 180-day trial, the key should be specified:
```
<UserData>
  <ProductKey>
    <Key>WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY</Key>
    <WillShowUI>OnError</WillShowUI>
  </ProductKey>
</UserData>
```

## UK Localisation
In the `Microsoft-Windows-International-Core-WinPE` component, set the values as follows:
```
<component xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
  <SetupUILanguage>
    <UILanguage>en-US</UILanguage>
  </SetupUILanguage>
  <InputLocale>0809:00000809</InputLocale>
  <SystemLocale>en-GB</SystemLocale>
  <UILanguage>en-US</UILanguage>
  <UILanguageFallback>en-GB</UILanguageFallback>
  <UserLocale>en-GB</UserLocale>
</component>
```
And again in the `oobeSystem`  settiongs pass:
```
<component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <InputLocale>e0809:00000809</InputLocale>
  <SystemLocale>en-GB</SystemLocale>
  <UILanguage>en-US</UILanguage>
  <UserLocale>en-GB</UserLocale>
</component>
```
Finally set the correct timezone within the `Microsoft-Windows-Shell-Setup` component:
```
<component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <TimeZone>GMT Standard Time</TimeZone>
</Component>
```
