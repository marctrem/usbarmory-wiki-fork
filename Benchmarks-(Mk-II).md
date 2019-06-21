The following benchmarks aim to provide an approximate reference for USB armory
CPU performance. 

Please keep in mind that all benchmarks are affected by a multitude of factors
including, and not limited to, compilation options, kernel options and
configuration, running processes, etc. These results provide only an
approximate reference and should be taken with a grain of salt.

## USB armory Mk II variants

The USB armory Mk II i.MX6ULZ (P/N MCIMX6Z0DVM09AB) clocks at 900 MHz.

The variant (available for custom orders, see [features
comparison](https://github.com/inversepath/usbarmory/wiki/Hardware-security-features-(Mk-II))),
using the i.MX6UL (P/N MCIMX6G3DVM05AB) clocks at 528 MHz.

## nbench

The [nbench](http://www.tux.org/~mayer/linux/bmark.html) utility is compiled with the following gcc (Debian 4.6.3-14) flags:
```
# $cpu set to either cortex-a7 (mark-one) or cortex-a8 (mark-two)
-s -static -Wall -O3 -mfpu=neon -mfloat-abi=hard -mcpu=$cpu -mtune=$cpu -fomit-frame-pointer -marm -funroll-loops -ffast-math
```

| Device           | Memory Index  | Integer Index | Floating-Point Index |
|:-----------------|--------------:|--------------:|---------------------:|
| Mk II - 900 MHz  |         4.723 |         6.449 |                5.609 |
| Mk II - 528 MHz  |         2.763 |         3.780 |                3.306 |

## OpenSSL

The standard openssl speed test (-evp <algorithm> -elapsed) for the USB armory performed with the following OpenSSL version:
```
OpenSSL 1.1.1c  28 May 2019
platform: linux-armv4
compiler: gcc -fPIC -pthread -Wa,--noexecstack -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wa,--noexecstack -D_FORTIFY_SOURCE=2 -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -DOPENSSL_USE_NODELETE -DOPENSSL_PIC -DOPENSSL_CPUID_OBJ -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DKECCAK1600_ASM -DAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPOLY1305_ASM -DNDEBUG -D_FORTIFY_SOURCE=2
```

| Device          | Algorithm   | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:----------------|:------------|----------:|----------:|----------:|-----------:|-----------:|
| Mk II - 900 MHz | aes-128-cbc | 17054.68k | 21172.78k | 22657.37k |  23062.19k |  23205.21k |
| Mk II - 528 MHz | aes-128-cbc |  9999.55k | 12403.37k | 13272.66k |  13509.29k |  13579.61k |
| Mk II - 900 MHz | aes-256-cbc | 13615.14k | 16110.95k | 16932.52k |  17150.29k |  17216.85k |
| Mk II - 528 MHz | aes-256-cbc |  7984.23k |  9445.63k |  9920.34k |  10048.17k |  10084.35k |
| Mk II - 900 MHz | md5         |  5780.65k | 18823.51k | 49765.03k |  83087.36k | 103410.35k |
| Mk II - 528 MHz | md5         |  3384.74k | 11023.49k | 29140.48k |  48677.21k |  60547.07k |
| Mk II - 900 MHz | sha1        |  4748.28k | 14280.92k | 32903.34k |  48758.78k |  56726.87k |
| Mk II - 528 MHz | sha1        |  2791.46k |  8385.94k | 19295.15k |  28568.23k |  33226.75k |
| Mk II - 900 MHz | sha256      |  3683.56k |  9855.96k | 20408.15k |  27864.06k |  31178.75k |
| Mk II - 528 MHz | sha256      |  2149.78k |  5787.99k | 11973.80k |  16328.70k |  18262.70k |
| Mk II - 900 MHz | sha512      |  2289.54k |  9198.98k | 15903.57k |  23693.31k |  27658.92k |
| Mk II - 528 MHz | sha512      |  1332.70k |  5363.24k |  9300.82k |  13867.35k |  16198.31k |
