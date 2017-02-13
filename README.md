# docker-rpi
In this tutorial I will go over the different steps involved in setting up a Docker swarm on a cluster of Raspberry Pi (2)'s.
## Prerequisites
Before setting up a Docker swarm you will need to install your RPi's properly. In order to do this follow the steps listed below.
Because I personally had a lot of trouble with SD-cards going bad in RPi's I rather prefer to use usb-drives as main storage drive. How you can do this is discribed below the steps involving the setup of the RPi's.
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
....2. Navigate to the bottom of the file and add the following lines:
....interface eth0
....static ip_address=192.168.0.10/24
....static routers=192.168.0.1
....static domain_name_servers=192.168.0.1
