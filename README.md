# Build-OLinuXino-LIME-image

This is how to create a lightweight Debian image for the A20-OLINUXINO-LIME board.

## Setup Building environment

First install Debian (7) Wheezy (current stable) system (in VMware Workstation or Virtualbox etc) for building the image.

###### install packages for building

sudo apt-get update
sudo apt-get upgrade
sudo echo "deb http://www.emdebian.org/debian/ unstable main" >> /etc/apt/sources.list

sudo dpkg --add-architecture armhf
sudo apt-get update
sudo apt-get install emdebian-archive-keyring

sudo apt-get install build-essential git debootstrap u-boot-tools ncurses-dev uboot-mkimage build-essential git pkg-config libusb++ libusb-1.0-0-dev qemu-user-static debootstrap binfmt-support fusefat exfat-utils exfat-fuse dosfstools dosfstools-dbg vim

sudo apt-get install gcc-4.7-arm-linux-gnueabihf 

###### link to compiler

mkdir ~/bin
cd ~/bin
for i in /usr/bin/arm-linux-gnueabihf*-4.7 ; do j=${i##/usr/bin/}; ln -s $i ${j%%-4.7} ; done
PATH=~/bin:$PATH
echo "PATH=~/bin:$PATH" >> ~/.bashrc
cd ..


## Create Image


## Copy Image to SD card


