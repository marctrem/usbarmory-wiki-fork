### Breakout header

| PIN | Beta                                  | Mk I                                |
|:---:|---------------------------------------|-------------------------------------|
|  1  | Ground                                | Ground                              |
|  2  | USB 5V                                | USB 5V                              |
|  3  | GPIO5\[27\]            (CSI0_DAT9)    | GPIO5\[26\]            (CSI0_DAT8)  |
|  4  | UART1_TX / GPIO5\[28\] (CSI0_DAT10)   | GPIO5\[27\]            (CSI0_DAT9)  |
|  5  | UART1_RX / GPIO5\[29\] (CSI0_DAT11)   | UART1_TX / GPIO5\[28\] (CSI0_DAT10) |
|  6  | GPIO5\[30\]            (CSI0_DAT12)   | UART1_RX / GPIO5\[29\] (CSI0_DAT11) |
|  7  | GPIO5\[31\]            (CSI0_DAT13)   | GPIO5\[30\]            (CSI0_DAT12) |

The breakout header can be accessed, among other options, with a solderless header such as [this one](https://www.sparkfun.com/products/10527) or [this one (346-87-107-41-036101)](http://www.precidip.com/pview/346-PP-1NN-41-036101.html).

The serial port can be connected to via a [USB to TTL cable](https://www.sparkfun.com/products/12977).

The header can also be used to route [I²C](https://github.com/inversepath/usbarmory/wiki/I2C) (Inter-Integrated Circuit) and [SPI](https://github.com/inversepath/usbarmory/wiki/SPI) (Serial Peripheral Interface) interfaces.

### GPIO Sysfs Interface on Linux

When using Linux the 5 GPIOs, exposed on the breakout header (pins 2-7), are
easily configurable using the GPIO Sysfs interface.

The following example shows how to set bit 30 of the i.MX53 GPIO5 interface,
corresponding to pin 7, in output mode and write 1 and 0 from a Linux shell.

```
# echo 158 > /sys/class/gpio/export             # 128 (GPIO5[0]) + 30 = GPIO5[30]
# echo out > /sys/class/gpio/gpio158/direction
# echo 1 > /sys/class/gpio/gpio158/value
# echo 0 > /sys/class/gpio/gpio158/value
```

### LED Control

The Mk I LED can be controlled in different ways depending on your Linux kernel configuration. The following examples shows usage on with default Debian and Arch Linux images.

Debian examples:

```
## unload the default LED modules
# modprobe -r leds_gpio led_class ledtrig_heartbeat

# echo 123 > /sys/class/gpio/export             # 96 (GPIO4[27]) + 27 == GPIO4[27]

## full brightness
# echo out > /sys/class/gpio/gpio123/direction
# echo 0 > /sys/class/gpio/gpio123/value        # On
# echo 1 > /sys/class/gpio/gpio123/value        # Off

## mid-brightness trick
# echo in > /sys/class/gpio/gpio123/direction
```

To permanently disable the default LED usage
comment the ledtrig_heartbeat line from /etc/modules
and blacklist leds_gpio and led_class:

```
sed -i /etc/modules -e 's/ledtrig_heartbeat/#ledtrig_heartbeat/'
echo -e "blacklist leds_gpio\nblacklist led_class" > /etc/modprobe.d/led-blacklist.conf
```

Arch Linux examples:

```
## trigger on USB activity
# echo usb-gadget > /sys/devices/platform/leds/leds/LED/trigger

## trigger on microSD activity
# echo mmc0 > /sys/devices/platform/leds/leds/LED/trigger

## disable trigger
# echo none > /sys/devices/platform/leds/leds/LED/trigger

# LED off
# echo 1 > /sys/devices/platform/leds/leds/LED/brightness

# LED on
# echo 0 > /sys/devices/platform/leds/leds/LED/brightness