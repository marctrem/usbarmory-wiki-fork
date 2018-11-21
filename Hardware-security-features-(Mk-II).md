### i.MX6UL System-on-Chip

#### High Assurance Boot (HABv4)

*IMPORTANT*: HABv4 on the i.MX6UL requires Silicon Revision 1.2 and HAB 4.1 or
greater, implemented on Part Numbers (P/N) with revision "AB" or greater, as
they include patches for erratas ERR010873 and ERR010872, see the following
[security advisory](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_QBVR2017-0001.txt).

The HAB feature enables on-chip internal Boot ROM authentication of initial
bootloader (i.e. Secure Boot) with a digital signature, establishing the first
trust anchor for code authentication. See
[Secure Boot](https://github.com/inversepath/usbarmory/wiki/Secure-boot-(Mk-II)) for
more information and usage instructions.

#### Cryptographic accelerator and assurance module (CAAM)

From the i.MX6UL datasheet: "CAAM is a cryptographic accelerator and assurance
module. CAAM implements several encryption and hashing functions, a run-time
integrity checker, and a Pseudo Random Number Generator (PRNG)...CAAM also
implements a Secure Memory mechanism."

The CAAM accelerator driver is included and operational in modern Linux
kernels, once loaded its activation is indicated by the following kernel log
message:

```
caam algorithms registered in /proc/crypto
```

#### Secure Non-Volatile Storage (SNVS)

From the i.MX53 datasheet: "Secure Non-Volatile Storage, including Secure Real
Time Clock, Security State Machine, Master Key Control, and Violation/Tamper
Detection and reporting."

A device specific random 256-bit OTPMK key is fused in each SoC at
manufacturing time, this key is unreadable and can only be used by the CAAM for
AES encryption/decryption of user data, through the Secure Non-Volatile Storage
(SNVS) companion block.

A Linux kernel driver for the CAAM, which takes advantage of the OTPMK released
by the SNVS, is available at
[https://github.com/inversepath/caam-keyblob](https://github.com/inversepath/caam-keyblob).

#### Bus Encryption Engine (BEE)

The BEE supports on-the-fly (OTF) AES-128 (ECB or CTR) encryption/decryption on
the AXI bus, allowing OTF DRAM encryption.

#### ARM® TrustZone®

The i.MX6UL SoC features an [ARM® TrustZone®](http://www.arm.com/products/processors/technologies/trustzone/)
implementation in its CPU core as well as its internal peripherals.

### External cryptographic co-processor (ATECC608A)

The [Microchip ATECC608A](https://www.microchip.com/wwwproducts/en/ATECC608A)
features hardware acceleration for elliptic-curve cryptography as well as
hardware based key storage.

Additionally it can provide high-endurance monotonic counters, useful for
external verification of firmware downgrade/rollback attacks.

It is available on the I²C bus and features authenticated and encrypted
sessions for host communication.

### eMMC Replay Protected Memory Blocks (RPMB)

The eMMC RPMB features allows replay protected authenticated access to flash
memory partition areas, using a shared secret between the host and the eMMC.