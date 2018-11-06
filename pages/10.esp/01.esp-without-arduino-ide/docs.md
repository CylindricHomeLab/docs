---
title: 'ESP Without The Arduino IDE'
taxonomy:
    category:
        - electronics
    tag:
        - esp
        - electronics
---

## Introduction
As the ESP is so much more powerful than most Arduino devices, it may be desirable to program in an IDE other than the Arduino IDE. Another use case is automated testing, which is difficult if it is necessary to launch a graphical IDE.

Fortunately, Espressif have made available the entire toolchain for building and uploading sketches to the ESP, which is of course how the IDE plugins do it too, and that is the Espressif IDF.

These instructions are all for working in Ubuntu under Windows 10 WSL. They should be pretty much identical for native Ubuntu, and very similar for other Linux variants. Mac is also supported, but I don't have a Mac to test it on and work out the differences.

## Espressif ESP-IDF

Using purely native-ESP code is relatively simple, and the many and varied examples shipped with the IDF provide a starting-off point, however the Arduino ecosystem is so large, it's a shame not to utilise all the great libraries already out there. This means there is still a requirement to have the Arduino code ecosystem, but now without the IDE.

## Set up The Pre-Requisites
Much of the trickery comes in the repository setup, but we'll cover that later. The main thing to have installed is the ESP-IDF. This lives outside of the project.

```sh
# Install all the general background cruft required by the toolchain.
sudo apt install -y gcc git wget make libncurses-dev flex bison gperf python python-pip python-setuptools python-serial python-crypto python-future

# Make a "home" folder for the toolchain, this should probably be somewhere LSB-compliant, but I'm using /esp for simpler examples
sudo mkdir -p /esp

# Some build tasks require write access to this directory. You'll want to sort out some sort of scheme for that to suit your needs, I just make it accesible to me.
sudo chown -R cylindric:root /esp

# Continue in our new home...
cd /esp

# Download the XTensa tool chain, and then extract it
wget https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz
tar xf xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz
rm xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz

# Download the ESP-IDF tools, including the git submodules, and then the Python dependencies
git clone --recursive https://github.com/espressif/esp-idf.git
cd /esp/esp-idf
python -m pip install --user -r requirements.txt

# Some environment variables are required, so it's useful to have them set by default
sudo cat << EOF > /etc/profile.d/esp.sh
#!/bin/bash
export PATH=$PATH:/esp/xtensa-esp32-elf/bin
export IDF_PATH=/esp/esp-idf
EOF

sudo chmod a+x /etc/profile.d/esp.sh
/etc/profile.d/esp.sh
```

## Set up a Project

My Arduino projects generally have two main components, one consisting of schematics, PCB layouts and enclosure designs, and then another for the firmware. Here we create a basic Awesome project along those lines. I'll assume we're in our home directory for this...

```sh
# Create the base directory, a hardware directory and a firmware directory.
mkdir -p ~/Awesome/{hardware,firmware}
cd ~/Awesome/firmware

# Now comes the ESP magic bit. First of all, grab a suitable Makefile from the examples.
# It doesn't need modifying, as it comes is fine.
cp $IDF_PATH/examples/get-started/hello_world/Makefile .

# Now we need a "main" directory where the code will live. It must be called "main", and contain a "main.cpp".
# Details of the main.cpp come later...
mkdir -p main
touch main/main.cpp

# Next we need a directory to hold the Arduino core. That's also where any other Arduino libraries will need to go.
mkdir -p components
cd components
git clone https://github.com/espressif/arduino-esp32.git esp32
cd esp32
git submodule update --init --recursive
cd ../..
```

In order to actually compile for ESP, there are a ton of settings that need to be present in a file called `sdkconfig`. Fortunately, Espressif have included a utiltity to do this for us, so we can simply run a GUI.

```sh
cd ~/Awesome.firmware
make menuconfig
```

The key item in here if you want everything to behave like the Arduino sketches do is to tell the ESP core to call the `setup()` and `loop()` functions as expected. So make sure to change:

* Arduino Configuration -> Autostart Arduino setup and loop on boot -> Enabled

Now we can put in a simple `main.cpp` to test things out:

```cpp
#include "Arduino.h"

void setup() {
}

void loop() {
}
```

```sh
make
```

For convenience, I also create an empty `main.ino`, so that if I do want to use the Arduino IDE for something, like a custom plugin tool or the Serial monitor, it doesn't complain about sketches in incorrectly named folders.

## Resources
* The official [ESP-IDF Programming Guide](https://docs.espressif.com/projects/esp-idf/en/latest/) home.
* The official [getting started](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html) pages for ESP-IDF.