---
title: 'WSUS Tips'
taxonomy:
    category:
        - Tech
    tag:
        - wsus

---

Some general tips for WSUS.

===

## Start Update Run

Start a Windows Update run from a Windows 10 or Server 2016 client has changed, and is no longer `wuauclt /detectnow`

    UsoClient.exe startscan

## Get Windows Update Log

The files on-disk are *Event Tracing for Windows* logs, so can’t be viewed directly. Export them:

    Get-WindowsUpdateLog

If the server is not connected to the internet, copy the logs to a machine that is, and is running the same version of Windows, and then run the command there:

    Get-WindowsUpdateLog -EtlPath C:\Path\to\etl_files
