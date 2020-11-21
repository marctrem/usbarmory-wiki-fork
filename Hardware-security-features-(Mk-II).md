# i.MX6UL/i.MX6ULZ System-on-Chip

The following table summarizes the available hardware security features,
depending on the possible USB armory Mk II [variants](https://github.com/f-secure-foundry/usbarmory/wiki/Models) for information on available variants.

The standard retail version opts for a faster (900 MHz) i.MX6ULZ SoC, compared
to the i.MX6UL (528 MHz), with the main trade-off of lack of OTF DRAM
encryption.

The i.MX6UL variant with additional security properties, allowing external RAM
encryption and a more complete internal cryptographic accelerator, is available
for custom/bulk orders.

| Name  | Use                                  | Variants            | Availability |
|-------|--------------------------------------|---------------------|--------------|
| HABv4 | Secure Boot                          | all                 | retail       |
| RNGB  | TRNG                                 | i.MX6ULZ (900 MHz)  | retail       |
| DCP   | Cryptographic acceleration           | i.MX6ULZ (900 MHz)  | retail       |
| CAAM  | Cryptographic acceleration, TRNG     | i.MX6UL  (528 MHz)  | custom order |
| SNVS  | Secure Non-Volatile Storage          | all                 | retail       |
| BEE   | On-the-fly external RAM encryption   | i.MX6UL  (528 MHz)  | custom order |
| TZ    | ARM® TrustZone®                      | all                 | retail       |
| ATECC | External cryptographic co-processor  | ATECC608A on rev. β | retail       |
| ATECC | External cryptographic co-processor  | ATECC608B on rev. γ | TBA          |
| A71CH | External cryptographic co-processor  | rev. β              | retail       |
| SE050 | External cryptographic co-processor  | rev. γ              | TBA          |
| RPMB  | Protected flash memory region        | all                 | retail       |

## High Assurance Boot (HABv4)

The HAB feature enables on-chip internal Boot ROM authentication of initial
bootloader (i.e. Secure Boot) with a digital signature, establishing the first
trust anchor for code authentication. See
[Secure Boot](https://github.com/f-secure-foundry/usbarmory/wiki/Secure-boot-(Mk-II)) for
more information and usage instructions.

## Random Number Generator (RNGB) - i.MX6ULZ

On boards mounting the i.MX6ULZ SoC option the TRNG functionality is
provided by the dedicated RNGB block.

The RNGB driver is included and operational in modern Linux kernels, once
loaded it enables the component within Linux hw_random framework (see
`/dev/hwrng` and `rng-tools`).

## Data Co-Processor (DCP) - i.MX6ULZ

From the i.MX6ULZ datasheet: "This module provides support for general
encryption and hashing functions typically used for security functions."

The DCP module driver is included and operational in modern Linux kernels, once
loaded it exposes its algorithms through the Crypto API interface (see
`/proc/crypto`).

## Cryptographic accelerator and assurance module (CAAM) - i.MX6UL

From the i.MX6UL datasheet: "CAAM is a cryptographic accelerator and assurance
module. CAAM implements several encryption and hashing functions, a run-time
integrity checker, and a Pseudo Random Number Generator (PRNG)...CAAM also
implements a Secure Memory mechanism."

The CAAM accelerator driver is included and operational in modern Linux
kernels, once loaded it exposes its algorithms through the Crypto API interface
(see `/proc/crypto`).

## Secure Non-Volatile Storage (SNVS)

From the i.MX6UL datasheet: "Secure Non-Volatile Storage, including Secure Real
Time Clock, Security State Machine, Master Key Control, and Violation/Tamper
Detection and reporting."

A device specific random 256-bit OTPMK key is fused in each SoC at
manufacturing time, this key is unreadable and can only be used by the DCP
(i.MX6ULZ) or CAAM (i.MX6UL) for AES encryption/decryption of user data, through
the Secure Non-Volatile Storage (SNVS) companion block.

A Linux kernel driver for the DCP (i.MX6ULZ), which takes advantage of the
OTPMK released by the SNVS, is available at
[https://github.com/f-secure-foundry/mxs-dcp](https://github.com/f-secure-foundry/mxs-dcp).

A Linux kernel driver for the CAAM (i.MX6UL), which takes advantage of the
OTPMK released by the SNVS, is available at
[https://github.com/f-secure-foundry/caam-keyblob](https://github.com/f-secure-foundry/caam-keyblob).

## Bus Encryption Engine (BEE) - i.MX6UL

The BEE is included only in boards mounting the i.MX6UL SoC, it supports
on-the-fly (OTF) AES-128 (ECB or CTR) encryption/decryption on the AXI bus,
allowing OTF DRAM encryption.

## ARM® TrustZone®

The i.MX6 SoC family features an [ARM® TrustZone®](http://www.arm.com/products/processors/technologies/trustzone/)
implementation in its CPU core as well as its internal peripherals.

# External cryptographic co-processors

On β revisions the [Microchip ATECC608A](https://www.microchip.com/wwwproducts/en/ATECC608A) and
[NXP AT71CH](https://www.nxp.com/products/identification-and-security/authentication/plug-and-trust-the-fast-easy-way-to-deploy-secure-iot-connections:A71CH)
feature hardware acceleration for elliptic-curve cryptography as well as
providing hardware based key storage. The ATECC608A also features symmetric AES-128-GCM encryption.

On γ revisions (TBA) the [Microchip ATECC608B](https://www.microchip.com/wwwproducts/en/ATECC608A) and
[NXP SE050](https://www.nxp.com/products/security-and-authentication/authentication/edgelock-se050-plug-trust-secure-element-family-enhanced-iot-security-with-maximum-flexibility:SE050)
feature hardware acceleration for elliptic-curve and AES cryptography as well as providing
hardware based key storage.

All such external co-processors provide high-endurance monotonic counters, useful for external
verification of firmware downgrade/rollback attacks.

The external components communicate on the I²C bus and feature authenticated and
encrypted sessions for host communication.

# eMMC Replay Protected Memory Blocks (RPMB)

The eMMC RPMB features allows replay protected authenticated access to flash
memory partition areas, using a shared secret between the host and the eMMC.
