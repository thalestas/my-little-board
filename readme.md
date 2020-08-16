# Linux Build and Boot Process
This repository describe how to compile bootloader, linux kernel and busybox to Raspberry Pi and BeagleBone.

## Toolchain
First, it's necessary to prepare the environment to perform the arm cross compilation. To do so, download Linaro arm toolchain (ex. ``i686_arm-linux-gnueabihf.tar.xz`` for 32bit machine or ``x86_64_arm-linux-gnueabihf.tar.xz`` for 64bit machine ) [here](https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/).

After download, export the path of the cross compilation toolchain in **.bashrc** including the following line in the end of the file.
```
export PATH=$PATH:<YOUR_EXTRACT_PATH>/gcc-linaro-<VERSION>_arm-linux-gnueabihf/bin
```
And run the command bellow to source the file.
```
source .bashrc
```
To test the cross compiler run the command bellow and check the version installed.
```
gcc --version
```
It is necessary to install the following dependencies too:
```
sudo apt-get install bison flex libssl-dev
```



## BeagleBone
- [Build](#beaglebone-builds) - Describe BeagleBone U-boot, Linux Kernel and BusyBox Builds.

## Raspberry Pi
- [Build](#raspberry-pi-builds) - Describe Raspbeery Pi U-boot, Linux Kernel and BusyBox Builds.

# BeagleBone Builds

## U-boot
1. Clone the source code of u-boot.
```
git clone https://github.com/u-boot/u-boot
```

# Raspberry PI Builds

## U-boot
First step that you need is get u-boot source code. You can get clone this [git](https://github.com/u-boot/u-boot).

After that, configure and compile u-boot to raspberry using respectively:

1. Raspberry Pi Zero W
```
make rpi_0_w_defconfig
```
2. Raspberry Pi 3
```
make rpi_3_32b_defconfig
```
So, compile using cross compile:
```
make CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
```

To boot kernel properly using u-boot, it is necessary create a file with boot arguments as following.
```
mmc dev 0
fatload mmc 0:1 ${kernel_addr_r} zImage
fatload mmc 0:1 0x2000000 bcm2835-rpi-zero-w.dtb
setenv bootargs 8250.nr_uarts=1 root=/dev/mmcblk0p2 rootwait console=ttyS0,115200
bootz ${kernel_addr_r} - 0x2000000
```
You can named as *boot.cmd*. But, u-boot only read *.scr* file. To create this file, run the following command to use a mkimage from u-boot.
```
tools/mkimage -C none -A arm -T script -d boot.cmd boot.scr
```

**Note** Even if you pass *bcm2835-rpi-zero-w.dtb* on bootloader, you need include the *bcm2708-rpi-zero-w.dtb* on boot partition. 
This device tree blob is used by Raspberry pi to configure uart. If you don't include, you not be able to see U-boot serial output

## Linux Kernel
The first thig that is necessary is clone this [git](https://github.com/raspberrypi/linux) to get Linux Kernel source code for Raspberry Pi.
```
git clone https://github.com/raspberrypi/linux
```

To initiate the compile process, it is a good idea to clean the previous configs if its exists. This can be done by the following command:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
```

Run the following command to create .config file. That file will set the build configurations.

Pi Zero W:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2835_defconfig
```
Then, to build use the following command:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs -j$(nproc)
```
The -j$(nproc) command will use all core of your computer to compile. If you would like to use less, change $(nproc) to the number of cores.

## How to prepare the SD Card
The following files are necessary to boot the Raspberry Pi. They are GPU firmware and bootloader. 
* bootcode.bin
* start.elf
* fixup.dat

You can download these file from [https://github.com/raspberrypi/firmware/tree/master/boot](https://github.com/raspberrypi/firmware/tree/master/boot)

```
wget https://github.com/raspberrypi/firmware/raw/master/boot/bootcode.bin
wget https://github.com/raspberrypi/firmware/raw/master/boot/start.elf
wget https://github.com/raspberrypi/firmware/raw/master/boot/fixup.dat
```

You also need *config.txt* file (Contents system configuration parameters) and *cmdline.txt* file (Arguments to Linux Kernel) to boot properly.

These files can be downloaded [here](https://github.com/thalestas/boot-and-compile-process/tree/master/raspberry). The *config.txt* provide only contains a configuration to enable UART but it is enough to work.
