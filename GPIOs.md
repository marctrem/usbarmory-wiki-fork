### Breakout header

| PIN | Alpha             | Beta                                  |
|:---:|-------------------|---------------------------------------|
|  1  | Ground            | Ground                                |
|  2  | USB 5V            | USB 5V                                |
|  3  | GPIO4\[0\] (GPIO10) | GPIO5\[27\]            (CSI0_DAT9)  |
|  4  | GPIO4\[1\] (GPIO11) | UART1_TX / GPIO5\[28\] (CSI0_DAT10) |
|  5  | GPIO4\[2\] (GPIO12) | UART1_RX / GPIO5\[29\] (CSI0_DAT11) |
|  6  | GPIO4\[3\] (GPIO13) | GPIO5\[30\]            (CSI0_DAT12) |
|  7  | GPIO4\[4\] (GPIO14) | GPIO5\[31\]            (CSI0_DAT13) |


### GPIO Sysfs Interface on Linux

When using Linux the 5 GPIOs, exposed on the breakout header (pins 2-7), are
easily configurable using the GPIO Sysfs interface.

The following example shows how to set bit 4 of the i.MX53 GPIO4 interface,
corresponding to pin 7, in output mode and write 1 and 0 from a Linux shell.

```
# echo 100 > /sys/class/gpio/export             # 96 (GPIO4[0]) + 4 == GPIO4[4]
# echo out > /sys/class/gpio/gpio100/direction
# echo 1 > /sys/class/gpio/gpio100/value
# echo 0 > /sys/class/gpio/gpio100/value
```
