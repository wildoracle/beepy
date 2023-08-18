# beepy
Personal build files and docs for https://beepy.sqfmi.com/docs/getting-started


----------------------------------------------------------------
Software Setup
Use the Raspberry Pi Imager tool to flash an SD card with the latest Raspberry Pi OS image

Choose the Raspberry Pi OS Lite (32-bit) image
Click the gear icon ⚙ to also setup WiFi and enable SSH
SSH into the Pi and update the kernel and reboot

`sudo apt-get update && sudo apt-get install raspberrypi-kernel`

`sudo shutdown -r now`

After reboot, SSH into the Pi again and run the setup script

`curl -s https://raw.githubusercontent.com/wildoracle/beepy/main/setup.sh | bash`

----------------------------------------------------------------
Steps to copy keymap to shared location:

`sudo -i`

`cd /home/<USERNAME>/beepberry-keyboard-driver`

Copy keymap to shared location

`mkdir -p /usr/share/keymaps/`

`cp ./beepy-kbd.map /usr/share/keymaps/`

`chmod 644 /usr/share/keymaps/beepy-kbd.map`

Copy init.d file

`cp ./init/S01beepykbd /etc/init.d/S01beepykbd`

`chmod 755 /etc/init.d/S01beepykbd`

Update init.d file with header info so we can load it on boot

`sed -i '2 i\### BEGIN INIT INFO\`
`# Provides:          S01beepykbd\`
`# Required-Start:    $all\`
`# Required-Stop:\`
`# Default-Start:     2 3 4 5\`
`# Default-Stop:\`
`# Short-Description: Loads Beepy Keymap...\`
`### END INIT INFO' /etc/init.d/S01beepykbd`

Loads the new service file

`systemctl daemon-reload`

Start it - you can test it working now

`systemctl start S01beepykbd`

Enable so it loads on boot

`systemctl enable S01beepykbd`

----------------------------------------------------------------
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

Resulting firmware file will be located:
`~/beepberry-rp2040/build/app/i2c_puppet.uf2`

----------------------------------------------------------------
Firmware Update
To update the Beepy's firmware:

Slide the power switch off (left if facing up)
Connect the Beepy to your computer via USB-C
While holding the "End Call" key (top right on the keypad), slide the power switch on
The Beepy will present itself as a USB mass storage device, drag'n'drop the new firmware (*.uf2) into the drive and it will reboot with the new firmware.
