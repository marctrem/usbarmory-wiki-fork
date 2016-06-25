#### Advanced High Assurance Boot (A-HAB)

The HAB feature enables on-chip [internal Boot ROM](https://github.com/inversepath/usbarmory/wiki/Internal-Boot-ROM) authentication of initial bootloader (i.e. Secure Bot) with a digital signature, establishing the first trust anchor for code authentication. See [Secure Boot](https://github.com/inversepath/usbarmory/wiki/Secure-boot) for more information and usage instructions.

#### Security Controller (SCCv2)

The SCCv2 implements secure RAM and a dedicated AES cryptographic engine for encryption/decryption operations. A device specific random 256-bit SCC key is fused in each i.MX53 SoC at manufacturing time, this key is unreadable and can only be used with the SCCv2 for AES encryption/decryption of user data.

#### SAHARAv4 Lite (Cryptographic accelerator)

From the SoC datasheet: "SAHARA (symmetric/asymmetric hashing and random accelerator), version 4, is a security coprocessor. It implements symmetric encryption algorithms, (AES, DES, 3DES, RC4 and C2), public key algorithms (RSA and ECC), hashing algorithms (MD5, SHA-1, SHA-224 and SHA-256), and a hardware true random number generator". 

The security co-processor driver is included and operational in Linux kernel >= 4.6, once loaded its activation is indicated by the following kernel log message:
```
sahara 63ff8000.crypto: SAHARA version 4 initialized
```

Note that the driver is present also in earlier kernel versions, however its support was broken in version >= 4.2 and < 4.6 due to a bug. The issue is described, along with a workaround patch, at [http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271](http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271).

#### ARM TrustZone

The i.MX53 SoC features an [ARM® TrustZone®](http://www.arm.com/products/processors/technologies/trustzone/) implementation in its CPU core as well as its internal peripherals. The [Genode OS Framework](https://github.com/inversepath/usbarmory/wiki/Genode-OS) includes a reference implementation for the USB armory.