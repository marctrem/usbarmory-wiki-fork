The following benchmarks aim to provide an approximate reference for USB armory CPU performance. 

Please keep in mind that all benchmarks are affected by a multitude of factors including, and not limited to, compilation options, kernel options and configuration, running processes, etc. These results provide only an approximate reference and should be taken with a grain of salt.

Also note that depending on the specific i.MX53 SoC model the USB armory clock can go from 800 Mhz to 1.2 Ghz, only the lowest frequency has been tested.

#### nbench

The [nbench](http://www.tux.org/~mayer/linux/bmark.html) utility is for the USB armory benchmark is compiled with the following gcc (Debian 4.6.3-14) flags:
```
-s -static -Wall -O3 -mfpu=neon -mfloat-abi=hard -mcpu=cortex-a8 -mtune=cortex-a8 -fomit-frame-pointer -marm -funroll-loops -ffast-math
```

| Device                           | Memory Index  | Integer Index | Floating-Point Index |
|:---------------------------------|--------------:|--------------:|---------------------:|
| USB armory @ 800 Mhz             |         5.605 |         5.113 |                1.404 |
| Raspberry Pi 2 Model B @ 900 Mhz |         4.228 |         5.607 |                4.769 | 
| Raspberry Pi Model B+ @ 700 Mhz  |         2.409 |         3.042 |                1.957 |

The USB armory ARM Cortex-A8 CPU includes the VFPLite floating point unit which is not as fast as other ARM FPUs. 

#### sysbench

| Device                           |Total Time | Events | Min req. | Avg req. | Max req. |
|---------------------------------:|----------:|-------:|---------:|---------:|---------:|
| USB armory @ 800 Mhz             | 315.2876s |  10000 |  31.48ms |  31.53ms |  32.67ms |
| Raspberry Pi 2 Model B @ 900 Mhz | 298.6816s |  10000 |  29.64ms |  29.87ms |  44.60ms |
| Raspberry Pi Model B+ @ 700 Mhz  | 523.7819s |  10000 |  51.99ms |  52.37ms |  54.81ms |

#### OpenSSL

The standard openssl speed test (-evp <algorithm> -elapsed) for the USB armory tests is compiled with the following gcc (Debian 4.6.3-14) flags:
```
OpenSSL 1.0.1e 11 Feb 2013
built on: Thu Jan  8 22:02:29 UTC 2015
options:bn(64,32) rc4(ptr,char) des(idx,cisc,16,long) aes(partial) blowfish(ptr) 
compiler: gcc -fPIC -DOPENSSL_PIC -DZLIB -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -DL_ENDIAN -DTERMIO -g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -Wl,-z,relro -Wa,--noexecstack -Wall -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DAES_ASM -DGHASH_ASM
```

| Algorithm                         | 16 bytes  | 64 bytes  | 256 bytes | 1024 bytes | 8192 bytes |
|:----------------------------------|----------:|----------:|----------:|-----------:|-----------:|
| USB armory @ 800 Mhz, aes-128-cbc | 24633.58k | 30861.16k | 34039.30k | 34790.06k  |  35211.95k |
| RPi B+     @ 700 Mhz, aes-128-cbc | 12978.72k | 14708.69k | 15387.40k | 15472.93k  |  15529.06k |
| USB armory @ 800 Mhz, aes-256-cbc | 19723.70k | 24371.41k | 25956.78k | 26540.71k  |  26678.61k |
| RPi B+     @ 700 Mhz, aes-256-cbc | 10267.01k | 11409.83k | 11744.41k | 11812.86k  |  11859.64k |
| USB armory @ 800 Mhz, md5         |  5223.14k | 16725.80k | 43241.39k | 71609.00k  |  88520.02k |
| RPi B+     @ 700 Mhz, md5         | 1954.05k  |  7217.96k | 20805.95k | 39365.29k  |  53226.15k |
| USB armory @ 800 Mhz, sha1        |  4702.72k | 14636.22k | 35539.37k | 55053.31k  |  65456.81k |
| RPi B+     @ 700 Mhz, sha1        | 2115.34k  |  6823.83k | 16264.45k | 25053.18k  |  30121.35k |
| USB armory @ 800 Mhz, sha256      |  4179.83k | 12485.21k | 27873.02k | 40619.01k  |  46940.16k |
| RPi B+     @ 700 Mhz, sha256      | 3598.03k  |  8377.26k | 14605.57k | 17979.39k  |  19300.35k |
| USB armory @ 800 Mhz, sha512      |  2351.87k |  9520.85k | 16779.69k | 26769.07k  |  31741.27k |
| RPi B+     @ 700 Mhz, sha512      | 1080.74k  |  4322.82k |  6151.85k | 8416.32k   |   9418.07k |