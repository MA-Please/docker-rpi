# docker-rpi
In this tutorial I will go over the different steps involved in setting up a Docker swarm on a cluster of Raspberry Pi (2)'s.
## Prerequisites
Before setting up a Docker swarm you will need to install your RPi's properly. In order to do this follow the steps listed below.
Because I personally had a lot of trouble with SD-cards going bad in RPi's I rather prefer to use a USB drive as main storage drive. How you can do this is discribed below the steps involving the setup of the RPi's.
### Setup your RPi's
1. Download and flash the latest version of Raspbian (I prefer to use Raspbian Lite) on your SD-card.
2. Plug in your Pi and wait till it's running.
3. After this you always should take a few basic steps:
..1. Starting with security change the default password to something better.
..2. After doing this use the command **"sudo raspi-config"** and set the memory split to 16, more isn't needed because the system is running headless (no gui).
..3. Change the hostname of the RPi, this involves changing two files:
....1. /etc/hostname --this is where your hostname is stored
....2. /etc/hosts --this is where your hostname is linked to the loopback address (127.0.0.1)
..4. Set the IP-address to a valid address in your range.
....1. Open the "/etc/dhcpcd.conf" file in your favourite text editor.
....2. Navigate to the bottom of the file and add the following lines (change them for your situation):
.... interface eth0
.... static ip_address=192.168.0.10/24
.... static routers=192.168.0.1
.... static domain_name_servers=192.168.0.1
### Boot from USB drive
In order to boot from a USB drive you also need to have an SD-card.
1. If you have a running RPi first check if your usb-drive can be accessed by the system.
2. Flash the image to your USB drive.
3. Open the cmdline.txt in your boot partition on your SD-card and edit the "root=" parameter to "root=/dev/sda2" (you need to use /dev/sda2 because of the boot partition on the usb-drive) and add "rootwait text" to the end.
4. Insert your SD-card and USB drive in your RPi.
5. Resize the file system -> fdisk good luck
