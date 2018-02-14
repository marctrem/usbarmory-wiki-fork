### Security information

**IMPORTANT**: this feature is currently deprecated, see the related [security advisory](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_QBVR2017-0001.txt).

### Disclaimer

**IMPORTANT**: enabling secure boot functionality on the USB armory SoC, unlike
similar features on modern PCs, is an **irreversible** action that
**permanently** fuses verification keys hashes on the device. This means that
any errors in the process or loss of the signing PKI will result in a
**bricked** device incapable of executing unsigned code. This is a security
feature, not a bug.

The activation and use of the secure boot functionality is therefore **at your
own risk** and must be approached with care.

### Introduction

The following instructions jointly illustrate the following:

* i.MX53 secure boot configuration, which permanently fuses a hash of four
  concatenated CA public keys in the USB armory SoC fuse box, so that only a
  signed bootloader can ever be executed.

* U-Boot Verified Boot configuration, which embeds a public key in the
  bootloader so that only a signed kernel can ever be executed.

The combination of i.MX53 secure boot and U-Boot verified boot features allows
a fully verified chain of trust, authenticating the executed Linux kernel.
When signing a kernel that embeds a root file system, such as the
[Embedded INTERLOCK distribution](https://github.com/inversepath/usbarmory/tree/master/software/buildroot/README-INTERLOCK.md),
the authentication has full (boot, not runtime) coverage, otherwise Linux kernel verification of
executed code is not covered in this guide and left out to implementors.

### Prerequisites

This document illustrates the procedure using custom developed open source
tools, an alternate method, using NXP Code Signing Tool (IMX_CST_TOOL), is
available [here](https://github.com/inversepath/usbarmory/wiki/Secure-boot-(with-NXP-tools)).

A working device tree compiler and make must be installed, on a recent Debian
and Ubuntu this can be done as follows:

```
sudo apt-get install device-tree-compiler make
```

Additionally a modern Ruby interpreter is required, with the following gems
installed: bit-struct, digest, getoptlong, openssl.

Required gems can be typically installed as follows:

```
gem install <name>
```

### Setting up the secure boot PKI infrastructure

The PKI infrastructure, following NXP conventions, requires the following 
certificates:

* Four Super Root Key (SRK) Certification Authorities, the USB armory SoC is
  fused with a SHA256 hash of their concatenated public keys.

* Command Sequence File (CSF) public/private key pair, used to sign CSFs.

* Image signing key (IMG) public/private key pair, used to sign application
  data (e.g. U-Boot image).

The key material can be created with your own existing CA, an helper
[Makefile](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/Makefile-pki)
is available to provide reference example for certificate creation and can be
used as follows:

```
# adjust the USBARMORY_GIT, KEYS_* variables according to your environment and preferences
make -C ${USBARMORY_GIT}/software/secure_boot -f Makefile-pki KEYS_PATH=sb_keys KEY_LENGTH=2048 KEY_EXPIRY=3650 usbarmory_sb_keys
```

The four SRKs must be merged in a table for SHA256 hash calculation, the hash
is going to be eventually fused on the USB armory SoC. The table and hash can
be generated with the
[usbarmory_srktool](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/usbarmory_srktool)
as follows:

```
usbarmory_srktool  \
  --key1  ${KEYS_PATH}/SRK_1_crt.pem \
  --key2  ${KEYS_PATH}/SRK_2_crt.pem \
  --key3  ${KEYS_PATH}/SRK_3_crt.pem \
  --key4  ${KEYS_PATH}/SRK_4_crt.pem \
  --hash  ${KEYS_PATH}/SRK_1_2_3_4_fuse.bin \
  --table ${KEYS_PATH}/SRK_1_2_3_4_table.bin
```

The SHA256 hash is created and can be inspected as follows (**WARNING**: this
is just an example, your hash will differ and should be used in the following
instructions instead):

```
hexdump -C ${KEYS_PATH}/SRK_1_2_3_4_fuse.bin
00000000  aa bb cc dd ee ff aa bb  cc dd ee ff aa bb cc dd  |................|
00000010  ee ff aa bb cc dd ee ff  aa bb cc dd ee ff aa bb  |................|
```

### Setting up the verified boot keys

A pair of RSA keys must be created for U-Boot verified boot:

```
# adjust the KEYS_PATH variable according to your environment
openssl genrsa -F4 -out ${KEYS_PATH}/usbarmory.key 2048
openssl req -batch -new -x509 -key ${KEYS_PATH}/usbarmory.key -out ${KEYS_PATH}/usbarmory.crt
```

### Prepare U-Boot (2018.01) with Verified Boot and HAB support

Download and extract U-Boot sources:

```
wget ftp://ftp.denx.de/pub/u-boot/u-boot-2018.01.tar.bz2
tar xvf u-boot-2018.01.tar.bz2 && cd u-boot-2018.01
```

Apply the following patch which enables i.MX53 High Assurance Boot (HAB)
support in U-Boot by adding the `hab_status` command, which allows verification
of secure boot state.

* [0001-Add-HAB-support.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot-2018.01_patches/0001-Add-HAB-support.patch)

Apply the following patches to enable Verified Boot support, disable the U-Boot
command line and external environment variables to further lock down physical
serial console access.

* [0002-Add-verified-boot-support.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot-2018.01_patches/0002-Add-verified-boot-support.patch)
* [0003-Disable-CLI.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot-2018.01_patches/0003-Disable-CLI.patch)

The U-Boot compilation requires a precompiled zImage Linux kernel image source
tree path, if using the
[Embedded INTERLOCK distribution](https://github.com/inversepath/usbarmory/tree/master/software/buildroot/README-INTERLOCK.md)
the path is under buildroot `output/build/linux-<version>` directory.

The following commands are meant to be issued within the U-Boot source
directory:

```
# adjust the KERNEL_SRC variable according to your environment
export KERNEL_SRC=$KERNEL_PATH
export CROSS_COMPILE=arm-none-eabi- # set to your arm toolchain prefix
make distclean
make usbarmory_config
make tools
```

Compile the Flattened Device Tree file by leaving room for later public key
insertion:

```
# adjust the USBARMORY_GIT variable according to your environment
dtc -p 0x1000 ${USBARMORY_GIT}/software/secure_boot/pubkey.dts -O dtb -o pubkey.dtb
```

Prepare image tree blob (itb) file according to the image tree source (its)
template in the repository:

```
tools/mkimage -D "-I dts -O dtb -p 2000 -i $KERNEL_SRC" -f ${USBARMORY_GIT}/software/secure_boot/usbarmory.its usbarmory.itb
```

Sign the itb file:

```
tools/mkimage -D "-I dts -O dtb -p 2000" -F -k ${KEYS_PATH} -K pubkey.dtb -r usbarmory.itb
```

Now the U-Boot image can be compiled, with inclusion of the embedded public
key:

```
make ARCH=arm EXT_DTB=pubkey.dtb
```

The compilation results in the two following files:

* u-boot-dtb.imx: bootloader image to be signed and flashed on the target
  microSD card (instead of u-boot.imx), as shown in the next sections.

* usbarmory.itb: image tree blob file containing the kernel, to be copied under
  `/boot` on the target microSD card (replaces zImage/uImage).

### Prepare the CSF file

Download the
[usbarmory_csftool](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/usbarmory_csftool)
tool and prepare the Command Sequence File (the example chooses SRK keypair #1):

```
usbarmory_csftool \
  --csf_key ${KEYS_PATH}/CSF_1_key.pem \
  --csf_crt ${KEYS_PATH}/CSF_1_crt.pem \
  --img_key ${KEYS_PATH}/IMG_1_key.pem \
  --img_crt ${KEYS_PATH}/IMG_1_crt.pem \
  --table   ${KEYS_PATH}/SRK_1_2_3_4_table.bin \
  --index   1 \
  --image   u-boot-dtb.imx \
  --output  csf.bin
```

The resulting CSF binary contains the signature for the bootloader image and
the commands that instruct the SoC to verify it.

### Prepare and flash the signed U-Boot

**IMPORTANT**: /dev/sdX must be replaced with your microSD device (not eventual
microSD partitions), ensure that you are specifying the correct one. Errors in
target specification will result in disk corruption.

```
cat u-boot-dtb.imx csf.bin > u-boot-signed.imx
sudo dd if=u-boot-signed.imx of=/dev/sdX bs=512 seek=2 conv=fsync
```

**HINT**: It is now a good idea to verify if the resulting boot loader and
kernel image are working correctly. Only after you have done so proceed with
the next steps for secure boot activation.

### Fuse the SRK table hash

The hash is stored in the SoC Electrical Fuse Array (FUSEBOX) which is accessed
via the IC Identification Module (IIM):

| Fuse name         | IIM bank | IIM addr[bits] | Function                             |
|:-----------------:|:--------:|:--------------:|--------------------------------------|
| SRK_HASH[255:248] | 1        | 0x0c04         | SRK table hash (part 1)              |
| SRK_HASH[247:160] | 3        | 0x1404-0x142c  | SRK table hash (part 2)              |
| SRK_HASH[159:0]   | 3        | 0x1430-0x147c  | SRK table hash (part 3)              |
| SRK_LOCK          | 1        | 0x0c00[2]      | lock for SRK_HASH[255:248]           |
| SRK_LOCK88        | 3        | 0x1400[1]      | lock for SRK_HASH[247:160]           |
| SRK_LOCK160       | 3        | 0x1400[0]      | lock for SRK_HASH[159:0]             |
| SRK_REVOKE[2:0]   | 4        | 0x1810[2:0]    | SRK keys revocation                  |
| SEC_CONFIG[1:0]   | 0        | 0x0810[1:0]    | Security configuration               |
| DIR_BT_DIS[1:0]   | 0        | 0x0814[0]      | Direct external memory boot disable  |

See the Addendum to Rev. 2 of the [i.MX53 Reference Manual](http://cache.nxp.com/files/32bit/doc/ref_manual/iMX53RM.pdf)
for details (Chapter 2 - Fusemap).

The following commands (=> prompt) are meant to be executed on the USB armory,
within the u-boot bootloader, using the serial port accessible through the
breakout header (see [Using external
GPIOs](https://github.com/inversepath/usbarmory/wiki/GPIOs) for details).

In order to fuse anything, the VDD_FUSE power supply must be enabled:

```
=> i2c mw 0x34 0x33 0xf9
=> i2c mw 0x34 0x10 0x40
```

**IMPORTANT**: the following commands permanently fuse values in the SoC and are
**irreversible**, take extra care in ensuring that the right data is written.

The SHA256 hash generated earlier can be fused as follows (**WARNING**: this is
just an example, your hash will differ and should be used in the following
commands instead):

```
# syntax: fuse prog [-y] <bank> <word> <hexval> [<hexval>...] - program 1 or
#         several fuse words, starting at 'word' (PERMANENT)
=> fuse prog -y 1 0x1 0xaa
=> fuse prog -y 3 0x1 0xbb 0xcc 0xdd 0xee 0xff 0xaa 0xbb 0xcc 0xdd 0xee 0xff
=> fuse prog -y 3 0xc 0xaa 0xbb 0xcc 0xdd 0xee 0xff 0xaa 0xbb 0xcc 0xdd 0xee
=> fuse prog -y 3 0x17 0xff 0xaa 0xbb 0xcc 0xdd 0xee 0xff 0xaa 0xbb
```

The fused key can be read and verified:
```
# syntax: fuse read <bank> <word> [<cnt>] - read 1 or 'cnt' fuse words,
#         starting at 'word'
=> fuse read 1 0x1 1
=> fuse read 3 0x1 31
```

The fused hash must be locked to prevent bits set to 0 to be fused to 1
later on:

```
=> fuse prog -y 1 0x0 0x4  # SRK_LOCK
=> fuse prog -y 3 0x0 0x3  # SRK_LOCK88 + SRK_LOCK160
```

### Verifying HAB

After rebooting the USB armory it can be verified, from the U-Boot shell,
that no HAB events are generated. If no errors are present then the bootloader
image was correctly authenticated:

```
=> hab_status

Secure boot disabled

HAB Configuration: 0xf0, HAB State: 0x66
No HAB Events Found!
```

### Secure boot activation

Only if you are confident that you can correctly generate signed U-Boot images,
the SoC can be placed in Closed Security Configuration.

**WARNING**: enabling secure boot functionality on the USB armory SoC, unlike
similar features on modern PCs, is an **irreversible** action that
**permanently** fuses verification keys hashes on the device. This means that
any errors in the process or loss of the signing PKI will result in a
**bricked** device incapable of executing unsigned code. This is a security
feature, not a bug.

The activation and use of the secure boot functionality is therefore **at your
own risk**, the following command permanently locks the fused configuration and
enables secure boot (remember to enable VDD_FUSE power supply as shown earlier):

```
=> fuse prog -y 0 0x4 0x2
=> fuse prog -y 0 0x5 0x1
```

The USB armory will now refuse to run bootloader images not correctly signed
with keys corresponding to the fused hashes.

### External documentation

* [i.MX53 Secure Boot Application Note](http://cache.nxp.com/files/32bit/doc/app_note/AN4581.pdf)
