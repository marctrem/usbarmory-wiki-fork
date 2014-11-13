U-Boot + Linux + Debian
=======================

Assumptions
-----------

- Debian or Ubuntu host
- microSD block device: /dev/sdb
- microSD file system mounting point: /mnt
- ssh public key: id_dsa.pub


Toolchain: Linaro 4.9
---------------------

```
$ wget http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz
$ tar xvf gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz -C /usr/local
```


Preparing a microSD with Debian 7 (Wheezy)
------------------------------------------

- host dependencies:

  Kernel support: binfmt_misc  
  debian packages: parted, debootstrap, binfmt-support, qemu-user-static, uboot-mkimage


```
# parted /dev/sdb --script rm 1
# parted /dev/sdb --script mkpart primary ext4 5M 100%
# mkfs.ext4 /dev/sdb1
# mount /dev/sdb1 /mnt
# qemu-debootstrap --arch=armhf --include=ssh wheezy /mnt ftp://ftp.debian.org/debian/
# echo "usbarmory" > /mnt/etc/hostname
# :> /mnt/etc/resolv.conf
# wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/debian_conf/inittab -O /mnt/etc/inittab
# echo -e '#!/bin/sh -e\nmodprobe g_ether\n/sbin/ifconfig usb0 10.0.0.1\nroute add -net default gw 10.0.0.2\nexit 0' > /mnt/etc/rc.local
# chroot /mnt
# passwd
# exit
```

Kernel: Linux 3.16.2
--------------------

```
# wget ftp://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.16.2.tar.gz
# tar xvf linux-3.16.2.tar.gz
# cd linux-3.16.2
# export ARCH=arm CROSS_COMPILE=/usr/local/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
# wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/usbarmory_linux-3.16.2.config -O .config
# make uImage LOADADDR=0x70008000
# cp arch/arm/boot/uImage /mnt/boot/
# make dtbs
# cp arch/arm/boot/dts/imx53-qsb.dtb /mnt/boot/imx53-usbarmory.dtb
# make modules
# make INSTALL_MOD_PATH=/mnt modules_install
# umount /mnt
```

Bootloader: U-Boot 2014.07
--------------------------

```
# git clone https://github.com/inversepath/u-boot-usbarmory.git
# cd u-boot-usbarmory
# export ARCH=arm CROSS_COMPILE=/usr/local/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
# make distclean
# make usbarmory_config
# make
# dd if=u-boot.imx of=/dev/sdb bs=512 seek=2
```
