# Build-OLinuXino-LIME-image

This is how to create a minimal Debian image for the A20-OLINUXINO-LIME board (https://www.olimex.com/wiki/A20-OLinuXino-LIME). First install a fresh Debian (7) Wheezy (current stable) system (in VMware Workstation or Virtualbox etc) for building the image. 

## Setup environment for compiling

The manual process of setting up the build environment is documented below. 

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
cd ..
```

#### sunxi tools
```
git clone https://github.com/linux-sunxi/sunxi-tools.git
cd sunxi-tools/
make
```

```
wget https://raw.githubusercontent.com/linux-sunxi/sunxi-boards/master/sys_config/a10/a10-olinuxino-lime.fex
mv a10-olinuxino-lime.fex script.fex

./fex2bin script.fex ~/script.bin

ls -la ~/script.bin
-rw-r--r-- 1 user user 52624 Jan 22 12:03 /home/user/image/script.bin
cd ..
```

#### compile the kernel
```
git clone https://github.com/linux-sunxi/linux-sunxi
cd linux-sunxi
```
download config file https://drive.google.com/file/d/0B-bAEPML8fwlNzBNN1N2SGpmblk/edit?pli=1 and save as  ~/linux-sunxi/arch/arm/configs/a20_defconfig
```
make ARCH=arm a20_defconfig
make ARCH=arm menuconfig
```
edit the kernel config if you need to, when done
```
make -j4 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage CONFIG_DEBUG_SECTION_MISMATCH=y
```

#### Debian rootfs
Pre-install some packages do some system config of the new image.

######create the Debian system that will go on the sdcard
```
mkdir ~/rootfs

sudo debootstrap --arch=armhf --foreign wheezy /home/user/rootfs/

sudo cp /usr/bin/qemu-arm-static /home/user/rootfs/usr/bin/
sudo cp /etc/resolv.conf /home/user/rootfs/etc
```

######Chroot into the new system
```
sudo chroot /home/user/rootfs/

I have no name!@debian:/# uname -a
Linux debian 3.2.0-4-amd64 #1 SMP Debian 3.2.65-1+deb7u1 armv7l GNU/Linux

I have no name!@debian:/# export distro=wheezy
I have no name!@debian:/# export LANG=C

I have no name!@debian:/# /debootstrap/debootstrap --second-stage

I have no name!@debian:/# cat > /etc/apt/sources.list << __REPO__
deb http://http.debian.net/debian wheezy main
deb-src http://http.debian.net/debian wheezy main

deb http://http.debian.net/debian wheezy-updates main
deb-src http://http.debian.net/debian wheezy-updates main

deb http://security.debian.org/ wheezy/updates main
deb-src http://security.debian.org/ wheezy/updates main
__REPO__

I have no name!@debian:/# apt-get update
I have no name!@debian:/# apt-get upgrade

I have no name!@debian:/# apt-get install -f openssh-server ntpdate sudo \
iptables-persistent macchanger ifplugd \
locales dialog htop lsof strace python vim screen git \
hostapd hostap-utils wpasupplicant wireless-tools iw

I have no name!@debian:/# dpkg-reconfigure locales
I have no name!@debian:/# dpkg-reconfigure tzdata

I have no name!@debian:/# passwd
I have no name!@debian:/# adduser user

I have no name!@debian:/# echo debian > /etc/hostname

I have no name!@debian:/# echo 'g_ether' >> /etc/modules
I have no name!@debian:/# echo 'sunxi_emac' >> /etc/modules

I have no name!@debian:/# echo 'T0:2345:respawn:/sbin/getty -L ttyS0 115200 vt100' >> /etc/inittab
```

Configure the network:
```
cat > /etc/network/interfaces << __ETHCONF__
auto eth0
allow-hotplug eth0
iface eth0 inet dhcp

auto usb0
iface usb0 inet static
	address 192.168.0.2
	netmask 255.255.255.0
__ETHCONF__
```
Do any other config. Now is a good time to install and run Ansbile scripts.

Exit and Clean up the chroot:
```
I have no name!@debian:/# exit
sudo rm /home/user/rootfs/etc/resolv.conf
sudo rm /home/user/rootfs/usr/bin/qemu-arm-static
```

###### install wireless firmware
```
git clone http://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git

sudo cp -rfv /home/user/linux-sunxi/out/lib/modules/3.4.103+ /home/user/rootfs/lib/modules/
sudo cp -rfv /home/user/linux-sunxi/out/lib/firmware/ /home/user/rootfs/lib/
sudo cp -av /home/user/linux-firmware/rtlwifi/ /home/user/rootfs/lib/firmware/
sudo cp -av /home/user/linux-firmware/rt28* /home/user/rootfs/lib/firmware/
```

# Copy Image to SD card

####partition the card

####copy system

# Using
Place the SD card into the A20, hopefully it boots :-)

# Referances

* https://github.com/OLIMEX/OLINUXINO/tree/master/SOFTWARE/A20/A20-build
* http://linux-sunxi.org/Toolchain#Debian
* https://olimex.wordpress.com/2014/07/21/how-to-create-bare-minimum-debian-wheezy-rootfs-from-scratch/
* https://olimex.wordpress.com/2013/12/13/building-debian-linux-image-for-a10-olinuxino-lime-with-kernel-3-4-67/

