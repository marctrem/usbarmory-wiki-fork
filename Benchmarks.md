The following benchmarks aim to provide an approximate reference for USB armory
CPU performance.

Please keep in mind that all benchmarks are affected by a multitude of factors
including, and not limited to, compilation options, kernel options and
configuration, running processes, etc. These results provide only an
approximate reference and should be taken with a grain of salt.

All tests have been performed under Arch Linux ARM.

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

| Device         | Memory Index  | Integer Index | Floating-Point Index |
|:---------------|--------------:|--------------:|---------------------:|
| Mk I -   1 GHz |         6.865 |         6.696 |                2.074 |
| Mk I - 800 MHz |         5.489 |         5.374 |                1.664 |

It should be noted that the USB armory ARM Cortex-A8 CPU includes the VFPLite floating point unit which is not as fast as other ARM FPUs.

## OpenSSL

The standard openssl speed test (`-evp <algorithm> -elapsed`) for the USB armory performed with the following OpenSSL version:
```
OpenSSL 1.1.1c  28 May 2019
platform: linux-armv4
compiler: gcc -fPIC -pthread -Wa,--noexecstack -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wa,--noexecstack -D_FORTIFY_SOURCE=2 -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector-strong -fno-plt -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -DOPENSSL_USE_NODELETE -DOPENSSL_PIC -DOPENSSL_CPUID_OBJ -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DKECCAK1600_ASM -DAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPOLY1305_ASM -DNDEBUG -D_FORTIFY_SOURCE=2
```

| Device         | Algorithm   | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:---------------|:------------|----------:|----------:|----------:|-----------:|-----------:|
| Mk I -   1 GHz | aes-128-cbc | 29687.22k | 39413.65k | 43330.99k |  44437.50k |  44763.82k |
| Mk I -   1 GHz | aes-256-cbc | 23856.62k | 29832.90k | 32042.33k |  32642.73k |  32814.42k |
| Mk I -   1 GHz | md5         |  8104.33k | 27627.54k | 76176.04k | 135892.65k | 176289.11k |
| Mk I -   1 GHz | sha1        |  7774.18k | 25597.29k | 68051.54k | 115800.06k | 145615.53k |
| Mk I -   1 GHz | sha256      |  6440.22k | 19502.38k | 44834.65k |  67462.83k |  79055.53k |
| Mk I -   1 GHz | sha512      |  3439.53k | 13391.85k | 24435.63k |  36465.32k |  42737.66k |
| Mk I - 800 MHz | aes-128-cbc | 23138.08k | 30913.64k | 34063.79k |  34768.21k |  34856.96k |
| Mk I - 800 MHz | aes-256-cbc | 18633.98k | 23418.56k | 25074.69k |  25591.81k |  25665.54k |
| Mk I - 800 MHz | md5         |  6900.10k | 23245.16k | 62323.29k | 108426.58k | 138171.73k |
| Mk I - 800 MHz | sha1        |  6582.23k | 21377.34k | 55330.73k |  92574.38k | 114890.07k |
| Mk I - 800 MHz | sha256      |  5297.29k | 15914.39k | 35794.94k |  53226.50k |  61794.99k |
| Mk I - 800 MHz | sha512      |  2528.83k | 10322.69k | 18838.95k |  28400.64k |  33314.13k |
