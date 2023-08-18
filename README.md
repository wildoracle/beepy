# beepy
Personal build files and docs for https://beepy.sqfmi.com/docs/getting-started


Steps to compile new firmware from git:
(via SSH to beepy)

`sudo apt-get install gcc-arm-none-eabi`

`sudo apt-get install binutils-arm-none-eabi cmake`

`sudo apt-get install git`

`git clone https://github.com/ardangelo/beepberry-rp2040`

`cd beepberry-rp2040`

`git submodule update --init`

`cd 3rdparty/pico-sdk`

`git submodule update --init`

`cd ../../`

`mkdir build`

`cd build`

`cmake -DPICO_BOARD=beepberry -DCMAKE_BUILD_TYPE=Debug ..`

`make`

Resulting firmware file will be `~/beepberry-rp2040/build/app/i2c_puppet.uf2`


Software Setup
Use the Raspberry Pi Imager tool to flash an SD card with the latest Raspberry Pi OS image

Choose the Raspberry Pi OS Lite (32-bit) image
Click the gear icon ⚙ to also setup WiFi and enable SSH
SSH into the Pi and update the kernel and reboot

`sudo apt-get update && sudo apt-get install raspberrypi-kernel`

`sudo shutdown -r now`

After reboot, SSH into the Pi again and run the setup script

`curl -s https://raw.githubusercontent.com/wildoracle/beepy/main/setup.sh | bash`

Your Beepy is now ready, enjoy!
