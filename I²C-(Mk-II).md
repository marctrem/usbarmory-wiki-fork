## On-board I²C slaves

The USB armory Mk II has the following components accessible as I²C slaves:

| Models | Address | P/N                       | Role                              | Max I²C frequency |
|--------|--------|----------------------------|-----------------------------------|-------------------|
| all    | 0x08    | NXP F1510                 | Power management controller       |           400 kHz |
| rev. γ | 0x48    | NXP SE050                 | Cryptographic co-processor        |           1.7 MHz |
| rev. β | 0x48    | NXP A71CH                 | Cryptographic co-processor        |           400 kHz |
| rev. β | 0x60    | Microchip ATECC608A       | Cryptographic co-processor        |             1 MHz |
| all    | 0x61    | Texas Instruments TUSB320 | Type-C plug port controller       |           400 kHz |
| all    | 0x31    | ON Semiconductor FUSB303  | Type-C receptacle port controller |           400 kHz |

### Control tool

The USB armory Mk II I²C slaves can be queried/controlled using [armoryctl](https://github.com/f-secure-foundry/armoryctl).

### Manual access

The following example shows how to perform detection of on-board slaves on a
Debian installation running on the USB armory Mk II:

```
apt-get install i2c-tools
modprobe i2c-dev
i2cdetect 0
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-0.
I will probe address range 0x03-0x77.
Continue? [Y/n] Y
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- 08 -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- 31 -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- 48 -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: 60 61 -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

The following example shows how to read the `DEVICE ID` register of the FUSB303
Type-C receptacle port controller:

```
i2cget -y 0 0x31 0x01
0x10
```

## External I²C slaves

Using the [debug accessory](https://github.com/f-secure-foundry/usbarmory/tree/master/hardware/mark-two-debug-accessory)
external I²C slaves can be accessed.
