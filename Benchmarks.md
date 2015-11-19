The following benchmarks aim to provide an approximate reference for USB armory CPU performance. 

Please keep in mind that all benchmarks are affected by a multitude of factors including, and not limited to, compilation options, kernel options and configuration, running processes, etc. These results provide only an approximate reference and should be taken with a grain of salt.

The USB armory i.MX53 CPU clocks by default at 800 MHz, models mounting the consumer version (i.MX535) can be clocked at 1 GHz (MCIMX535DVV1).

The first production run (batch number 0415) mounts the automotive version (i.MX534, P/N MCIMX534AVV8C), which is officially advertised with a clock of 800 MHz, however it has been positively tested running with a 1 GHz clock with no issues.

The second (batch number 1315) and third (batch number 4115) production runs mount, with the exception of 9 boards, the consumer version (i.MX535, P/N MCIMX535DVV1C), which is officially advertised with a maximum clock of 1 GHz.

#### nbench

The [nbench](http://www.tux.org/~mayer/linux/bmark.html) utility is compiled with the following gcc (Debian 4.6.3-14) flags:
```
-s -static -Wall -O3 -mfpu=neon -mfloat-abi=hard -mcpu=cortex-a8 -mtune=cortex-a8 -fomit-frame-pointer -marm -funroll-loops -ffast-math
```

| Device             | Memory Index  | Integer Index | Floating-Point Index |
|:-------------------|--------------:|--------------:|---------------------:|
| USB armory   1 GHz |         7.005 |         6.401 |                1.751 |
| USB armory 800 MHz |         5.605 |         5.113 |                1.404 |

It should be noted that the USB armory ARM Cortex-A8 CPU includes the VFPLite floating point unit which is not as fast as other ARM FPUs.

#### sysbench

| Device             |Total Time | Events | Min req. | Avg req. | Max req. |
|-------------------:|----------:|-------:|---------:|---------:|---------:|
| USB armory   1 GHz | 252.3089s |  10000 |  25.17ms |  25.23ms |  77.30ms |
| USB armory 800 MHz | 315.2876s |  10000 |  31.48ms |  31.53ms |  32.67ms |

#### OpenSSL

The standard openssl speed test (-evp <algorithm> -elapsed) for the USB armory performed with the following OpenSSL version:
```
OpenSSL 1.0.2c 12 Jun 2015
platform: linux-armv4
compiler: gcc -I. -I.. -I../include  -fPIC -DOPENSSL_PIC -DZLIB -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -Wa,--noexecstack -D_FORTIFY_SOURCE=2 -march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 -O2 -pipe -fstack-protector --param=ssp-buffer-size=4 -Wl,-O1,--sort-common,--as-needed,-z,relro -O3 -Wall -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DAES_ASM -DBSAES_ASM -DGHASH_ASM
```

| Device             | Algorithm   | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:-------------------|:------------|----------:|----------:|----------:|-----------:|-----------:|
| USB armory   1 GHz | aes-128-cbc | 33570.21k | 40444.07k | 43008.85k |  43665.41k |  43900.93k |
| USB armory 800 MHz | aes-128-cbc | 26857.25k | 32359.13k | 34405.72k |  34959.02k |  35121.83k |
| USB armory   1 GHz | aes-256-cbc | 26992.09k | 31242.22k | 32757.76k |  33170.09k |  33289.56k |
| USB armory 800 MHz | aes-256-cbc | 21590.80k | 25010.94k | 26915.88k |  26533.89k |  26604.89k |
| USB armory   1 GHz | md5         |  8132.58k | 27274.54k | 72405.08k | 120400.21k | 151988.91k |
| USB armory 800 MHz | md5         |  6609.91k | 22986.75k | 58538.33k |  98356.57k | 121632.09k |
| USB armory   1 GHz | sha1        |  7762.81k | 25088.96k | 66894.93k | 115372.37k | 146784.26k |
| USB armory 800 MHz | sha1        |  6096.58k | 20896.64k | 55720.96k |  93835.95k | 117951.15k |
| USB armory   1 GHz | sha256      |  6270.98k | 18815.74k | 45117.18k |  66911.57k |  79549.78k |
| USB armory 800 MHz | sha256      |  5062.33k | 15670.25k | 35452.76k |  53976.06k |  63569.92k |
| USB armory   1 GHz | sha512      |  3396.38k | 13230.72k | 24188.93k |  36439.72k |  42972.50k |
| USB armory 800 MHz | sha512      |  2656.08k | 10449.17k | 19477.08k |  29248.17k |  34384.55k |
