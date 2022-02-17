# Realtek RTL8811CU/RTL8821CU USB wifi adapter driver version 5.4.1 for Linux 4.4.x up to 5.x

Before build this driver make sure `make`, `gcc`, `linux-header`/`kernel-devel`, `bc` and `git` have been installed.

## First, clone this repository
```
mkdir -p ~/build
cd ~/build
git clone https://github.com/brektrou/rtl8821CU.git
```
## Check the name of the interface

Check the interface name of your wifi adapter using `ifconfig`. Usually, it will be wlan0 by default, but it may vary depends on the kernel and your device. On Ubuntu, for example, it may be named as wlx + MAC address. (https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/) 

If this is the case, you can either disable the feature following the link above, or replace the name used in the driver by

```
grep -lr . | xargs sed -i '' -e '/ifcfg-wlan0/!s/wlan0/<name of the device>/g'
```

## Build and install with DKMS

DKMS is a system which will automatically recompile and install a kernel module when a new kernel gets installed or updated. To make use of DKMS, install the dkms package.

### Debian/Ubuntu:
```
sudo apt-get install dkms
```
### Arch Linux/Manjaro:
```
sudo pacman -S dkms
```
To make use of the **DKMS** feature with this project, just run:
```
./dkms-install.sh
```
If you later on want to remove it, run:
```
./dkms-remove.sh
```

### Plug your USB-wifi-adapter into your PC
If wifi can be detected, congratulations.
If not, maybe you need to switch your device usb mode by the following steps in terminal:
1. find your usb-wifi-adapter device ID, like "0bda:1a2b", by type:
```
lsusb
```
2. switch the mode by type: (the device ID must be yours.)

Need install `usb_modeswitch` (Archlinux: `sudo pacman -S usb_modeswitch`)
```
sudo usb_modeswitch -KW -v 0bda -p 1a2b
systemctl start bluetooth.service - starting Bluetooth service if it's in inactive state
```

It should work.

### Make it permanent

If steps above worked fine and in order to avoid periodically having to make `usb_modeswitch` you can make it permanent (Working in **Ubuntu 18.04 LTS**):

1. Edit `usb_modeswitch` rules:

   ```bash
   sudo nano /lib/udev/rules.d/40-usb_modeswitch.rules
   ```

2. Append before the end line `LABEL="modeswitch_rules_end"` the following:

   ```
   # Realtek 8211CU Wifi AC USB
   ATTR{idVendor}=="0bda", ATTR{idProduct}=="1a2b", RUN+="/usr/sbin/usb_modeswitch -K -v 0bda -p 1a2b"
   ```   
Make sure to set your `ATTR{idVendor}` and the `-v` argument to the left portion of the output of lsusb device ID, and your `ATTR{idProduct}` and `-p` argument to the right portion of the lsusb device ID. For example (for the Cudy AC600 usb wifi adapter) the output from `lsusb` command looks like this:

   ```
   Bus 001 Device 016: ID 0bda:c811 Realtek Semiconductor Corp. 802.11ac NIC
   ```
   
then your configuration in `/lib/udev/rules.d/40-usb_modeswitch.rules` should be 

   ```
   # Realtek 8211CU Wifi AC USB
   ATTR{idVendor}=="0bda", ATTR{idProduct}=="c811", RUN+="/usr/sbin/usb_modeswitch -K -v 0bda -p c811"
   ```   


## Build and install without DKMS
Use following commands:
```
cd ~/build/rtl8821CU
make
sudo make install
```
If you later on want to remove it, do the following:
```
cd ~/build/rtl8821CU
sudo make uninstall
```
## Checking installed driver
If you successfully install the driver, the driver is installed on `/lib/modules/<linux version>/kernel/drivers/net/wireless/realtek/rtl8821cu`. Check the driver with the `ls` command:
```
ls /lib/modules/$(uname -r)/kernel/drivers/net/wireless/realtek/rtl8821cu
```
Make sure `8821cu.ko` file present on that directory

### Check with **DKMS** (if installing via **DKMS**):

``
sudo dkms status
``

### Monitor mode
Use the tool 'iw', please don't use other tools like 'airmon-ng'
```
iw dev wlan0 set monitor none
```





### Notes abeyt
```
https://forums.raspberrypi.com/viewtopic.php?t=309068
git clone https://github.com/brektrou/rtl8821CU.git
cd rtl8821CU
chmod +x dkms-install.sh
sudo ./dkms-install.sh

pi@raspberrypi:~/rtl8821CU $ sudo ./dkms-install.sh 
About to run dkms install steps...

Creating symlink /var/lib/dkms/rtl8821CU/5.4.1/source ->
                 /usr/src/rtl8821CU-5.4.1

DKMS: add completed.

Kernel preparation unnecessary for this kernel.  Skipping...

Building module:
cleaning build area...
'make' KVER=5.10.92-v7l+..................................................................................................................................
cleaning build area...

DKMS: build completed.

8821cu.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/5.10.92-v7l+/kernel/drivers/net/wireless/realtek/rtl8821cu/

depmod......

DKMS: install completed.
Finished running dkms install steps.

pi@raspberrypi:~/rtl8821CU $ sudo modprobe -v 8821cu
insmod /lib/modules/5.10.92-v7l+/kernel/drivers/net/wireless/realtek/rtl8821cu/8821cu.ko 
pi@raspberrypi:~/rtl8821CU $ sudo iw dev
phy#1
	Interface wlx54c9e0001bc6
		ifindex 4
		wdev 0x100000001
		addr 54:c9:e0:00:1b:c6
		type managed
		txpower 12.00 dBm

ip link set wlx54c9e0001bc6 down
iw wlx54c9e0001bc6 set monitor none
ip link set wlx54c9e0001bc6 up
pi@raspberrypi:~/rtl8821CU $ sudo iw dev
phy#1
	Interface wlx54c9e0001bc6
		ifindex 4
		wdev 0x100000001
		addr 54:c9:e0:00:1b:c6
		type monitor
		txpower 12.00 dBm

```
