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

# MQTT

Decent Clients are:

* mqtt-explorer

Host: hassio.home.cylindric.net
Username: mqtt
Password: chocolatemqtt

# Specific configs

## Tasmo01 - Christmas Tree
* [172.29.14.103](http://172.29.14.103/)
* Module > Type = Sonoff Basic (1)
* Wifi -> Hostname = tasmo01
* MQTT -> Host = 172.29.14.116
* MQTT -> Port = 1883
* MQTT -> Client = Tasmota02
* MQTT -> User = mqtt
* MQTT -> Password = chocolatemqtt
* MQTT -> Topic = tasmota01
* Other -> Friend Name 1 = Tasmota01

Make sure to set `setoption19 1` in the console

## Tasmo02 - Snowman
* [172.29.14.106](http://172.29.14.106/)
* Module > Type = Sonoff Basic (1)
* Wifi -> Hostname = tasmo02
* MQTT -> Host = 172.29.14.116
* MQTT -> Port = 1883
* MQTT -> Client = Tasmota02
* MQTT -> User = mqtt
* MQTT -> Password = chocolatemqtt
* MQTT -> Topic = tasmota02
* Other -> Friend Name 1 = Tasmota02

Make sure to set `setoption19 1` in the console

## Tasmo04 - Window
* [172.29.14.104](http://172.29.14.104/)
* Module > Type = Sonoff Basic (1)
* Wifi -> Hostname = tasmo04
* MQTT -> Host = 172.29.14.116
* MQTT -> Port = 1883
* MQTT -> Client = Tasmota04
* MQTT -> User = mqtt
* MQTT -> Password = chocolatemqtt
* MQTT -> Topic = tasmota04
* Other -> Friend Name 1 = Tasmota04

Make sure to set `setoption19 1` in the console
