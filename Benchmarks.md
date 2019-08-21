The following benchmarks aim to provide an approximate reference for USB armory
CPU performance. 

Please keep in mind that all benchmarks are affected by a multitude of factors
including, and not limited to, compilation options, kernel options and
configuration, running processes, etc. These results provide only an
approximate reference and should be taken with a grain of salt.

## USB armory Mk I variants

The USB armory Mk I i.MX53 CPU is an ARM Cortex-A8, with an approximate
efficiency rating of 2.0 DMIPS/MHz.

The i.MX53 CPU clocks by default at 800 MHz, depending on the mounted SoC model
the CPU can be optionally clocked at 1 GHz.

Production runs (batch number 0415) mounting the automotive version (i.MX534,
P/N MCIMX534AVV8C), which is officially advertised with a clock of 800 MHz,
have been positively tested running with a 1 GHz clock with no issues.

Production runs (batch numbers 1315, 4115, 0416, 3516) mounting the consumer
version (i.MX535, P/N MCIMX535DVV1C) are officially advertised with a clock of
1 GHz.

Production runs (batch numbers 4817, 1518) mounting the industrial version
(i.MX537, P/N MCIMX537CVV8C) are officially advertised with a clock of 800 MHz.

## nbench

The [nbench](https://github.com/santoshsk007/nbench) utility is compiled with the following gcc flags:
```
-s -static -Wall -O3 -mfpu=neon -mfloat-abi=hard -mcpu=cortex-a8 -mtune=cortex-a8 -fomit-frame-pointer -marm -funroll-loops -ffast-math
```

| Device          | Memory Index  | Integer Index | Floating-Point Index |
|:----------------|--------------:|--------------:|---------------------:|
| Mk I  - 800 MHz |         5.489 |         5.374 |                1.664 |

It should be noted that the USB armory ARM Cortex-A8 CPU includes the VFPLite floating point unit which is not as fast as other ARM FPUs.

## OpenSSL

The standard openssl speed test (-evp <algorithm> -elapsed) for the USB armory performed with the following OpenSSL version:
```
OpenSSL 1.1.1c  28 May 2019
platform: linux-armv4
compiler: gcc -fPIC -pthread -Wa,--noexecstack -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wa,--noexecstack -D_FORTIFY_SOURCE=2 -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -DOPENSSL_USE_NODELETE -DOPENSSL_PIC -DOPENSSL_CPUID_OBJ -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DKECCAK1600_ASM -DAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPOLY1305_ASM -DNDEBUG -D_FORTIFY_SOURCE=2
```

| Device         | Algorithm   | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:---------------|:------------|----------:|----------:|----------:|-----------:|-----------:|
| Mk I - 800 MHz | aes-128-cbc | 26857.25k | 32359.13k | 34405.72k |  34959.02k |  35121.83k |
| Mk I - 800 MHz | aes-256-cbc | 21590.80k | 25010.94k | 26915.88k |  26533.89k |  26604.89k |
| Mk I - 800 MHz | md5         |  6609.91k | 22986.75k | 58538.33k |  98356.57k | 121632.09k |
| Mk I - 800 MHz | sha1        |  6096.58k | 20896.64k | 55720.96k |  93835.95k | 117951.15k |
| Mk I - 800 MHz | sha256      |  5062.33k | 15670.25k | 35452.76k |  53976.06k |  63569.92k |
| Mk I - 800 MHz | sha512      |  2656.08k | 10449.17k | 19477.08k |  29248.17k |  34384.55k |