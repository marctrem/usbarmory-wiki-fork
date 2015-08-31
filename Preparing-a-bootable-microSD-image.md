Prerequisites
-------------

Recent Debian and Ubuntu (x86 or x86-64):
```
sudo apt-get install parted debootstrap binfmt-support qemu-user-static u-boot-tools wget git xz-utils tar
```

Only on x86-64 Debian add:
```
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install zlib1g-dev:i386 libstdc++6:i386
```

Only on x86-64 Ubuntu add:
```
sudo apt-get install ia32-libs
```

Toolchain
---------

On Debian 8 use the packaged one:
```
sudo apt-get install gcc-arm-none-eabi
export CROSS_COMPILE=arm-none-eabi-
```

On Ubuntu (12.04 LTS, 14.04 LTS, 14.10) use the packaged one:
```
sudo apt-get install gcc-arm-linux-gnueabihf
export CROSS_COMPILE=arm-linux-gnueabihf-
```

On Debian 7 install the Linaro 4.9:
```
export TOOLCHAIN_DIR=~          # set the directory where to install the toolchain
wget http://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz
tar xvf gcc-linaro-arm-none-eabi-4.9-2014.09_linux.tar.xz -C $TOOLCHAIN_DIR
export CROSS_COMPILE=${TOOLCHAIN_DIR}/gcc-linaro-arm-none-eabi-4.9-2014.09_linux/bin/arm-none-eabi-
```

Root file system
----------------

Prepare the microSD:
```
export TARGET_DEV=/dev/sdX     # pick the appropriate device name for your microSD card (e.g. /dev/sdb)
export TARGET_MNT=/mnt         # set the microSD root file system mounting path
sudo parted $TARGET_DEV --script mklabel msdos
sudo parted $TARGET_DEV --script mkpart primary ext4 5M 100%
sudo mkfs.ext4 ${TARGET_DEV}1
sudo mount ${TARGET_DEV}1 $TARGET_MNT
```

For Debian 8 (Jessie):
```
sudo qemu-debootstrap --arch=armhf --include=ssh,sudo,ntpdate,fake-hwclock,openssl,shellinabox,vim,nano,cryptsetup,lvm2,locales,less,cpufrequtils jessie $TARGET_MNT http://ftp.debian.org/debian/
sudo wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/debian_conf/rc.local -O ${TARGET_MNT}/etc/rc.local
sudo wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/debian_conf/sources.list -O ${TARGET_MNT}/etc/apt/sources.list
echo "tmpfs /tmp tmpfs defaults 0 0" | sudo tee ${TARGET_MNT}/etc/fstab
echo -e "\nUseDNS no" | sudo tee -a ${TARGET_MNT}/etc/ssh/sshd_config
echo "nameserver 8.8.8.8" | sudo tee ${TARGET_MNT}/etc/resolv.conf
sudo chroot $TARGET_MNT systemctl mask getty-static.service
```

For Ubuntu 14.10 (Utopic Unicorn):
```
wget http://cdimage.ubuntu.com/ubuntu-core/releases/14.10/release/ubuntu-core-14.10-core-armhf.tar.gz
sudo tar xvf ubuntu-core-14.10-core-armhf.tar.gz -C $TARGET_MNT
sudo wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/ubuntu_conf/ttymxc0.conf -O ${TARGET_MNT}/etc/init/ttymxc0.conf
sudo cp /usr/bin/qemu-arm-static ${TARGET_MNT}/usr/bin/qemu-arm-static
echo "nameserver 8.8.8.8" | sudo tee ${TARGET_MNT}/etc/resolv.conf
sudo chroot $TARGET_MNT apt-get install -y openssh-server whois fake-hwclock
```

Finalize and set the password:
```
echo "ledtrig_heartbeat" | sudo tee -a ${TARGET_MNT}/etc/modules
echo "ci_hdrc_imx" | sudo tee -a ${TARGET_MNT}/etc/modules
echo "g_ether" | sudo tee -a ${TARGET_MNT}/etc/modules
echo "options g_ether use_eem=0 dev_addr=1a:55:89:a2:69:41 host_addr=1a:55:89:a2:69:42" | sudo tee -a ${TARGET_MNT}/etc/modprobe.d/usbarmory.conf
echo -e 'allow-hotplug usb0\niface usb0 inet static\n  address 10.0.0.1\n  netmask 255.255.255.0\n  gateway 10.0.0.2'| sudo tee -a ${TARGET_MNT}/etc/network/interfaces
echo "usbarmory" | sudo tee ${TARGET_MNT}/etc/hostname
echo "usbarmory  ALL=(ALL) NOPASSWD: ALL" | sudo tee -a ${TARGET_MNT}/etc/sudoers
echo -e "127.0.1.1\tusbarmory" | sudo tee -a ${TARGET_MNT}/etc/hosts
sudo chroot $TARGET_MNT /usr/sbin/useradd -s /bin/bash -p `mkpasswd -m sha-512 usbarmory` -m usbarmory
sudo rm ${TARGET_MNT}/usr/bin/qemu-arm-static
```

Kernel: Linux 4.2
-----------------

```
export ARCH=arm
wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.2.tar.xz
tar xvf linux-4.2.tar.xz && cd linux-4.2
wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/usbarmory_linux-4.2.config -O .config
wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/imx53-usbarmory-common.dtsi -O arch/arm/boot/dts/imx53-usbarmory-common.dtsi
wget https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/imx53-usbarmory.dts -O arch/arm/boot/dts/imx53-usbarmory.dts
make uImage LOADADDR=0x70008000 modules imx53-usbarmory.dtb
sudo cp arch/arm/boot/uImage ${TARGET_MNT}/boot/
sudo cp arch/arm/boot/dts/imx53-usbarmory.dtb ${TARGET_MNT}/boot/imx53-usbarmory.dtb
sudo make INSTALL_MOD_PATH=$TARGET_MNT ARCH=arm modules_install
sudo umount $TARGET_MNT
```

Bootloader: U-Boot 2015.07
--------------------------

```
wget http://ftp.denx.de/pub/u-boot/u-boot-2015.07.tar.bz2
tar xvf u-boot-2015.07.tar.bz2 && cd u-boot-2015.07
make distclean
make usbarmory_config
make ARCH=arm
sudo dd if=u-boot.imx of=$TARGET_DEV bs=512 seek=2 conv=fsync
```

Connecting
----------

After successful image preparation the CDC Ethernet section of [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication) can be followed for pointers on establishing SSH communication with the OpenSSH server started by default by the installation.
