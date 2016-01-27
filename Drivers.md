#### SAHARAv4 Lite (Cryptographic accelerator)

Quoting the datasheet "SAHARA (symmetric/asymmetric hashing and random accelerator), version 4, is a security coprocessor. It implements symmetric encryption algorithms, (AES, DES, 3DES, RC4 and C2), public key algorithms (RSA and ECC), hashing algorithms (MD5, SHA-1, SHA-224 and SHA-256), and a hardware true random number generator". 

The security co-processor driver is included in recent Linux kernels, however note that its support is broken since later versions in the 4.2 series due to a bug. The issue is described, along with a workaround patch, at http://permalink.gmane.org/gmane.linux.kernel.cryptoapi/18271