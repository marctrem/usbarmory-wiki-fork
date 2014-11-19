U-Boot + Linux + Debian
=======================

Assumptions
-----------

- Debian 7 (Wheezy) host
- microSD block device: /dev/sdb
- microSD file system mounting point: /mnt


Toolchain: Linaro 4.9
---------------------

```
$ wget http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz
$ tar xvf gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz -C ~
```


Preparing a microSD with Debian 7 (Wheezy)
------------------------------------------

- Host dependencies:

  Kernel support: binfmt_misc  
  Debian packages: parted, debootstrap, binfmt-support, qemu-user-static, uboot-mkimage


```
$ sudo parted /dev/sdb --script mklabel msdos
$ sudo parted /dev/sdb --script mkpart primary ext4 5M 100%
$ sudo mkfs.ext4 /dev/sdb1
$ sudo mount /dev/sdb1 /mnt
$ sudo qemu-debootstrap --arch=armhf --include=ssh wheezy /mnt ftp://ftp.debian.org/debian/
$ sudo wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/debian_conf/inittab -O /mnt/etc/inittab
$ sudo chroot /mnt
(chroot)# echo -e '#!/bin/sh -e\nmodprobe g_ether\n/sbin/ifconfig usb0 10.0.0.1\nroute add -net default gw 10.0.0.2\nexit 0' > /etc/rc.local
(chroot)# echo "usbarmory" > /etc/hostname
(chroot)# :> /etc/resolv.conf
(chroot)# passwd
(chroot)# exit
```

Kernel: Linux 3.16.2
--------------------

```
$ wget ftp://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.16.2.tar.gz
$ tar xvf linux-3.16.2.tar.gz
$ cd linux-3.16.2
$ wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/usbarmory_linux-3.16.2.config -O .config
$ make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- uImage LOADADDR=0x70008000
$ make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- dtbs
$ make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- modules
$ sudo cp arch/arm/boot/uImage /mnt/boot/
$ sudo cp arch/arm/boot/dts/imx53-qsb.dtb /mnt/boot/imx53-usbarmory.dtb
$ sudo make ARCH=arm INSTALL_MOD_PATH=/mnt modules_install
$ sudo umount /mnt
```

Bootloader: U-Boot 2014.07
--------------------------

```
$ git clone https://github.com/inversepath/u-boot-usbarmory.git
$ cd u-boot-usbarmory
$ make distclean
$ make usbarmory_config
$ make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
$ sudo dd if=u-boot.imx of=/dev/sdb bs=512 seek=2
```
