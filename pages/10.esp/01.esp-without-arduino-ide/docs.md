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

## Setup The Pre-Requisites
Much of the trickery comes in the repository setup, but we'll cover that later. The main thing to have installed is the ESP-IDF. This lives outside of the project.

```sh
# Install all the general background cruft required by the toolchain.
apt install -y gcc git wget make libncurses-dev flex bison gperf python python-pip python-setuptools python-serial python-crypto python-future

# Make a "home" folder for the toolchain, this should probably be somewhere LSB-compliant, but I'm using /esp for simpler examples
mkdir -p /esp
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
cat << EOF > /etc/profile.d/esp.sh
#!/bin/bash
export PATH=$PATH:/esp/xtensa-esp32-elf/bin
export IDF_PATH=/esp/esp-idf
EOF
chmod a+x /etc/profile.d/esp.sh
/etc/profile.d/esp.sh
```

## Resources
* The official [ESP-IDF Programming Guide](https://docs.espressif.com/projects/esp-idf/en/latest/) home.
* The official [getting started](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html) pages for ESP-IDF.