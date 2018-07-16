---
title: 'AD Tips'
taxonomy:
    category:
        - Windows
    tag:
        - ad
---

Some general tips for AD.

===
# Change Default Computer OU

This will cause newly-joined computers to be placed into the “InitialOU” container.

    redircmp OU=InitialOU,DC=patricio,dc=local
