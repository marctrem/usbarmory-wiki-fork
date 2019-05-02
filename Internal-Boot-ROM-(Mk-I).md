The internal i.MX53 64KB Boot ROM is permanently set by the SoC manufacturer, for verification purposes it can be dumped with a custom [imx53_bootrom-dump utility](https://github.com/inversepath/usbarmory/blob/master/software/util/imx53_bootrom-dump.c).

```
$ gcc -o imx53_bootrom-dump imx53_bootrom-dump.c
$ sudo ./imx53_bootrom-dump 0x00000000 16 > imx53-bootrom-16K.bin
$ sudo ./imx53_bootrom-dump 0x00404000 48 > imx53-bootrom-48K.bin
```

The resulting binary images can be verified by comparing their SHA256 hashes with the following ones:

| Section                   | SHA256                                                             |
|:--------------------------|--------------------------------------------------------------------|
| `0x00000000 - 0x00003fff` | `bee79626931b045024a886e9f0fc298381a301e717894e08c33ea63cb99036c7` |
| `0x00404000 - 0x0040ffff` | `22133edf0f93482a68e77727eab66f0545dccd7807d70bd9db0faf939674a4fb` |
