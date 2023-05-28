# Embedded-Linux-for-Raspberry-Pi3b

In this repo, I present the minimal boot file system built from scratch for raspberry Pi 3b. 

steps

    1) build u-boot
    2) build kernel
    3) build rootfilesystem

    1) u-boot part

First of all need to install dependencies for u-boot

sudo apt -y install gcc-arm-linux-gnueabihf binutils-arm-linux-gnueabihf
sudo apt-get -y install bison flex bc libssl-dev make gcc 

then go through download source code

git clone https://github.com/u-boot/u-boot.git then got through directory

cd u-boot

go to configs file and check the version you have and apply the default for example in my case for raspberry pi3-b

make rpi_3_32b_defconfig ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

Adjust your settings through menuconfig

make menuconfig ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

Then run make to build the binary

make -j 6 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

    2) Kernel stage

git clone --depth=1 https://github.com/raspberrypi/linux
cd linux

Apply default config for RPi3

make bcm2709_defconfig ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
  
make menuconfig ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-


Compile actual kernel, modules, DTBs Takes ~45 minutes on modern i5 with SSD


make -j'proc' zImage modules dtbs ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

May require sudo:
creat a folder outside the kernel folder and remember the path

mkdir rootfs
export INSTALL_MOD_PATH=/home/hassan/rootfs
make modules_install ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

copy to sdacard


cp arch/arm/boot/zImage /media/hassan/boot
cp arch/arm/boot/dts/bcm2710-rpi-3-b.dtb /media/hassan/boot
cp arch/arm/boot/dts/overlays/disable-bt.dtbo /media/hassan/boot/overlays

    3) Busy Box

Download repo


git clone git://busybox.net/busybox.git --branch=1_33_0 --depth=1
cd BusyBox

Build


make menuconfig


Settings -> Build static binary 	(no shared libraries) 	Enable
Settings -> Cross compiler prefix 	arm-Linux-gnueabihf-
Settings -> Destination path for ‘make install’ 	Same as INSTALL_MOD_PATH from kernel modules ste

make -j12
make install

output

usr bin sbin lib

Adjust Rootfile system


# Create directories to mount stuff:
mkdir proc
mkdir sys
mkdir dev
mkdir etc

# Create config directory:
mkdir etc/init.d
touch etc/init.d/rcS
chmod +x etc/init.d/rcS

rcS


Add the following entries to etc/init.d/rcS:

#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys

echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s  # -s	Scan /sys and populate /dev\n"
