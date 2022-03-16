## High Assurance Boot (HABv4)

**IMPORTANT**: this feature is currently deprecated, see the related [security advisory](https://github.com/usbarmory/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_QBVR2017-0001.txt).

The HAB feature enables on-chip [internal Boot ROM](https://github.com/usbarmory/usbarmory/wiki/Internal-Boot-ROM-(Mk-I)) authentication of initial bootloader (i.e. Secure Boot) with a digital signature, establishing the first trust anchor for code authentication. See [Secure Boot](https://github.com/usbarmory/usbarmory/wiki/Secure-boot-(iMX53)) for more information and usage instructions.

## Security Controller (SCCv2)

From the i.MX53 datasheet: "The security controller is a security assurance hardware module designed to safely hold sensitive data, such as encryption keys, digital right management (DRM) keys, passwords and biometrics reference data. The SCCv2 monitors the system’s alert signal to determine if the data paths to and from it are secure, that is, it cannot be accessed from outside of the defined security perimeter. If not, it erases all sensitive data on its internal RAM. The SCCv2 also features a key encryption module (KEM) that allows non-volatile (external memory) storage of any sensitive data that is temporarily not in use. The KEM utilizes a device-specific hidden secret key and a symmetric cryptographic algorithm to transform the sensitive data into encrypted data".

A Linux kernel driver for the SCCv2 is available at [https://github.com/usbarmory/mxc-scc2](https://github.com/usbarmory/mxc-scc2).

## Cryptographic accelerator (SAHARAv4 Lite)

From the i.MX53 datasheet: "SAHARA (symmetric/asymmetric hashing and random accelerator), version 4, is a security coprocessor. It implements symmetric encryption algorithms, (AES, DES, 3DES, RC4 and C2), public key algorithms (RSA and ECC), hashing algorithms (MD5, SHA-1, SHA-224 and SHA-256), and a hardware true random number generator". 

The security co-processor driver is included and operational in Linux kernel >= 4.6, once loaded its activation is indicated by the following kernel log message:
```
sahara 63ff8000.crypto: SAHARA version 4 initialized
```

Note that the driver is present also in earlier kernel versions, however its support was broken in version >= 4.2 and < 4.6 due to a bug. The issue is described, along with a workaround patch, at [http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271](http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271).

## ARM® TrustZone®

The i.MX53 SoC features an [ARM® TrustZone®](http://www.arm.com/products/processors/technologies/trustzone/) implementation in its CPU core as well as its internal peripherals. The [Genode OS Framework](https://github.com/usbarmory/usbarmory/wiki/Genode-OS) includes a reference implementation for the USB armory.
