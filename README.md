# Build-OLinuXino-LIME-image

This is how to create a minimal Debian image for the A20-OLINUXINO-LIME board.

## Setup environment for compiling

First install Debian (7) Wheezy (current stable) system (in VMware Workstation or Virtualbox etc) for building the image.

###### install packages needed for building
```
sudo apt-get update
sudo apt-get upgrade
sudo echo "deb http://www.emdebian.org/debian/ unstable main" >> /etc/apt/sources.list

sudo dpkg --add-architecture armhf
sudo apt-get update
sudo apt-get install emdebian-archive-keyring

sudo apt-get install build-essential git debootstrap u-boot-tools ncurses-dev uboot-mkimage build-essential git pkg-config libusb++ libusb-1.0-0-dev qemu-user-static debootstrap binfmt-support fusefat exfat-utils exfat-fuse dosfstools dosfstools-dbg vim

sudo apt-get install gcc-4.7-arm-linux-gnueabihf 
```

###### link to compiler
```
mkdir ~/bin
cd ~/bin
for i in /usr/bin/arm-linux-gnueabihf*-4.7 ; do j=${i##/usr/bin/}; ln -s $i ${j%%-4.7} ; done
PATH=~/bin:$PATH
echo "PATH=~/bin:$PATH" >> ~/.bashrc
cd ..
```

## Create Image

#### uboot
```
git clone -b sunxi https://github.com/linux-sunxi/u-boot-sunxi.git
cd u-boot-sunxi/

make A20-OLinuXino-Lime_config ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
```
You should create these 6 files:
```
ls u-boot.bin u-boot-sunxi-with-spl.bin spl/sunxi-spl.bin spl/sunxi-spl.bin  u-boot-sunxi-with-spl.bin u-boot.bin -la

-rw-r--r-- 1 user user  20480 Jan 22 11:54 spl/sunxi-spl.bin
-rw-r--r-- 1 user user  20480 Jan 22 11:54 spl/sunxi-spl.bin
-rw-r--r-- 1 user user 233972 Jan 22 11:54 u-boot.bin
-rw-r--r-- 1 user user 233972 Jan 22 11:54 u-boot.bin
-rw-r--r-- 1 user user 266804 Jan 22 11:54 u-boot-sunxi-with-spl.bin
-rw-r--r-- 1 user user 266804 Jan 22 11:54 u-boot-sunxi-with-spl.bin

cd..
```

#### sunxi tools
```
git clone https://github.com/linux-sunxi/sunxi-tools.git
cd sunxi-tools/
make
```

# Copy Image to SD card


# Using


# Referances

* https://github.com/OLIMEX/OLINUXINO/tree/master/SOFTWARE/A20/A20-build
* http://linux-sunxi.org/Toolchain#Debian
* https://olimex.wordpress.com/2014/07/21/how-to-create-bare-minimum-debian-wheezy-rootfs-from-scratch/
* https://olimex.wordpress.com/2013/12/13/building-debian-linux-image-for-a10-olinuxino-lime-with-kernel-3-4-67/

