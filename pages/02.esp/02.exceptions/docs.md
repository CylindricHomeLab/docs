---
title: 'Exceptions'
taxonomy:
    category:
        - electronics
    tag:
        - esp
        - electronics
---

## ESP Stack Trace
If the ESP code crashes, there should be a stack trace dumped out to Serial. As there aren't any symbols included in that, it's pretty much useless.

The [EspExceptionDecoder](https://github.com/me-no-dev/EspExceptionDecoder) is a little Arduino plugin that gives you a UI that you can paste the stack into, and in combination with the original Elf file will give detailed information about where the problem occurred.

Download the latest release from GitHub, and extract it into the Arduino tools folder. Should be something like `%HOMEPATH%\Documents\Arduino\tools`. The full path to the actual jar file will be something like `C:\Users\username\Documents\Arduino\tools\EspExceptionDecoder\tool\EspExceptionDecoder.jar`.

![EspExceptionDecoder](esp32-exception.png)