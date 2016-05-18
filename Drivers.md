#### SAHARAv4 Lite (Cryptographic accelerator)

Quoting the datasheet "SAHARA (symmetric/asymmetric hashing and random accelerator), version 4, is a security coprocessor. It implements symmetric encryption algorithms, (AES, DES, 3DES, RC4 and C2), public key algorithms (RSA and ECC), hashing algorithms (MD5, SHA-1, SHA-224 and SHA-256), and a hardware true random number generator". 

The security co-processor driver is included and operational in Linux kernel >= 4.6, once loaded its activation is indicated by the following kernel log message:
```
sahara 63ff8000.crypto: SAHARA version 4 initialized
```

Note that the driver is present also in earlier kernel versions, however its support was broken in version >= 4.2 and < 4.6 due to a bug. The issue is described, along with a workaround patch, at [http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271](http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271).