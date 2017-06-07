# docker-rpi
In this tutorial I will go over the different steps involved in setting up a Docker swarm on a cluster of Raspberry Pi (2)'s.
## Prerequisites
Before setting up a Docker swarm you will need to install your RPi's properly. In order to do this follow the steps listed below.
Because I personally had a lot of trouble with SD-cards going bad in RPi's I rather prefer to use a USB drive as main storage drive. How you can do this is discribed below the steps involving the setup of the RPi's.
### Setup your RPi's

  1. I recommend starting with a clean installation of Raspbian. If you want to use usb drives go to the section below.
  2. After this you always should take a few basic steps:
    1. Starting with security change the default password to something better.
    2. After doing this use the command `sudo raspi-config` and set the memory split to 16, more isn't needed because the system is running headless (no gui).
    3. Change the hostname of the RPi, this involves changing two files:
      1. /etc/hostname --this is where your hostname is stored
      2. /etc/hosts --this is where your hostname is linked to the loopback address (127.0.0.1)
    4. Set the IP-address to a valid address in your range.
      1. Open the "/etc/dhcpcd.conf" file in your favourite text editor.
      2. Navigate to the bottom of the file and add the following lines (change them for your situation):
      ```
interface eth0 static
ip_address=192.168.0.10/24
static routers=192.168.0.1 static
domain_name_servers=192.168.0.1
      ```

### Boot from USB drive

In order to boot from a USB drive you also need to have an SD-card.
  1. If you have a running RPi first check if your usb-drive can be accessed by the system.
  2. Flash the image to your USB drive.
  3. Open the cmdline.txt in your boot partition on your SD-card and edit the `root=` parameter to `root=/dev/sda2` (you need to use /dev/sda2 because of the boot partition on the USB drive) and add `rootwait text` to the end. My cmdline.txt file contained this afterwards:
  ```
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/sda2 rootfstype=ext4 rootwait text

  ```
  4. After booting your RPi you need to expand the filesystem of the USB drive. Because the option in raspi-config doesn't work in this config you will need to do this manually.
    1. If you started with a clean image and nothing is installed yet just go allong, otherwise I would recommend to backup your usb drive.
    2. Use `sudo fdisk /dev/sda`
    3. After this type p to list the partition table. You should see 3 partitions: 1->boot, 2->linux, 3->swap. Take a note of the start number of partition 2 .
    4. Now you need to delete partition 2 and 3, in order to do this type d and then type 2 after doing this repeat this for partition 3.
    5. After doing this you can resize the main partition. Type n and choose for primary partition by typing p. When asked for a partition number type 2. Next you need to enter the first sector, enter the number you previously took a note on. After this you need to enter the last sector, the default option will be using the remaining size of the partition so hit enter to accept this.
    5. To save the configuration type w.
    6. Reboot the system after this: `sudo reboot`
    7. After rebooting the system type: `sudo resize2fs /dev/sda2`. This could take long depending on the size and speed of the USB drive.
  8. You can check if the full capacity is used with `df -h`

## Install Docker

In this chapter I will go over the different steps involved in installing Docker on RPi.
  1. To install Docker you can use the script provided by and maintained by Docker. Type `curl -sSL get.docker.com | sh` to start the script. This will take a while to run.
  2. After the installation you need to add your user to the docker group to be able to manage Docker. `sudo usermod -aG docker pi` is used in case you use the default **pi** user.
  3. To start the Docker service type `sudo systemctl enable docker` after doing this start the Docker daemon using `sudo systemctl start docker`. When you have done this you should be able to run the command `docker info`.

### Swarm mode (native Docker cluster)
Swarm is the native clustering technology for Docker. Running Docker in Swarm mode enables you to deploy and manage containers on a bunch of node's. One thing to keep in mind when using Docker Swarm is that containers aren't statefull, this means that if a node fail and the containers running on that node are restarted on another node the state isn't transfered so data that isn't saved isn't available anymore. A solution to this is develop applications that are stateless and make redundant systems that are cluster aware and sync their information constantly (a database application you can use for this is crate.io). Below you can see the different steps involved in setting up a Swarm.
  1. To initiate a Swarm use the command `docker swarm init --advertise-addr 10.0.0.1` the advertise address should be changed to the address on which you can access your RPi and want to connect to your Swarm. The node on which you run this command will be a Manager node in your Swarm. After doing this command a token and the commands to add worker nodes will be displayed. My output looked like this:
  ```
docker swarm join \
--token SWMTKN-1-2t3lg51f5n48g9p8xmbtg70fpirmtmvvt3nc38d6fsr0p9brd8-9d2f42stdw0h9eafak02hoknn \
10.0.0.11:2377

  ```
  2. If you want to add worker nodes you need to use to commands you could see in the output of the previous step. You need to run this on every node you want to add as a worker node.
  3. If you would like to add manager nodes to your Swarm you should run docker swarm join-token manager on the node where you initialy started your Swarm. This command will output the command and token needed to add manager nodes to your swarm.

