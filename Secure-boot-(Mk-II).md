### Security information

The USB armory Mk II is produced with up to date hardware and firmware to
address all known security issues, nonetheless the following security
information is highlighted for anyone that wants to apply these instructions on
other hardware based on i.MX6UL P/Ns.

To address the [HABv4 security advisory](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_QBVR2017-0001.txt),
the secure boot architecture is meant to work on i.MX6UL/i.MX6ULL/i.MX6ULZ
parts with Silicon Revision 1.2 or greater, implemented on Part Numbers (P/N)
with revision "AB" or greater.

To address the [U-Boot security advisory](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_IPVR2018-0001.txt),
always ensure that all listed mitigations are implemented.

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

* i.MX6UL secure boot configuration, which permanently fuses a hash of four
  concatenated CA public keys in the USB armory SoC fuse box, so that only a
  signed bootloader can ever be executed.

* U-Boot Verified Boot configuration, which embeds a public key in the
  bootloader so that only a signed kernel can ever be executed.

The combination of i.MX6UL secure boot and U-Boot verified boot features allows
a fully verified chain of trust, authenticating the executed Linux kernel.
When signing a kernel that embeds a root file system, such as the
[Embedded INTERLOCK distribution](https://github.com/inversepath/usbarmory/tree/master/software/buildroot/README-INTERLOCK-mark-two.md),
the authentication has full (boot, not runtime) coverage, otherwise Linux kernel verification of
executed code is not covered in this guide and left out to implementors.

The following instructions apply identically to variants i.MX6ULL and i.MX6ULZ.

### Prerequisites

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
[Makefile](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/hab-pki/Makefile-pki)
is available to provide reference example for certificate creation and can be
used as follows:

```
# adjust the USBARMORY_GIT, KEYS_* variables according to your environment and preferences
make -C ${USBARMORY_GIT}/software/secure_boot/hab-pki -f Makefile-pki KEYS_PATH=sb_keys KEY_LENGTH=2048 KEY_EXPIRY=3650 usbarmory_sb_keys
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

### Prepare U-Boot with Verified Boot and HAB support

Download and extract U-Boot sources:

```
wget ftp://ftp.denx.de/pub/u-boot/u-boot-2019.07.tar.bz2
tar xvf u-boot-2019.07.tar.bz2 && cd u-boot-2019.07
```

Apply the following patches for USB armory Mk II support within U-Boot:

* [0001-ARM-mx6-add-support-for-USB-armory-Mk-II-board.patch](https://github.com/inversepath/usbarmory/tree/master/software/u-boot/0001-ARM-mx6-add-support-for-USB-armory-Mk-II-board.patch)

* [0001-Drop-linker-generated-array-creation-when-CONFIG_CMD.patch](https://github.com/inversepath/usbarmory/tree/master/software/u-boot/0001-Drop-linker-generated-array-creation-when-CONFIG_CMD.patch)

The following commands are meant to be issued within the U-Boot source
directory.

The default boot media is the external micro SD card, if you wish to compile a
bootloader for the internal eMMC card the following changes are required:

```
# disable CONFIG_SYS_BOOT_DEV_MICROSD
# enable  CONFIG_SYS_BOOT_DEV_EMMC
sed -i -e 's/CONFIG_SYS_BOOT_DEV_MICROSD=y/# CONFIG_SYS_BOOT_DEV_MICROSD is not set/' configs/usbarmory-mark-two_defconfig
sed -i -e 's/# CONFIG_SYS_BOOT_DEV_EMMC is not set/CONFIG_SYS_BOOT_DEV_EMMC=y/' configs/usbarmory-mark-two_defconfig
```

The bootloader boot mode must be changed to enable Verified Boot, in doing so,
to perform the step later described in _Verifying HAB (pre-activation)_, it is
a good idea to leave the bootloader command line available before creating
production images:

```
# disable CONFIG_SYS_BOOT_MODE_NORMAL
# enable  CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN
sed -i -e 's/CONFIG_SYS_BOOT_MODE_NORMAL=y/# CONFIG_SYS_BOOT_MODE_NORMAL is not set/' configs/usbarmory-mark-two_defconfig
sed -i -e 's/# CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN is not set/CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN=y/' configs/usbarmory-mark-two_defconfig
```

After successful activation, see _Activate HAB_, remember to re-compile the
bootloader with the command line disabled, changing the boot mode as follows
and repeating the subsequent steps:

```
# disable CONFIG_SYS_BOOT_MODE_NORMAL, CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN
# enable  CONFIG_SYS_BOOT_MODE_VERIFIED_LOCKED
sed -i -e 's/CONFIG_SYS_BOOT_MODE_NORMAL=y/# CONFIG_SYS_BOOT_MODE_NORMAL is not set/' configs/usbarmory-mark-two_defconfig
sed -i -e 's/CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN=y/# CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN is not set/' configs/usbarmory-mark-two_defconfig
sed -i -e 's/# CONFIG_SYS_BOOT_MODE_VERIFIED_LOCKED is not set/CONFIG_SYS_BOOT_MODE_VERIFIED_LOCKED=y/' configs/usbarmory-mark-two_defconfig
```

The U-Boot compilation requires a precompiled zImage Linux kernel image source
tree path, if using the
[Embedded INTERLOCK distribution](https://github.com/inversepath/usbarmory/tree/master/software/buildroot/README-INTERLOCK-mark-two.md),
the path is under buildroot `output/build/linux-<version>` directory.

Apply the configuration and compile the required tools:

```
# adjust the KERNEL_SRC variable according to your environment
export KERNEL_SRC=$KERNEL_PATH
export CROSS_COMPILE=arm-none-eabi- # set to your arm toolchain prefix
make distclean
make usbarmory-mark-two_config
make tools CONFIG_MKIMAGE_DTC_PATH="scripts/dtc/dtc"
make dtbs ARCH=arm CONFIG_MKIMAGE_DTC_PATH="scripts/dtc/dtc"
```

Prepare image tree blob (itb) file according to the image tree source (its)
template in the repository:

```
# adjust the USBARMORY_GIT variable according to your environment
tools/mkimage -D "-I dts -O dtb -p 2000 -i $KERNEL_SRC" -f ${USBARMORY_GIT}/software/secure_boot/mark-two/usbarmory.its usbarmory.itb
```

Sign the itb file:

```
tools/mkimage -D "-I dts -O dtb -p 2000" -F -k ${KEYS_PATH} -K arch/arm/dts/imx6ull-usbarmory.dtb -r usbarmory.itb
```

Now the U-Boot image can be compiled, with inclusion of the embedded public
key:

```
make ARCH=arm
```

The compilation results in the two following files:

* u-boot-dtb.imx: bootloader image to be signed and flashed on the target
  microSD or eMMC (instead of u-boot.imx), as shown in the next sections.

* usbarmory.itb: image tree blob file containing the kernel, to be copied under
  `/boot` on the target microSD or eMMC (replaces zImage/uImage).

### Sign the bootloader

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

Concatenate the bootloader and the CSF file to create the signed bootloader.

```
cat u-boot-dtb.imx csf.bin > u-boot-signed.imx
```

### Install the bootloader and kernel image

The signed bootloader `u-boot-signed.imx` and image tree blob `usbarmory.itb` can
now be installed as shown in the last two sections of the documentation for the device
[buildroot profile](https://github.com/inversepath/usbarmory/blob/master/software/buildroot/README-INTERLOCK-mark-two.md).

The `u-boot-signed.imx` file replaces `u-boot.imx` in all procedures,
similarily `usbarmory.itb` replaces `zImage`.

### Install fusing tool

The One-Time-Programmable (OTP) fuses are stored in the SoC fuse array which is
accessed via the On-Chip OTP Controller (OCOTP_CTRL). See Table 5-9 of the
[i.MX6UL Reference Manual](https://www.nxp.com/docs/en/reference-manual/IMX6ULRM.pdf)
for details.

The [crucible](https://github.com/inversepath/crucible) tool provides user
space support for reading, and writing, OTP fuses. It is used in all following
sections to derive register bit map illustrations and implement OTP fuses read
and write commands.

The crucible tool is meant to be executed on the USB armory itself on a running
Linux instance with the `nvmem-imx-ocotp` kernel module loaded.

### Fuse the SRK table hash

The SRK hash itself is located in registers ranging from OCOTP_SRK0 to OCOTP_SRK7:

```
 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00  OCOTP_SRK0
┏━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┓ Bank:3 Word:0
┃SRK_HASH                                                                                       ┃
┗━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━┛
...
 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00  OCOTP_SRK7
┏━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┓ Bank:3 Word:7
┃SRK_HASH                                                                                       ┃
┗━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━┛

```

The SRK hash generated earlier can be fused as follows (WARNING: this is just
an example, your hash will differ and should be used in the following commands
instead):

```
# write fuse
crucible -m IMX6UL -r 1 -b 16 -e little blow SRK_HASH aabbccddeeffaabbccddeeffaabbccddeeffaabbccddeeffaabbccddeeffaabb

# verify fuse
crucible -s -m IMX6UL -r 1 -b 16 -e little read SRK_HASH
```

The fused SRK hash must be locked to prevent bits set to 0 to be fused to 1
later on, this is accomplished in register OCOTP_LOCK with fuse SRK_LOCK:

```
 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00  OCOTP_LOCK
┏━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┓ Bank:0 Word:0
┃GP6_L┃GP8_L┃GP7_L┃PI┃GP┃GP┃MI┃RO┃OT┃ANALO┃OT┃SW┃GP┃SR┃GP2_L┃GP1_L┃MAC_A┃  ┃SJ┃MEM_T┃BOOT_┃TESTE┃
┗━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━┛
                                                    14 ─────────────────────────────────────────  SRK_LOCK
```

The SRK lock fuse can be fused as follows:

```
# write fuse
crucible -m IMX6UL -r 1 -b 2 -e big blow SRK_LOCK 1

# verify fuse, output should be 1
crucible -s -m IMX6UL -r 1 -b 2 read SRK_LOCK
```
### Verifying HAB (pre-activation)

After rebooting the USB armory it can be verified, from the U-Boot shell and
only when configuration setting `CONFIG_SYS_BOOT_MODE_VERIFIED_OPEN` is set,
that no HAB events are generated. If no errors are present then the bootloader
image was correctly authenticated:

```
=> hab_status

Secure boot disabled

HAB Configuration: 0xf0, HAB State: 0x66
No HAB Events Found!
```

**IMPORTANT**: in case of errors activation must be avoided to prevent any
irreversible lock out from the device. Always ensure that this verification
step passes before following the activation steps of the next section.

### Activate HAB

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
enables secure boot.

In order to enable secure boot the SoC must be placed in Closed Security
Configuration, additionally every debugging aid that might allow its bypass or
execute arbitrary code must be disabled.

The following fuses, within registers `OCOTP_CFG5` and `OCOTP_CFG6`, are in scope of such
procedure:

```
 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00  OCOTP_CFG5
┏━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┓ Bank:0 Word:6
┃SD_PW┃PW┃TZ┃JT┃KT┃  ┃DL┃JTAG_┃WD┃SJ┃  ┃SD┃SD┃FO┃DDR3_CONFIG            ┃  ┃  ┃FO┃BT┃DI┃  ┃SEC_C┃ R: 0x00000018
┗━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━┛ W: 0x00000018
             27 ────────────────────────────────────────────────────────────────────────────────  JTAG_HEO
                26 ─────────────────────────────────────────────────────────────────────────────  KTE
                         23 22 ─────────────────────────────────────────────────────────────────  JTAG_SMODE
                                  20 ───────────────────────────────────────────────────────────  SJC_DISABLE
                                        18 ─────────────────────────────────────────────────────  SDP_READ_DISABLE
                                                                                     03 ────────  DIR_BT_DIS
                                                                                           01 00  SEC_CONFIG

 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00  OCOTP_CFG6
┏━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┳━━┓ Bank:0 Word:7
┃OV┃MMC_DLL_DLY         ┃  ┃LPB_B┃BT┃BOOT_FAILUR┃SD┃EM┃OV┃US┃EN┃BO┃US┃US┃DL┃SD┃SD┃UA┃DI┃L1┃BT┃OV┃ R: 0x0000001c
┗━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━╋━━┻━━┻━━┻━━┻━━┻━━┻━━┻━━┛ W: 0x0000001c
                                                                                  04 ───────────  UART_SERIAL_DOWNLOAD_DISABLE
```

The relevant fuses are described below, along with their fusing commands:

```
# set device in Closed Configuration (IMX6ULRM Table 8-2, p245)
crucible -m IMX6UL -r 1 -b 2 -e big blow SEC_CONFIG 0b11

# disable NXP reserved mode (IMX6ULRM 8.2.6, p244)
crucible -m IMX6UL -r 1 -b 2 -e big blow DIR_BT_DIS 1

# Disable debugging features (IMX6ULRM Table 5-9, p216)
# * disable Secure JTAG controller
# * disable JTAG debug mode
# * disable HAB ability to enable JTAG
# * disable tracing
crucible -m IMX6UL -r 1 -b 2 -e big blow SJC_DISABLE 1
crucible -m IMX6UL -r 1 -b 2 -e big blow JTAG_SMODE 0b11
crucible -m IMX6UL -r 1 -b 2 -e big blow JTAG_HEO 1
crucible -m IMX6UL -r 1 -b 2 -e big blow KTE 1

# To further reduce the attack surface:
#  * disable Serial Download Protocol (SDP) READ_REGISTER command (IMX6ULRM 8.9.3, p310)
#  * disable SDP over UART (IMX6ULRM 8.9, p305)
crucible -m IMX6UL -r 1 -b 2 -e big blow SDP_READ_DISABLE 1
crucible -m IMX6UL -r 1 -b 2 -e big blow UART_SERIAL_DOWNLOAD_DISABLE 1
```

The USB armory will now refuse to run bootloader images not correctly signed
with keys corresponding to the fused hashes.

The security state log (see _Verify HAB configuration_) should now print a
`Trusted State detected` when the relevant cryptographic co-processor module is
loaded.

For security purposes, the U-Boot image should now be re-compiled with
`CONFIG_SYS_BOOT_MODE_VERIFIED_LOCKED` enabled, to ensure that command line
access is disabled.

### Verify HAB configuration (i.MX6ULL/i.MX6ULZ)

After rebooting the USB armory, the security state can be verified by checking
the [mxs-dcp driver](https://github.com/inversepath/mxs-dcp) log
message which should read:

```
mxs_dcp: Secure State detected
```

The `mxs_dcp` kernel module is compiled by default in the
[Embedded INTERLOCK distribution](https://github.com/inversepath/usbarmory/tree/master/software/buildroot/README-INTERLOCK-mark-two.md).

### Verify HAB configuration (i.MX6UL)

After rebooting the USB armory, the security state can be verified by checking
the [caam-keyblob driver](https://github.com/inversepath/caam-keyblob) log
message which should read:

```
caam_keyblob: Secure State detected
```

### External documentation

* [i.MX6 Secure Boot Application Note](http://cache.nxp.com/files/32bit/doc/app_note/AN4581.pdf)
