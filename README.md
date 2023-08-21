<h1>Give your Beepy a fresh start!</h1>
<h3>The "official" steps will load an earlier version of the drivers and firmware, but instead you can take the steps I'm sharing here to install Ardangelo aka Excel's drivers and firmware using my custom script.</h3>

This started as a personal log of what build files worked and documentation for the [Beepy](https://beepy.sqfmi.com/)

But then developed into little more than a "guide for dummies" version of [the Ardangelo method](https://ardangelo.github.io/beepy-ppa/)

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
<h3>Removing/cleaning old source builds of drivers</h3>
If you already ran the "official" process and installed those drivers, and you don't want to start over with a fresh SD flash of the system, you can perform the following steps to remove the old drivers first before you proceed with installing the Ardangelo drivers.

If you have not installed previous versions of the drivers from source, disregard this section.

<h4>Driver packages in the Driver install options further down this page will detect if one of these old modules is installed and cancel installation of the package.</h4>
<h4>The `bbqX0kbd` driver has been renamed to `beepy-kbd`, and `sharp` to `sharp-drm` - with a note from the developer:</h4>
"I renamed `bbqX0kbd` to `beepy-kbd` because: 1) the driver is heavily customized for Beepy and 2) The Debian DKMS system does not allow capital X in driver package names. 
I renamed `sharp` to `sharp-drm` because it diverged heavily from the original `sharp` driver, I rewrote it entirely to use the Linux DRM system.
And as a side effect, this allows the driver packages to check for any previous source installs of the drivers. There are a lot of people here with drivers built from source and having the old versions lying around can eventually be an issue for the firmware and software updates."

--------------------------------------------------------------

1. Remove the following files:
```
/lib/modules/<uname>/extra/bbqX0kbd.ko*
/lib/modules/<uname>/extra/sharp.ko*
/boot/overlays/i2c-bbqX0kbd.dtbo
/boot/overlays/sharp.dtbo
```

2. Rebuild the module list:
```
depmod -a
```

3. Remove the following lines from `/boot/config.txt`:
```
dtoverlay=bbqX0kbd,irq_pin=4
dtoverlay=sharp
```

4. Remove the following lines from `/etc/modules`:
```
bbqX0kbd
sharp
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
<h3>To update the Beepy's firmware:</h3>

Slide the power switch off (left if facing up)
Connect the Beepy to your computer via USB-C
While holding the "End Call" key (top right on the keypad), slide the power switch on
The Beepy will present itself as a USB mass storage device, drag'n'drop the new firmware (*.uf2) into the drive and it will reboot with the new firmware.

----------------------------------------------------------------
<h2>Driver Install, preferred method</h2>
(From https://ardangelo.github.io/beepy-ppa/)

After reboot, SSH into the Pi again and run these commands to add the Ardangelo repository to APT and install the drivers:
```
curl -s --compressed "https://ardangelo.github.io/beepy-ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/beepy.gpg >/dev/null
sudo curl -s --compressed -o /etc/apt/sources.list.d/beepy.list "https://ardangelo.github.io/beepy-ppa/beepy.list"
sudo apt update
sudo apt-get -y install beepy-kbd sharp-drm
```
You should now see `beepberry-keyboard-driver` and `sharp-drm-driver` when you type `ls`

The keyboard and firmware driver package will run a preinstall check to ensure that the Beepy firmware is compatible with the driver.

If the installed firmware is detected as incompatible, the installation will be canceled.

A link to a compatible firmware release will be output as part of the error message.

-----------------------------------------------------------------
<h2>Driver Install, backup method</h2>

Alternatively, you can try the following process which is a custom version of the official setup script from SQFMI.

After reboot, SSH into the Pi again and run this customized setup script that loads [keyboard](https://github.com/ardangelo/beepberry-keyboard-driver) and [display](https://github.com/ardangelo/sharp-drm-driver) drivers from Ardangelo
```
curl -s https://raw.githubusercontent.com/wildoracle/beepy/main/setup.sh | bash
```
You should now see `beepberry-keyboard-driver` and `sharp-drm-driver` when you type `ls`

----------------------------------------------------------------
<h2>These remaining steps are to troubleshoot and fix common problems</h2>
<h3>Steps to copy keymap to shared location:</h3>

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
<h3>Reload keymap</h3>
To reload the keymap:
I'll assume you've followed the guide and installed it to 

`/usr/share/keymaps/beepy-kbd.map`

if that is the case try this 

`sudo loadkeys /usr/share/keymaps/beepy-kbd.map`

this will manually load the keymap, fixing your alt and sym keys

-----------------------------------------------------------------
<h2>But wait... there's more!</h2>

Is your keyboard still not working right?
These commands might help...

Reinstall linux headers?

`sudo apt-get install linux-headers`

Need to see what the keycode is for any key?

Type `showkeys` then press a key

-----------------------------------------------------------------
<h2>Previous driver layout reference</h2>

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
