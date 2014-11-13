U-Boot + Linux + Debian
=======================

Toolchain: Linaro 4.9
---------------------

- download:

  http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz

- install:

```
$ tar xvf gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz -C /usr/local
```

Bootloader: U-Boot 2014.07
--------------------------

- download:

```
$ git clone https://github.com/inversepath/u-boot-usbarmory.git
```

- build:

```
$ export ARCH=arm CROSS_COMPILE=/usr/local/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
$ make distclean
$ make armory_config
$ make
```

- result:

  u-boot.imx

Kernel: Linux 3.16.2
--------------------

- download:

  wget ftp://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.16.2.tar.gz

- build:

```
$ export ARCH=arm CROSS_COMPILE=/usr/local/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
$ cp linux-3.16.2.config .config
$ make uImage LOADADDR=0x70008000
$ make modules
$ make INSTALL_MOD_PATH=<modules_path> modules_install
```

- U-Boot configuration

```
> set bootargs console=ttymxc0,115200 root=/dev/mmcblk0p1 rootwait rw
> mmc read 0x70800000 0x1000 0xf000
> bootm 0x70800000a
```

- booting from first partition in mmc 0 (mmcblk0p1):

```
> ext2load mmc 0:1 0x70800000 uImage
> bootm 0x70800000
```

RootFS: Debian 7 (Wheezy)
-------------------------

- dependencies:

  kernel support: binfmt_misc  
  debianpackages: debootstrap binfmt-support qemu-user-static

- build:

```
$ qemu-debootstrap --arch=armhf wheezy armory-rootfs ftp://ftp.debian.org/debian/
$ chroot armory-rootfs
$ echo "usbarmory" > etc/hostname
$ :> etc/resolv.conf
$ passwd
```

Preparing the microSD
---------------------
```
$ dd if=u-boot.imx of=/dev/sdb bs=512 seek=2
$ cp -raP /rootfs/* /mnt/sd/
$ cp uImage /mnt/sd/boot
```
