### Disclaimer

**IMPORTANT**: enabling Secure Boot functionality on the USB armory SoC, unlike
similar features on modern PCs, is an **irreversible** action that
**permanenty** fuses verification keys hashes on the device. This means that
any errors in the process or loss of the signing PKI will result in a
**bricked** device uncapable of executing unsigned code. This is a security
feature, not a bug.

The activation and use of the Secure Boot functionality is therefore **at your
own risk** and must be approached with care.

### Prerequisites

At this time the Secure Boot functionality requires usage of the Code Signing
Tool from Freescale (IMX_CST_TOOL), available for [download](http://www.freescale.com/products/arm-processors/i.mx-applications-processors-based-on-arm-cores/i.mx-software-and-tools/i.mx-design-tools:IMX_DESIGN)
(requires registration).

**NOTE**: The last published IMX_CST_TOOL version 2.3.1 does not generate
correct signed images for the i.MX53 SoC, however version 2.2 correctly
supports the i.MX53. We are working with Freescale to have this addressed
and/or receive clear instructions for downloading the older version. In the
meantime we encourage users to write directly to Freescale support to request
IMX_CST_TOOL version 2.2.

### Setting up the PKI infrastructure

Setup and create the key material as follows (changing the passphrase with your
own):

```
tar xvf cst-2.2.tar.gz
cd cst-2.2/keys
echo "00" > serial
# the tool requires the passphrase to be placed twice in the following file
echo "YOUR_CA_PASSPHRASE_CHANGEME" >  key_pass.txt
echo "YOUR_CA_PASSPHRASE_CHANGEME" >> key_pass.txt
./hab4_pki_tree.sh
```

The hab4_pki_tree.sh script prompts a few questions, here are typical answers:

```
Do you want to use an existing CA key (y/n)?: n
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
cd ../crts # cst-2.2/crts
../linux/srktool -h 4 -t SRK_1_2_3_4_table.bin -e SRK_1_2_3_4_fuse.bin -d sha256 \
  -c SRK1_sha256_2048_65537_v3_ca_crt.pem,SRK2_sha256_2048_65537_v3_ca_crt.pem,SRK3_sha256_2048_65537_v3_ca_crt.pem,SRK4_sha256_2048_65537_v3_ca_crt.pem
```

The SHA256 hash is created and can be inspected as follows (**warning**: this
is just an example, your hash will differ and should be used in the following
instructions instead):

```
hexdump -C SRK_1_2_3_4_fuse.bin
00000000  aa bb cc dd ee ff aa bb  cc dd ee ff aa bb cc dd  |................|
00000010  ee ff aa bb cc dd ee ff  aa bb cc dd ee ff aa bb  |................|
```

### Prepare U-Boot with HAB support

The following step shows how to enable i.MX53 High Assurance Boot (HAB) support
in U-Boot. The patches implement support for the 'hab_status' command, which
helps in verifying the secure boot status.

You can follow the [U-Boot compilation
instructions](https://github.com/inversepath/usbarmory/wiki/Preparing-a-bootable-microSD-image#bootloader-u-boot-201507)
but with application of the following patches before compilation:

* [0001-imx-move-HAB-code-to-imx-general-directories.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot_patches/0001-imx-move-HAB-code-to-imx-general-directories.patch)
* [0002-ARM-mx53-add-support-for-HAB-commands.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot_patches/0002-ARM-mx53-add-support-for-HAB-commands.patch)
* [0003-usbarmory-add-secure-boot-configuration-commands.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot_patches/0003-usbarmory-add-secure-boot-configuration-commands.patch)
* [0004-ARM-mx53-disables-hab_auth_img-command.patch](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/u-boot_patches/0004-ARM-mx53-disables-hab_auth_img-command.patch)

The image must be compiled in verbose mode to take note of the three hex
numbers present on the 'HAB Blocks:' line:

```
make ARCH=arm V=1
```

### Prepare the CSF file

Download the example Command Sequence File:

* [hab4.csf](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/secure_boot/hab4.csf)

The file must be modified with the correct hex triple for the u-boot.imx file
compiled in the previous step, along with its path.

### Sign the U-boot image

```
cd cst-2.2
./linux/cst -o csf.bin < hab4.csf
```

### Prepare and Flash the signed U-Boot

**IMPORTANT**: /dev/sdX must be replaced with your microSD device (not eventual
microSD partitions), ensure that you are specifying the correct one. Errors in
target specification will result in disk corruption.

```
objcopy -I binary -O binary --pad-to 0x2000 --gap-fill=0x00 csf.bin csf_pad.bin
cat u-boot.imx csf_pad.bin > u-boot-signed.imx
sudo dd if=u-boot-signed.imx of=/dev/sdX bs=512 seek=2 conv=fsync
```

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

See the Addendum to Rev. 2 of the [i.MX53 Reference Manual](http://cache.freescale.com/files/32bit/doc/ref_manual/iMX53RM.pdf)
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

Secure boot enabled

HAB Configuration: 0xf0, HAB State: 0x66
No HAB Events Found!
```

### Secure boot activation

Only if you are confident that you can correctly generate signed U-Boot images,
the SoC can be placed in Closed Security Configuration.

**WARNING**: enabling Secure Boot functionality on the USB armory SoC, unlike
similar features on modern PCs, is an **irreversible** action that
**permanenty** fuses verification keys hashes on the device. This means that
any errors in the process or loss of the signing PKI will result in a
**bricked** device uncapable of executing unsigned code. This is a security
feature, not a bug.

The activation and use of the Secure Boot functionality is therefore **at your
own risk**, the following command permanently locks the fused configuration and
enables Secure Boot:

```
=> fuse prog -y 0 0x4 0x2
=> fuse prog -y 0 0x5 0x1
```

The USB armory will now refuse to run bootloader images not correctly signed
with keys corresponding to the fused hashes.

### Freescale documentation

* [i.MX53 Secure Boot Application Note](http://cache.freescale.com/files/32bit/doc/app_note/AN4581.pdf)
