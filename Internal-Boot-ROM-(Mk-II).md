The internal SoC Boot ROM is permanently set by the processor manufacturer (NXP), for
verification purposes its initial 92KB can be dumped with a custom
[imx6ul_bootrom-dump utility](https://github.com/usbarmory/usbarmory/blob/master/software/util/imx6ul_bootrom-dump.c).

The last 4KB section is a protected area and therefore not accessible.

```
$ gcc -o imx6ul_bootrom-dump imx6ul_bootrom-dump.c
$ sudo ./imx6ul_bootrom-dump 0x00000000 92 > imx6ul-bootrom-92K.bin
```

The resulting binary images can be verified by comparing their SHA256 hashes with the following ones:

| Model    | P/N             | SHA256                                                             |
|:---------|-----------------|--------------------------------------------------------------------|
| i.MX6ULZ | MCIMX6Z0DVM09AB | `1727a0f46dbde555b583e9a138ae359389974b7be4369ffd4a252a8730f7e59b` |
| i.MX6ULL | MCIMX6Y2DVM09AB | `1727a0f46dbde555b583e9a138ae359389974b7be4369ffd4a252a8730f7e59b` |
| i.MX6UL  | MCIMX6G3DVM05AB | `1f7bb1207c378e9b3da051b6195151661522eeaec17d557ecd855fb56eb374e7` |
| i.MX6UL  | MCIMX6G2DVM05AB | `ee21dac73ac185510b2d5cb5953db29cb2f20b95be8971a1b2f10b33d337b996` |
