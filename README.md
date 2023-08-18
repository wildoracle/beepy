# beepy
Personal build files and docs for https://beepy.sqfmi.com/docs/getting-started


Steps to compile new firmware from git:
(via SSH to beepy)

`sudo apt-get install gcc-arm-none-eabi`

`sudo apt-get install binutils-arm-none-eabi cmake`

`sudo apt-get install git`

`git clone https://github.com/sqfmi/i2c_puppet`

`cd i2c_puppet`

`git submodule update --init`

`cd 3rdparty/pico-sdk`

`git submodule update --init`

`cd ../../`

`mkdir build`

`cd build`

`cmake -DPICO_BOARD=beepy ../`

`make`
