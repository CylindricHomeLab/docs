---
title: 'Tasmota'
taxonomy:
    category:
        - automation
    tag:
        - tasmota
        - sonnoff
---

# Basic Switch

1. Connect a button to the two pins furthest from the on-board button.
1. In the web config, set the module GPIO14 Sensor to "09 Switch 1"
1. Using the console, set the switch to be a button with `switchmode 3`

# Equipment Settings

1. Make sure relays stay off after a power outage with `PowerOnState 0`