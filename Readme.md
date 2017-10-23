# Raspberry pi server setup

## Prerequisites

### Installation

Download the appropriate rasbian image

https://www.raspberrypi.org/downloads/raspbian/

Use the flash util to setup a unique hostname

```
git clone https://github.com/hypriot/flash
cd flash
./Linux/flash https://downloads.raspberrypi.org/raspbian_lite_latest
```

Note: an alternative would be to use dd

```
unzip 2017-06-21-raspbian-jessie-lite.zip
sudo dd if=./2017-06-21-raspbian-jessie-lite.img of=/dev/mmcblk0 bs=4M
```

## Getting started

use **proot** with qemu in order to setup.

```
sudo apt-get update && sudo apt-get install qemu-user proot
sudo mkdir -p /mnt/raspi
sudo mount /dev/sdx2 /mnt/raspi
sudo PROOT_NO_SECCOMP=1 proot -S /mnt/raspi -q qemu-arm --pwd=/home/pi
# now one should be able to play within a raspberry pi root file system
# happy hacking !!!
```

Hacking raspberry pi img

```
# get the second partition start offset.
sudo fdisk -lu /home/users/kellere5/Downloads/2017-06-21-raspbian-jessie-lite.img
#...
#Device                                                              Boot Start     End Sectors  Size Id Type
# /home/users/kellere5/Downloads/2017-06-21-raspbian-jessie-lite.img1       8192   93486   85295 41.7M  c W95 FAT32 (LBA)
# /home/users/kellere5/Downloads/2017-06-21-raspbian-jessie-lite.img2      94208 2548186 2453979  1.2G 83 Linux
#...
# mount the second partition to /mnt/raspi, the 94208 should differ from this example.
sudo mount -o loop,offset="$(( 512 * 94208))" /home/users/kellere5/Downloads/2017-06-21-raspbian-jessie-lite.img /mnt/raspi
```

## DHCP

https://www.raspberrypi.org/learning/networking-lessons/lesson-3/plan/

## DNS

https://www.raspberrypi.org/learning/networking-lessons/lesson-4/plan/

## Automation

The poor man script automation would look like:

```
cat << 'EOF' | sudo tee /mnt/raspi/var/local/configure.sh
#!/bin/sh
apt-get update
apt-get install -y dnsmasq
mv /etc/dnsmasq.conf /etc/dnsmasq.default
echo serverpi > /etc/hostname
sed '/$127.0.0.1.*pi@raspberrypi/d' /etc/hosts
echo "192.168.0.1 serverpi" > /etc/hosts.dnsmasq
cat << CONF > /etc/dnsmasq.conf
interface=eth0
dhcp-range=192.168.0.2,192.168.0.254,255.255.255.0,12h
dhcp-option=6,192.168.0.1
no-hosts
addn-hosts=/etc/hosts.dnsmasq
CONF

cat << INTERFACE > /etc/network/interfaces.d/eth0
allow-hotplug eth0
auto eth0
iface eth0 inet static
address 192.168.0.1
netmask 255.255.255.0
INTERFACE

systemctl enable ssh
EOF
sudo chmod +x /mnt/raspi/var/local/configure.sh
sudo PROOT_NO_SECCOMP=1 proot -S /mnt/raspi -q qemu-arm --pwd=/ bash -x /var/local/configure.sh
```

FIXME with ansible or a docker container...

