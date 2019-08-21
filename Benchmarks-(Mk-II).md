The following benchmarks aim to provide an approximate reference for USB armory
CPU performance.

Please keep in mind that all benchmarks are affected by a multitude of factors
including, and not limited to, compilation options, kernel options and
configuration, running processes, etc. These results provide only an
approximate reference and should be taken with a grain of salt.

All tests have been performed under Arch Linux ARM.

## USB armory Mk II variants

The USB armory Mk II CPU is an ARM Cortex-A7, with an approximate efficiency
rating of 1.9 DMIPS/MHz.

The i.MX6ULZ CPU (P/N MCIMX6Z0DVM09AB) clocks at 900 MHz.

The variant (available for custom orders, see [features
comparison](https://github.com/inversepath/usbarmory/wiki/Hardware-security-features-(Mk-II))),
using the i.MX6UL (P/N MCIMX6G3DVM05AB) clocks at 528 MHz.

## nbench

The [nbench](https://github.com/santoshsk007/nbench) utility is compiled with the following gcc flags:

```
-s -static -Wall -O3 -mfpu=neon-vfpv4 -mfloat-abi=hard -mcpu=cortex-a7 -mtune=cortex-a7 -fomit-frame-pointer -marm -funroll-loops -ffast-math
```

| Device           | Memory Index  | Integer Index | Floating-Point Index |
|:-----------------|--------------:|--------------:|---------------------:|
| Mk II - 900 MHz  |         4.665 |         6.383 |                5.987 |
| Mk II - 528 MHz  |         2.660 |         3.755 |                3.578 |

## OpenSSL

The standard openssl speed test (`-evp <algorithm> -elapsed`) for the USB armory performed with the following OpenSSL version:
```
OpenSSL 1.1.1c  28 May 2019
platform: linux-armv4
compiler: gcc -fPIC -pthread -Wa,--noexecstack -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wa,--noexecstack -D_FORTIFY_SOURCE=2 -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -DOPENSSL_USE_NODELETE -DOPENSSL_PIC -DOPENSSL_CPUID_OBJ -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DKECCAK1600_ASM -DAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPOLY1305_ASM -DNDEBUG -D_FORTIFY_SOURCE=2
```

| Device          | Algorithm   | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:----------------|:------------|----------:|----------:|----------:|-----------:|-----------:|
| Mk II - 900 MHz | aes-128-cbc | 17018.06k | 21100.07k | 22626.22k |  23031.13k |  23109.63k |
| Mk II - 900 MHz | aes-256-cbc | 13572.35k | 16064.06k | 16906.33k |  17120.26k |  17164.97k |
| Mk II - 900 MHz | md5         |  5651.23k | 18548.25k | 49282.13k |  82440.87k | 102883.33k |
| Mk II - 900 MHz | sha1        |  4611.13k | 14033.11k | 32441.09k |  48374.10k |  56279.04k |
| Mk II - 900 MHz | sha256      |  3359.35k |  9276.48k | 19726.68k |  27468.12k |  31001.26k |
| Mk II - 900 MHz | sha512      |  2238.45k |  8984.17k | 15719.85k |  23549.61k |  27473.24k |
| Mk II - 528 MHz | aes-128-cbc |  9999.55k | 12403.37k | 13272.66k |  13509.29k |  13579.61k |
| Mk II - 528 MHz | aes-256-cbc |  7984.23k |  9445.63k |  9920.34k |  10048.17k |  10084.35k |
| Mk II - 528 MHz | md5         |  3384.74k | 11023.49k | 29140.48k |  48677.21k |  60547.07k |
| Mk II - 528 MHz | sha1        |  2791.46k |  8385.94k | 19295.15k |  28568.23k |  33226.75k |
| Mk II - 528 MHz | sha256      |  2149.78k |  5787.99k | 11973.80k |  16328.70k |  18262.70k |
| Mk II - 528 MHz | sha512      |  1332.70k |  5363.24k |  9300.82k |  13867.35k |  16198.31k |
