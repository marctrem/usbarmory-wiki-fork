# i.MX6UL/i.MX6ULZ System-on-Chip

## High Assurance Boot (HABv4)

The HAB feature enables on-chip internal Boot ROM authentication of initial
bootloader (i.e. Secure Boot) with a digital signature, establishing the first
trust anchor for code authentication. See
[Secure Boot](https://github.com/inversepath/usbarmory/wiki/Secure-boot-(Mk-II)) for
more information and usage instructions.

## Cryptographic accelerator and assurance module (CAAM) - i.MX6UL

From the i.MX6UL datasheet: "CAAM is a cryptographic accelerator and assurance
module. CAAM implements several encryption and hashing functions, a run-time
integrity checker, and a Pseudo Random Number Generator (PRNG)...CAAM also
implements a Secure Memory mechanism."

The CAAM accelerator driver is included and operational in modern Linux
kernels, once loaded it exposes its algorithms through the Crypto API interface
(see `/proc/crypto`).

## Data Co-Processor (DCP) - i.MX6ULZ

On boards mounting the i.MX6ULZ SoC option the CAAM is replaced with the DCP
module, providing a subset of the CAAM features.

From the i.MX6ULZ datasheet: "This module provides support for general
encryption and hashing functions typically used for security functions."

The DCP module driver is included and operational in modern Linux kernels, once
loaded it exposes its algorithms through the Crypto API interface (see
`/proc/crypto`).

## Random Number Generator (RNGB) - i.MX6ULZ

On boards mounting the i.MX6ULZ SoC option the CAAM TRNG functionality is
replaced with a dedicated RNGB block, which incorporates a TRNG.

The RNGB driver is included and operational in modern Linux kernels, once
loaded it enables the component within Linux hw_random framework (see
`/dev/hwrng` and `rng-tools`).

## Secure Non-Volatile Storage (SNVS)

From the i.MX6UL datasheet: "Secure Non-Volatile Storage, including Secure Real
Time Clock, Security State Machine, Master Key Control, and Violation/Tamper
Detection and reporting."

A device specific random 256-bit OTPMK key is fused in each SoC at
manufacturing time, this key is unreadable and can only be used by the CAAM
(i.MX6UL) or DCP (i.MX6ULZ) for AES encryption/decryption of user data, through
the Secure Non-Volatile Storage (SNVS) companion block.

A Linux kernel driver for the CAAM (i.MX6UL), which takes advantage of the
OTPMK released by the SNVS, is available at
[https://github.com/inversepath/caam-keyblob](https://github.com/inversepath/caam-keyblob).

A Linux kernel driver for the DCP (i.MX6ULZ), which takes advantage of the
OTPMK released by the SNVS, is available at
[https://github.com/inversepath/mxs-dcp](https://github.com/inversepath/mxs-dcp).

## Bus Encryption Engine (BEE) - i.MX6UL

The BEE is included only in boards mounting the i.MX6UL SoC, it supports
on-the-fly (OTF) AES-128 (ECB or CTR) encryption/decryption on the AXI bus,
allowing OTF DRAM encryption.

## ARM® TrustZone®

The i.MX6 SoC family features an [ARM® TrustZone®](http://www.arm.com/products/processors/technologies/trustzone/)
implementation in its CPU core as well as its internal peripherals.

# External cryptographic co-processors (ATECC608A, A71CH)

The [Microchip ATECC608A](https://www.microchip.com/wwwproducts/en/ATECC608A) and
[AT71CH](https://www.nxp.com/products/identification-and-security/authentication/plug-and-trust-the-fast-easy-way-to-deploy-secure-iot-connections:A71CH)
feature hardware acceleration for elliptic-curve cryptography as well as
hardware based key storage. The ATECC608A also features symmetric AES-128-GCM encryption.

Both components provide high-endurance monotonic counters, useful for external
verification of firmware downgrade/rollback attacks.

Both components communicate on the I²C bus and feature authenticated and
encrypted sessions for host communication.

The USB armory Mk II PCB layout features landing areas for both P/Ns, either or
both will be assembled on board.

# eMMC Replay Protected Memory Blocks (RPMB)

The eMMC RPMB features allows replay protected authenticated access to flash
memory partition areas, using a shared secret between the host and the eMMC.
