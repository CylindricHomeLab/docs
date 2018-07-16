---
title: 'Licensing Tips'
taxonomy:
    category:
        - Tech
    tag:
        - licensing
---

Some general tips for Windows Licensing.

===

Change Eval to Standard

This changes an evaluation edition of Server 2016 to Standard edition, ready for KMS activation:

    dism /online /set-edition:ServerStandard /productkey:WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY /accepteula

Reboot, and then apply the KMS key

    slmgr /ipk WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY

Finally activate

    slmgr /ato
