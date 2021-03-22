### Security information

The USB armory Mk II is produced with up to date hardware and firmware to
address all known security issues, nonetheless the following security
information is highlighted for anyone that wants to apply these instructions on
other hardware based on i.MX6UL P/Ns.

To address the [HABv4 security advisory](https://github.com/f-secure-foundry/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_QBVR2017-0001.txt),
the secure boot architecture is meant to work on i.MX6UL/i.MX6ULL/i.MX6ULZ
parts with Silicon Revision 1.2 or greater, implemented on Part Numbers (P/N)
with revision "AB" or greater.

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

The pre-requisites of an effective secure booted environment are the following:

* i.MX6UL secure boot configuration, which permanently fuses a hash of four
  concatenated CA public keys in the USB armory SoC fuse box, so that only a
  signed bootloader can ever be executed.

* A bootloader with an embedded public key to ensure authenticated
  configuration, to this end [armory-boot](https://github.com/f-secure-foundry/armory-boot)
  is used on the USB armory Mk II.

The combination of i.MX6UL secure boot and armory-boot authentication features
allow a fully verified chain of trust, to boot a trusted Linux kernel image.

When signing a [TamaGo unikernel](https://github.com/f-secure-foundry/tamago)
or a Linux kernel which embeds a root file system the authentication has full
boot (not runtime) coverage. The runtime Linux kernel verification of executed
code, if required, is out of scope of this guide.

The following instructions apply identically to variants i.MX6ULL and i.MX6ULZ.

### Prerequisites

A modern Ruby interpreter is required, with the following gems installed:
bit-struct, digest, getoptlong, openssl.

Required gems can be typically installed as follows:

```
gem install <name>
```

Either [signify](https://man.openbsd.org/signify) (sometimes packaged as
`signify-openbsd`) or [minisign](https://jedisct1.github.io/minisign/) are
required for authenticating bootloader configuration.

### Setting up the secure boot PKI infrastructure

The PKI infrastructure, following NXP conventions, requires the following 
certificates:

* Four Super Root Key (SRK) Certification Authorities, the USB armory SoC is
  fused with a SHA256 hash of their concatenated public keys.

* Command Sequence File (CSF) public/private key pair, used to sign CSFs.

* Image signing key (IMG) public/private key pair, used to sign application
  data (e.g. bootloader image).

The [habtool](https://github.com/f-secure-foundry/crucible/tree/master/cmd/habtool)
utility provides a reference for certificate creation (existing CAs or OpenSSL
can also be used, see this
[Makefile](https://github.com/f-secure-foundry/usbarmory/blob/master/software/secure_boot/hab-pki/Makefile-pki)
for an alternative method).

```
# adjust the HAB_KEYS variables according to your preferred path

habtool \
  -C ${HAB_KEYS}/SRK_1_crt.pem
  -c ${HAB_KEYS}/SRK_1_key.pem

habtool \
  -C ${HAB_KEYS}/SRK_2_crt.pem
  -c ${HAB_KEYS}/SRK_2_key.pem

habtool \
  -C ${HAB_KEYS}/SRK_3_crt.pem
  -c ${HAB_KEYS}/SRK_3_key.pem

habtool \
  -C ${HAB_KEYS}/SRK_4_crt.pem
  -c ${HAB_KEYS}/SRK_4_key.pem
```

The four SRKs must be merged in a table for SHA256 hash calculation, the hash
is going to be eventually fused on the USB armory SoC. The table and hash can
be generated with [habtool](https://github.com/f-secure-foundry/crucible/tree/master/cmd/habtool)
as follows:

```
habtool \
  -1 ${HAB_KEYS}/SRK_1_crt.pem \
  -2 ${HAB_KEYS}/SRK_2_crt.pem \
  -3 ${HAB_KEYS}/SRK_3_crt.pem \
  -4 ${HAB_KEYS}/SRK_4_crt.pem \
  -o ${HAB_KEYS}/SRK_1_2_3_4_fuse.bin \
  -t ${HAB_KEYS}/SRK_1_2_3_4_table.bin
```

The SHA256 hash is created and can be inspected as follows (**WARNING**: this
is just an example, your hash will differ and should be used in the following
instructions instead):

```
hexdump -C ${HAB_KEYS}/SRK_1_2_3_4_fuse.bin
00000000  aa bb cc dd ee ff aa bb  cc dd ee ff aa bb cc dd  |................|
00000010  ee ff aa bb cc dd ee ff  aa bb cc dd ee ff aa bb  |................|
```

### Compiling and installing the bootloader

[armory-boot](https://github.com/f-secure-foundry/armory-boot) is a minimal
primary boot loader which allows starting Linux kernel images with authenticated
configuration.

The signing procedure applies to any IMX executable, but it is shown against
the `armory-boot` bootloader in this document.

Signatures for IMX executable images can be generated with `habtool` as
follows:

```
habtool \
  -A ${HAB_KEYS}/CSF_1_key.pem \
  -a ${HAB_KEYS}/CSF_1_crt.pem \
  -B ${HAB_KEYS}/IMG_1_key.pem \
  -b ${HAB_KEYS}/IMG_1_crt.pem \
  -t ${HAB_KEYS}/SRK_1_2_3_4_table.bin \
  -x 1 \
  -i armory-boot.imx \
  -o armory-boot.csf && \
```

The signature is meant to be concatenated to the executable for producing the
final signed image:

```
cat armory-boot.imx armory-boot.csf > armory-boot-signed.imx
```

This procedure is automatically performed by the `imx_signed` Makefile target
for `armory-boot`, when compiled passing the keys created in the previous
section through the `HAB_KEYS` variable, creating the signed bootloader
image.

```
make CROSS_COMPILE=arm-none-eabi- imx_signed
```

Please refer to [armory-boot documentation](https://github.com/f-secure-foundry/armory-boot/blob/master/README.md)
for details on the bootloade compilation, installation and configuration.

Specifically the armory-boot [Secure Boot documentation](https://github.com/f-secure-foundry/armory-boot/blob/master/README.md#secure-boot)
illustrates how to maintain the chain of trust with authenticated bootloader
configuration, ensuring target Linux kernel authentication.

### Fusing the SRK table hash

The One-Time-Programmable (OTP) fuses are stored in the SoC fuse array which is
accessed via the On-Chip OTP Controller (`OCOTP_CTRL`). See Table 5-9 of the
[i.MX6UL Reference Manual](https://www.nxp.com/docs/en/reference-manual/IMX6ULRM.pdf)
for details.

The [crucible](https://github.com/f-secure-foundry/crucible/tree/master/cmd/crucible)
tool provides user space support for reading, and writing, OTP fuses. It is
used in all following sections to derive register bit map illustrations and
implement OTP fuses read and write commands.

The crucible tool is meant to be executed on the USB armory Mk II itself, on a
running Linux instance with the `nvmem-imx-ocotp` kernel module loaded.

The SRK hash itself is located in registers ranging from `OCOTP_SRK0` to `OCOTP_SRK7`:

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
later on, this is accomplished in register `OCOTP_LOCK` with fuse `SRK_LOCK`:

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

### Activating HAB

Only if you are confident that you can correctly generate signed bootloader
images, the SoC can be placed in Closed Security Configuration.

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

### Verifying HAB configuration (i.MX6ULL/i.MX6ULZ)

After rebooting the USB armory, the security state can be verified by checking
the [mxs-dcp driver](https://github.com/f-secure-foundry/mxs-dcp) log
message which should read:

```
mxs_dcp: Trusted State detected
```

The `mxs_dcp` kernel module is compiled by default in the USB armory Mk II
[Debian base image](https://github.com/f-secure-foundry/usbarmory-debian-base_image/releases).

### Verifying HAB configuration (i.MX6UL)

After rebooting the USB armory, the security state can be verified by checking
the [caam-keyblob driver](https://github.com/f-secure-foundry/caam-keyblob) log
message which should read:

```
caam_keyblob: Trusted State detected
```

### Revoking keys

Each of the first 3 Super Root Keys (SRK) (corresponding to index 1-3 passed to
`habtool`) can be revoked on a single unit, this is useful to prevent
vulnerable (e.g. because of known security issues) signed images from being
used again on a device.

To do so the `OCOTP_SRK_REVOKE` fuse can be used with the
[crucible](https://github.com/f-secure-foundry/crucible) tool to set
`SRK_REVOKE[0:2]` bits, reflecting the index of the revoked key.

Example of revocation for key with index 3 (corresponding
to `SRK_REVOKE` bit 2):

```
crucible -m IMX6UL -r 1 -b 2 -e big blow OCOTP_SRK_REVOKE 0b100
```

### External documentation

* [i.MX6 Secure Boot Application Note](http://cache.nxp.com/files/32bit/doc/app_note/AN4581.pdf)