## Using Docker & Swarm
In this chapter you can read information on how to use Docker and Docker Swarm.
### Commands
You can read a bunch of Docker commands and what they do below.
`docker info` -> display information over the Docker node you're working on<br />
`docker node ls` -> display a list of the nodes in your Docker Swarm<br />
`docker service ls` -> display a list of the services currently running in your Swarm<br />
`docker service ps ID/NAME` -> display information about the service defined by its name or id<br />
`docker ps` -> display the services running on the node you're working on<br />
`docker service create` -> create a service (in order to do this you will also need to use options of this command)<br />
`docker serivce update ID/NAME` -> update options of a running service (for example starting more replicas or replacing the running with a newer image)
### Running a service
In this topic we will start a simple http service displaying a web page.
To start the service type `docker service create --name hypriot-httpd -p 8080:80 --replicas 2 hypriot/rpi-busybox-httpd`. In this command the `--name` option is used to set the name of the service, the `-p` option is used to forward trafic to a specific port to the containers and the `--replicas` option will set how many replicas are running. I use the hypriot/rpi-busybox-httpd image to deploy my containers as this is a simple httpd busybox that will show a simple page to test the cluster. You can test this by surfing to your clusters advertise address on port 8080, in my case this would be `10.0.0.11:8080`. You can use the `docker service ps hypriot-rpi` command to display information about the service.
If you want to scale the service you can use `docker service update hypriot-httpd --replicas=4` to start 2 extra containers for a total of 4 running containers.
### HA on Swarm (High Availability)
We will use the service created in the previous step to give an example of the HA features of Swarm. To start of ensure the service is running correctly using `docker service ps hypriot-rpi` you should have 4 containers running. Now disconnect one of the RPi's in your cluster from your swarm (don't disconnect the manager you are working on for your own convenience). If you rerun `docker service ps hypriot-rpi` you should see that after a little time the container (or containers) that ran on the node you disconnected restarted on a different node.

# HA with keepalived on RPi

## Add repository to list
To be able to install keepalived on RPi you will need to add a repository in your sources list. To do this follow these steps:
  1. Add a key for the server, do this with: `sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0x908332071dd2`
	2. Make a file and open it to write: `sudo nano /etc/apt/sources.list.d/best-hosting.list`
	3. Paste the following in the file: `deb http://deb.best-hosting.cz/debian/ raspberry main`

## Install keepalived
Follow these steps to install keepalived:
  1. `sudo apt-get update`
  2. `sudo apt-get install bh-keepalived` we install this version instead of the normal one because the normal one doesn't work on ARM.
  3. Now use `rm -rf /etc/sysconfig/keepalived` to remove this file.
  4. Make a directory in place of the file from the previous step: `mkdir /etc/sysconfig/keepalived`.
  5. Use: `nano /etc/sysconfig/keepalived/keepalived.sysconfig` to add the config to the file:
  ```
# Options for keepalived. See `keepalived --help' output and keepalived(8) and
# keepalived.conf(5) man pages for a list of all options. Here are the most
# common ones :
#
# --vrrp               -P    Only run with VRRP subsystem.
# --check              -C    Only run with Health-checker subsystem.
# --dont-release-vrrp  -V    Dont remove VRRP VIPs & VROUTEs on daemon stop.
# --dont-release-ipvs  -I    Dont remove IPVS topology on daemon stop.
# --dump-conf          -d    Dump the configuration data.
# --log-detail         -D    Detailed log messages.
# --log-facility       -S    0-7 Set local syslog facility (default=LOG_DAEMON)
#
KEEPALIVED_OPTIONS="-D"
  ```
  6. After this you need to execute: `wget http://deb.best-hosting.cz/_scripts_/keepalived/init.d/keepalived -O /etc/init.d/keepalived`
  7. Make this file executable: `chmod +x /etc/init.d/keepalived`
  8. And then use `update-rc.d keepalived defaults`
  9. Now you need to start keepalived, do this with: `/etc/init.d/keepalived start`
  10. In order to have a working system you need to edit the configuration of keepalived. You can edit the file with `sudo nano /etc/keepalived/keepalived.conf`. You can find the configurations I used at the next chapter.
  11. After editing the file restart keepalived with: `/etc/init.d/keepalived restart`

## Configuration of keepalived.
Here you can find the config file's I used.
### Node 1 - master
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.20
    }
}
```
### Node 2 - backup
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.20
    }
}
```
### Node 3 - backup
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 99
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.20
    }
}
```
