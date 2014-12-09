U-Boot + Linux + Debian
=======================

Assumptions
-----------

- Debian 7 (Wheezy) host

- Host dependencies:

  Kernel support: binfmt_misc  
  Debian packages: parted, debootstrap, binfmt-support, qemu-user-static, uboot-mkimage, wget

```
export TARGET_DEV=/dev/sdX     # pick the appropriate device name for your microSD card (e.g. /dev/sdb)
export TARGET_MNT=/mnt         # set the microSD root file system mounting path
```

Toolchain: Linaro 4.9
---------------------

```
wget http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz
tar xvf gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz -C ~
```

Root file system
----------------

Prepare the microSD:
```
sudo parted $TARGET_DEV --script mklabel msdos
sudo parted $TARGET_DEV --script mkpart primary ext4 5M 100%
sudo mkfs.ext4 ${TARGET_DEV}1
sudo mount ${TARGET_DEV}1 $TARGET_MNT
```

For Debian 7 (Wheezy):
```
sudo qemu-debootstrap --arch=armhf --include=ssh wheezy $TARGET_MNT http://ftp.debian.org/debian/
sudo wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/debian_conf/inittab -O ${TARGET_MNT}/etc/inittab
echo "deb http://ftp.debian.org/debian wheezy main" | sudo tee ${TARGET_MNT}/etc/apt/sources.list
echo "deb http://ftp.debian.org/debian wheezy-updates main" | sudo tee -a ${TARGET_MNT}/etc/apt/sources.list
echo "deb http://security.debian.org wheezy/updates main" | sudo tee -a ${TARGET_MNT}/etc/apt/sources.list
```

For Ubuntu 14.10 (Utopic Unicorn):
```
wget http://cdimage.ubuntu.com/ubuntu-core/releases/14.10/release/ubuntu-core-14.10-core-armhf.tar.gz
sudo tar xvf ubuntu-core-14.10-core-armhf.tar.gz -C $TARGET_MNT
sudo wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/ubuntu_conf/ttymxc0.conf -O ${TARGET_MNT}/etc/init/ttymxc0.conf
sudo cp /usr/bin/qemu-arm-static ${TARGET_MNT}/usr/bin/qemu-arm-static
```

Finalize and set the password:
```
echo -e '#!/bin/sh -e\nmodprobe g_ether\n/sbin/ifconfig usb0 10.0.0.1\nroute add -net default gw 10.0.0.2\nexit 0' | sudo tee ${TARGET_MNT}/etc/rc.local
echo "usbarmory" | sudo tee ${TARGET_MNT}/etc/hostname
echo "nameserver 8.8.8.8" | sudo tee ${TARGET_MNT}/etc/resolv.conf
sudo chroot $TARGET_MNT /usr/bin/passwd
sudo rm ${TARGET_MNT}/usr/bin/qemu-arm-static
```

Kernel: Linux 3.16.2
--------------------

```
wget http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.16.2.tar.gz
tar xvf linux-3.16.2.tar.gz
cd linux-3.16.2
wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/usbarmory_linux-3.16.2.config -O .config
make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- uImage LOADADDR=0x70008000
make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- dtbs
make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi- modules
sudo cp arch/arm/boot/uImage ${TARGET_MNT}/boot/
sudo cp arch/arm/boot/dts/imx53-qsb.dtb ${TARGET_MNT}/boot/imx53-usbarmory.dtb
sudo make ARCH=arm INSTALL_MOD_PATH=$TARGET_MNT modules_install
sudo umount $TARGET_MNT
```

Bootloader: U-Boot 2014.07
--------------------------

```
git clone https://github.com/inversepath/u-boot-usbarmory.git
cd u-boot-usbarmory
make distclean
make usbarmory_config
make ARCH=arm CROSS_COMPILE=~/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
sudo dd if=u-boot.imx of=$TARGET_DEV bs=512 seek=2
```

Connecting
----------

After successful image preparation the CDC Ethernet section of [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication) can be followed for pointers on establishing SSH communication with the OpenSSH server started by default by the installation.