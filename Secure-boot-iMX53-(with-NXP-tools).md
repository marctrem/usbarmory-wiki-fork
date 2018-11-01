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

This document illustrates the procedure using the Code Signing Tool from NXP
(IMX_CST_TOOL), available for
[download](https://www.nxp.com/webapp/Download?colCode=IMX_CST_TOOL&appType=license&Parent_nodeId=1297866175545717803818&Parent_pageType=product)
(requires registration). An alternate method, using custom developed open
source tools, is described
[here](https://github.com/inversepath/usbarmory/wiki/Secure-boot-iMX53).

A working device tree compiler must be installed, on a recent Debian and Ubuntu
this can be done as follows:

```
sudo apt-get install device-tree-compiler
```

### Setting up the secure boot PKI infrastructure

Setup and create the secure boot key material as follows (changing the
passphrase with your own):

```
tar xvf cst-2.3.2.tar.gz
cd cst-2.3.2/keys
echo "00" > serial
# the tool requires the passphrase to be placed twice in the following file
echo "YOUR_CA_PASSPHRASE_CHANGEME" >  key_pass.txt
echo "YOUR_CA_PASSPHRASE_CHANGEME" >> key_pass.txt
./hab4_pki_tree.sh
```

The hab4_pki_tree.sh script prompts a few questions, here are typical answers:

```
Do you want to use an existing CA key (y/n)?: n
Do you want to use Elliptic Curve Cryptography (y/n)?: n
Enter key length in bits for PKI tree: 2048
Enter PKI tree duration (years): 10
How many Super Root Keys should be generated? 4
Do you want the SRK certificates to have the CA flag set? (y/n)?: y
```

The script generates four Super Root Keys (SRK), for each SRK a Command
Sequence File (CSF) key and an image signing key (IMG) are also generated.  The
CSF keys are used for signing the CSFs, while the IMG keys are used for signing
application data (e.g. U-Boot image).

The four SRKs must be merged in a table for SHA256 hash calculation:

```
cd ../crts # cst-2.3.2/crts
../linux64/srktool -h 4 -t SRK_1_2_3_4_table.bin -e SRK_1_2_3_4_fuse.bin -d sha256 \
  -c SRK1_sha256_2048_65537_v3_ca_crt.pem,SRK2_sha256_2048_65537_v3_ca_crt.pem,SRK3_sha256_2048_65537_v3_ca_crt.pem,SRK4_sha256_2048_65537_v3_ca_crt.pem
```

The SHA256 hash is created and can be inspected as follows (**WARNING**: this
is just an example, your hash will differ and should be used in the following
instructions instead):

```
hexdump -C SRK_1_2_3_4_fuse.bin
00000000  aa bb cc dd ee ff aa bb  cc dd ee ff aa bb cc dd  |................|
00000010  ee ff aa bb cc dd ee ff  aa bb cc dd ee ff aa bb  |................|
```

### Setting up the verified boot keys

A pair of RSA keys must be created for U-Boot verified boot:

```
# adjust the RSA_KEYS_PATH variable according to your environment
openssl genrsa -F4 -out ${RSA_KEYS_PATH}/usbarmory.key 2048
openssl req -batch -new -x509 -key ${RSA_KEYS_PATH}/usbarmory.key -out ${RSA_KEYS_PATH}/usbarmory.crt
```

### Prepare U-Boot (2018.01) with Verified Boot and HAB support

Download and extract U-Boot sources:

```
wget ftp://ftp.denx.de/pub/u-boot/u-boot-2018.01.tar.bz2
tar xvf u-boot-2018.01.tar.bz2 && cd u-boot-2018.01
```

Apply the following patch which enables i.MX53 High Assurance Boot (HAB)
support in U-Boot by adding the `hab_status` command, which helps verification
of secure boot state (optional but highly recommended).

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
tools/mkimage -D "-I dts -O dtb -p 2000" -F -k ${RSA_KEYS_PATH} -K pubkey.dtb -r usbarmory.itb
```

Now the U-Boot image can be compiled, it will include the embedded public key.
The image must be compiled in verbose mode to take note of the three
hexadecimal numbers present on the 'HAB Blocks:' line:

```
make ARCH=arm V=1 EXT_DTB=pubkey.dtb
```

The compilation results in the two following files:

* u-boot-dtb.imx: bootloader image to be signed and flashed on the target
  microSD card (instead of u-boot.imx), as shown in the next sections.

* usbarmory.itb: image tree blob file containing the kernel, to be copied under
  `/boot` on the target microSD card (replaces zImage/uImage).

### Prepare the CSF file

Download the example Command Sequence File:

* [hab4.csf](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/hab4.csf)

The file must be modified with the correct 'HAB Blocks' hex triple for the
u-boot.imx file compiled in the previous step, along with its path.

### Sign the U-Boot image

```
cd cst-2.3.2
linux64/cst -o csf.bin -i hab4.csf
```

### Prepare and flash the signed U-Boot

**IMPORTANT**: /dev/sdX must be replaced with your microSD device (not eventual
microSD partitions), ensure that you are specifying the correct one. Errors in
target specification will result in disk corruption.

```
objcopy -I binary -O binary --pad-to 0x2000 --gap-fill=0x00 csf.bin csf_pad.bin
cat u-boot-dtb.imx csf_pad.bin > u-boot-signed.imx
sudo dd if=u-boot-signed.imx of=/dev/sdX bs=512 seek=2 conv=fsync
```

**HINT**: It is now a good idea to verify if the resulting boot loader and kernel image
are working correctly. Only after you have done so proceed with the next steps
for secure boot activation.

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
with keys corresponding to the fused hashes:

```
=> hab_status

Secure boot enabled

HAB Configuration: 0xcc, HAB State: 0x99
No HAB Events Found!
```

### External documentation

* [i.MX53 Secure Boot Application Note](http://cache.nxp.com/files/32bit/doc/app_note/AN4581.pdf)
