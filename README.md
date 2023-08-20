<h1>Give your Beepy a fresh start!</h1>
<h3>DON'T follow the "official" steps in the link below, but instead take the steps I'm sharing here to install Ardangelo aka Excel's drivers and firmware using my custom script.</h3>
This started as a personal log of what build files worked and documentation for the https://beepy.sqfmi.com/docs/getting-started

----------------------------------------------------------------
<h1>Software Setup</h1>
<h3>Use the Raspberry Pi Imager tool to flash an SD card with the latest Raspberry Pi OS image</h3>

Choose the Raspberry Pi OS Lite (32-bit) image
Click the gear icon ⚙ to also setup WiFi and enable SSH
SSH into the Pi and update the kernel and reboot
```
sudo apt-get update && sudo apt-get install raspberrypi-kernel
sudo shutdown -r now
```
----------------------------------------------------------------
<h2>Now you can either compile the firmware yourself or download from this repository</h2>
<h3>Steps to compile new firmware from git:</h3>
(via SSH to beepy or another Raspberry Pi)

```
sudo apt-get install binutils-arm-none-eabi cmake gcc-arm-none-eabi
sudo apt-get install git
git clone https://github.com/ardangelo/beepberry-rp2040
cd beepberry-rp2040
git submodule update --init
cd 3rdparty/pico-sdk
git submodule update --init
cd ../../
mkdir build
cd build
cmake -DPICO_BOARD=beepberry -DCMAKE_BUILD_TYPE=Debug ..
make
```
Resulting firmware file will be located:
`~/beepberry-rp2040/build/app/i2c_puppet.uf2`

(or you can download my [`ic2_puppet.uf2`](https://github.com/wildoracle/beepy/raw/main/i2c_puppet.uf2) file from this repository)

----------------------------------------------------------------
<h2>Firmware Update</h2>
To update the Beepy's firmware:

Slide the power switch off (left if facing up)
Connect the Beepy to your computer via USB-C
While holding the "End Call" key (top right on the keypad), slide the power switch on
The Beepy will present itself as a USB mass storage device, drag'n'drop the new firmware (*.uf2) into the drive and it will reboot with the new firmware.

-----------------------------------------------------------------
<h2>Driver Install</h2>

After reboot, SSH into the Pi again and run this customized setup script that loads [keyboard](https://github.com/ardangelo/beepberry-keyboard-driver) and [display](https://github.com/ardangelo/sharp-drm-driver) drivers from Ardangelo
```
curl -s https://raw.githubusercontent.com/wildoracle/beepy/main/setup.sh | bash
```
You should now see `beepberry-keyboard-driver` and `sharp-drm-driver` when you type `ls`

----------------------------------------------------------------
<h2>Steps to copy keymap to shared location:</h2>

```
sudo -i
cd /home/<USERNAME>/beepberry-keyboard-driver
```
Copy keymap to shared location
```
mkdir -p /usr/share/keymaps/
cp ./beepy-kbd.map /usr/share/keymaps/
chmod 644 /usr/share/keymaps/beepy-kbd.map
```
Copy init.d file
```
cp ./init/S01beepykbd /etc/init.d/S01beepykbd
chmod 755 /etc/init.d/S01beepykbd
```
Update init.d file with header info so we can load it on boot
```
sed -i '2 i\### BEGIN INIT INFO\
# Provides:          S01beepykbd\
# Required-Start:    $all\
# Required-Stop:\
# Default-Start:     2 3 4 5\
# Default-Stop:\
# Short-Description: Loads Beepy Keymap...\
### END INIT INFO' /etc/init.d/S01beepykbd
```
Loads the new service file
```
systemctl daemon-reload
```
Start it - you can test it working now
```
systemctl start S01beepykbd
```
Enable so it loads on boot
```
systemctl enable S01beepykbd
```

-----------------------------------------------------------------
<h2>Reload keymap</h2>
To reload the keymap:
I'll assume you've followed the guide and installed it to 

`/usr/share/keymaps/beepy-kbd.map`

if that is the case try this 

`sudo loadkeys /usr/share/keymaps/beepy-kbd.map`

this will manually load the keymap, fixing your alt and sym keys

-----------------------------------------------------------------
<h2>But wait... there's more!</h2>
Is your keyboard still not working right?
These commands might help?

`sudo apt-get install linux-headers`

-----------------------------------------------------------------
<h2>Previous driver layout</h2>

```
 *	               BBQ20KBD PMOD KEYBOARD LAYOUT
 *
 *	+------+-----+----+----+----+----+----+-----+-----+-------+
 *	|      |          |BR     ↑TPY-       |           |       |
 *	| Ctrl |   PgDn   |←TPX- BL(HOME)TPX+→|   PgUp    | MENU  |
 *	|      |          |       ↓TPY+       |           |       |
 *	+------+-----+----+----+----+----+----+-----+-----+-------+
 *	|                                                         |
 *	+------+-----+----+----+----+----+----+-----+-----+-------+
 *	|#     |1    |2   |3   |(   |   )|_   |    -|    +|      @|
 *	|  Q   |  W  | E  | R  | T  |  Y |  U |  I  |  O  |   P   |
 *	|      |     |PgDn|PgUp|   \|UP  |^   |=    |{    |}      |
 *	+------+-----+----+----+----+----+----+-----+-----+-------+
 *	|*     |4    |5   |6   |/   |   :|;   |    '|    "|    ESC|
 *	|  A   |  S  | D  | F  | G  |  H |  J |  K  |  L  |  BKSP |
 *	|     ?|     |   [|   ]|LEFT|HOME|RGHT|V+   |V-   |DLT    |
 *	+------+-----+----+----+----+----+----+-----+-----+-------+
 *	|      |7    |8   |9   |?   |   !|,   |    .|    `|       |
 *	|LFTALT|  Z  | X  | C  | V  |  B |  N |  M  |  $  | ENTER |
 *	|      |   K+|  K-|   °|   <|DOWN|>   |MENU |Vx   |       |
 *	+------+-----+----+----+----+----+----+-----+-----+-------+
 *	|            |0   |TAB                |     |             |
 *	| LEFT_SHIFT | ~  |       SPACE       |RTALT| RIGHT_SHIFT |
 *	|            |  Kx|                  &|     |             |
 *	+------------+----+-------------------+-----+-------------+
 *
```
-----------------------------------------------------------------

<h2>Apps and Tools</h2>

[Install fbterm](https://gist.github.com/charlestsai1995/54ab65a87e2e063ea25eb3aec4193fe1) for a terminal with better font options - but currently working out issues with the way it functions with keyboard and display

[Install cmatrix](https://www.linuxfordevices.com/tutorials/linux/install-cmatrix) for an appropriate "screensaver"!

[Install googler](https://lindevs.com/install-googler-on-raspberry-pi/) 
```
GOOGLER_VERSION=$(curl -s "https://api.github.com/repos/jarun/googler/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')
sudo curl -o /usr/local/bin/googler "https://raw.githubusercontent.com/jarun/googler/v${GOOGLER_VERSION}/googler"
sudo chmod a+x /usr/local/bin/googler
```
Check the installed version
```
googler --version
```
Testing Googler
Execute the `googler` command without any arguments.
Googler will prompt you for a search term. Type something and press Enter key. It will print search results. Press CTRL+C to quit Googler.
