# docker-rpi
In this tutorial I will go over the different steps involved in setting up a Docker swarm on a cluster of Raspberry Pi (2)'s.
## Prerequisites
Before setting up a Docker swarm you will need to install your RPi's properly. In order to do this follow the steps listed below.
Because I personally had a lot of trouble with SD-cards going bad in RPi's I rather prefer to use a USB drive as main storage drive. How you can do this is discribed below the steps involving the setup of the RPi's.
### Setup your RPi's
1. I recommend starting with a clean installation of Raspbian. If you want to use usb drives go to the section below.
2. After this you always should take a few basic steps:
  1. Starting with security change the default password to something better.
  2. After doing this use the command **sudo raspi-config** and set the memory split to 16, more isn't needed because the system is running headless (no gui).
  3. Change the hostname of the RPi, this involves changing two files:
    1. /etc/hostname --this is where your hostname is stored
    2. /etc/hosts --this is where your hostname is linked to the loopback address (127.0.0.1)
  4. Set the IP-address to a valid address in your range.
    1. Open the "/etc/dhcpcd.conf" file in your favourite text editor.
    2. Navigate to the bottom of the file and add the following lines (change them for your situation):
    ```
    interface eth0
    static ip_address=192.168.0.10/24
    static routers=192.168.0.1
    static domain_name_servers=192.168.0.1
    ```
### Boot from USB drive
In order to boot from a USB drive you also need to have an SD-card.
1. If you have a running RPi first check if your usb-drive can be accessed by the system.
2. Flash the image to your USB drive.
3. Open the cmdline.txt in your boot partition on your SD-card and edit the **root=** parameter to **root=/dev/sda2** (you need to use /dev/sda2 because of the boot partition on the USB drive) and add **rootwait text** to the end. My cmdline.txt file contained this afterwards:
```
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/sda2 rootfstype=ext4 rootwait text
```
4. After booting your RPi you need to expand the filesystem of the USB drive. Because the option in raspi-config doesn't work in this config you will need to do this manually.
  1. If you started with a clean image and nothing is installed yet just go allong, otherwise I would recommend to backup your usb drive.
  2. Use **sudo fdisk /dev/sda**
  3. After this type p to list the partition table. You should see 3 partitions: 1->boot, 2->linux, 3->swap. Take a note of the start number of partition 2.
  4. Now you need to delete partition 2 and 3, in order to do this type d and then type 2 after doing this repeat this for partition 3.
  5. After doing this you can resize the main partition. Type n and choose for primary partition by typing p. When asked for a partition number type 2. Next you need to enter the first sector, enter the number you previously took a note on. After this you need to enter the last sector, the default option will be using the remaining size of the partition so hit enter to accept this.
  5. To save the configuration type w.
  6. Reboot the system after this: **sudo reboot**
  7. After rebooting the system type: **sudo resize2fs /dev/sda2**. This could take long depending on the size and speed of the USB drive.
  8. You can check if the full capacity is used with **df -h**
