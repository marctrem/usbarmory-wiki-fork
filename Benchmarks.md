The following benchmarks aim to provide an approximate reference for USB armory CPU performance. 

Please keep in mind that all benchmarks are affected by a multitude of factors including, and not limited to, compilation options, kernel options and configuration, running processes, etc. These results provide only an approximate reference and should be taken with a grain of salt.

The USB armory i.MX53 CPU clocks by default at 800 MHz, models mounting the consumer version (i.MX535) can be clocked at 1 GHz (MCIMX535DVV1) and 1.2 GHz (MCIMX535DVV2). The first production batch mounts the automotive version (i.MX534, P/N MCIMX534AVV8C), while officially advertised as 800 MHz clock this model has been positively tested running with a 1 GHz clock without any issues.

#### nbench

The [nbench](http://www.tux.org/~mayer/linux/bmark.html) utility is for the USB armory benchmark is compiled with the following gcc (Debian 4.6.3-14) flags:
```
-s -static -Wall -O3 -mfpu=neon -mfloat-abi=hard -mcpu=cortex-a8 -mtune=cortex-a8 -fomit-frame-pointer -marm -funroll-loops -ffast-math
```

| Device             | Memory Index  | Integer Index | Floating-Point Index |
|:-------------------|--------------:|--------------:|---------------------:|
| USB armory   1 GHz |         7.005 |         6.401 |                1.751 |
| USB armory 800 MHz |         5.605 |         5.113 |                1.404 |
| RPi 2      900 MHz |         4.228 |         5.607 |                4.769 | 
| RPi 1      700 MHz |         2.409 |         3.042 |                1.957 |

The USB armory ARM Cortex-A8 CPU includes the VFPLite floating point unit which is not as fast as other ARM FPUs.

#### sysbench

| Device             |Total Time | Events | Min req. | Avg req. | Max req. |
|-------------------:|----------:|-------:|---------:|---------:|---------:|
| USB armory   1 GHz | 252.3089s |  10000 |  25.17ms |  25.23ms |  77.30ms |
| USB armory 800 MHz | 315.2876s |  10000 |  31.48ms |  31.53ms |  32.67ms |
| RPi 2      900 MHz | 298.6816s |  10000 |  29.64ms |  29.87ms |  44.60ms |
| RPi 1      700 MHz | 523.7819s |  10000 |  51.99ms |  52.37ms |  54.81ms |

#### OpenSSL

The standard openssl speed test (-evp <algorithm> -elapsed) for the USB armory tests is compiled with the following gcc (Debian 4.6.3-14) flags:
```
OpenSSL 1.0.1e 11 Feb 2013
built on: Thu Jan  8 22:02:29 UTC 2015
options:bn(64,32) rc4(ptr,char) des(idx,cisc,16,long) aes(partial) blowfish(ptr) 
compiler: gcc -fPIC -DOPENSSL_PIC -DZLIB -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -DL_ENDIAN -DTERMIO -g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -Wl,-z,relro -Wa,--noexecstack -Wall -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DAES_ASM -DGHASH_ASM
```

| Device             | Algorithm   | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:-------------------|:------------|----------:|----------:|----------:|-----------:|-----------:|
| USB armory   1 GHz | aes-128-cbc | 30000.81k | 39167.77k | 42836.22k | 43861.33k  |  44316.25k |
| USB armory 800 MHz | aes-128-cbc | 24633.58k | 30861.16k | 34039.30k | 34790.06k  |  35211.95k |
| RPi 1      700 MHz | aes-128-cbc | 12978.72k | 14708.69k | 15387.40k | 15472.93k  |  15529.06k |
| USB armory   1 GHz | aes-256-cbc | 24144.75k | 29751.57k | 31817.73k | 32489.91k  |  32658.41k |
| USB armory 800 MHz | aes-256-cbc | 19723.70k | 24371.41k | 25956.78k | 26540.71k  |  26678.61k |
| RPi 1      700 MHz | aes-256-cbc | 10267.01k | 11409.83k | 11744.41k | 11812.86k  |  11859.64k |
| USB armory   1 GHz | md5         |  6294.64k | 20913.71k | 54362.37k | 89534.24k  | 110712.15k |
| USB armory 800 MHz | md5         |  5223.14k | 16725.80k | 43241.39k | 71609.00k  |  88520.02k |
| RPi 1      700 MHz | md5         |  1954.05k |  7217.96k | 20805.95k | 39365.29k  |  53226.15k |
| USB armory   1 GHz | sha1        |  5769.64k | 18817.41k | 43909.97k | 67154.54k  |  81788.93k |
| USB armory 800 MHz | sha1        |  4702.72k | 14636.22k | 35539.37k | 55053.31k  |  65456.81k |
| RPi 1      700 MHz | sha1        |  2115.34k |  6823.83k | 16264.45k | 25053.18k  |  30121.35k |
| USB armory   1 GHz | sha256      |  5148.47k | 15314.75k | 33576.67k | 50387.31k  |  58755.75k |
| USB armory 800 MHz | sha256      |  4179.83k | 12485.21k | 27873.02k | 40619.01k  |  46940.16k |
| RPi 1      700 MHz | sha256      |  3598.03k |  8377.26k | 14605.57k | 17979.39k  |  19300.35k |
| USB armory   1 GHz | sha512      |  2878.30k | 11499.56k | 21367.55k | 33400.89k  |  39652.01k |
| USB armory 800 MHz | sha512      |  2351.87k |  9520.85k | 16779.69k | 26769.07k  |  31741.27k |
| RPi 1      700 MHz | sha512      |  1080.74k |  4322.82k |  6151.85k | 8416.32k   |   9418.07k |