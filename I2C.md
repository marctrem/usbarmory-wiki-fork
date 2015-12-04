### Device tree

Download and compile the example Inter-Integrated Circuit (I²C) device tree file [imx53-usbarmory-i2c.dts](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/imx53-usbarmory-i2c.dts)
making sure that the latest version of the device tree include file [imx53-usbarmory-common.dtsi](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/imx53-usbarmory-common.dtsi),
is included in your kernel source directory.

Also ensure that i2c support is enabled in the kernel configuration, otherwise
you can recompile the kernel with the latest configuration
[usbarmory_linux-4.3.config](https://raw.githubusercontent.com/inversepath/usbarmory/master/software/kernel_conf/usbarmory_linux-4.3.config).

The example device tree enables this I²C configuration on the pin header:

| PIN | Mk I         |
|:---:|--------------|
|  1  | Ground       |
|  2  | USB 5V       |
|  3  | SDA          |
|  4  | SCL          |
|  5  | GPIO5[28]    |
|  6  | GPIO5[29]    |
|  7  | GPIO5[30]    |


### I²C EEPROM

Once the i2c-dev kernel module is loaded, it is possible to access the I²C
device via character device /dev/i2c-0 using userland tools such as i2c-tools
or, in case of an EEPROM device, a modified verision of
[eeprog](http://darkswarm.org/eeprog-0.7.6-tear5.tar.gz).

The following example shows detection, read and write operations on the
Microchip 24LC16B EEPROM.

```
# apt-get install i2c-tools
# modprobe i2c-dev
# i2cdetect 0
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will probe file /dev/i2c-0.
I will probe address range 0x03-0x77.
Continue? [Y/n]
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: 50 51 52 53 54 55 56 57 -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --

# eeprog /dev/i2c-0 0x50 -fxr 0:32
eeprog 0.7.6-tear12, a 24Cxx EEPROM reader/writer
Copyright (c) 2003-2004 by Stefano Barbato - All rights reserved.
Copyright (c) 2011 by Kris Rusocki - All rights reserved.
  Bus: /dev/i2c-0, Address: 0x50, Mode: 8bit
  Operation: read 32 bytes from offset 0, Output file: <stdout>
  Reading 32 bytes from 0x0

 0000|  ff ff ff ff ff ff ff ff   ff ff ff ff ff ff ff ff
 0010|  ff ff ff ff ff ff ff ff   ff ff ff ff ff ff ff ff

# echo -n "USB armory" | ./eeprog /dev/i2c-0 0x50 -w 0 -f -t 5
eeprog 0.7.6-tear12, a 24Cxx EEPROM reader/writer
Copyright (c) 2003-2004 by Stefano Barbato - All rights reserved.
Copyright (c) 2011 by Kris Rusocki - All rights reserved.
  Bus: /dev/i2c-0, Address: 0x50, Mode: 8bit
  Operation: write at offset 0, Input file: <stdin>
  Write cycle time: 5 milliseconds
  Writing <stdin> starting at address 0x0
..........

# eeprog /dev/i2c-0 0x50 -fxr 0:32
eeprog 0.7.6-tear12, a 24Cxx EEPROM reader/writer
Copyright (c) 2003-2004 by Stefano Barbato - All rights reserved.
Copyright (c) 2011 by Kris Rusocki - All rights reserved.
  Bus: /dev/i2c-0, Address: 0x50, Mode: 8bit
  Operation: read 32 bytes from offset 0, Output file: <stdout>
  Reading 32 bytes from 0x0

 0000|  55 53 42 20 61 72 6d 6f   72 79 ff ff ff ff ff ff
 0010|  ff ff ff ff ff ff ff ff   ff ff ff ff ff ff ff ff
```
