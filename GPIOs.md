### Breakout header

| PIN | Description     |  i.MX53 pad |
|:---:|-----------------|:-----------:|
|  1  | Ground          |   N/A       |
|  2  | USB 5V          |   N/A       |
|  3  | GPIO4[0]        |   GPIO10    |
|  4  | GPIO4[1]        |   GPIO11    |
|  5  | GPIO4[2]        |   GPIO12    |
|  6  | GPIO4[3]        |   GPIO13    |
|  7  | GPIO4[4]        |   GPIO14    |


### GPIO Sysfs Interface

The 5 GPIOs present on the breakout header (pins 2-7) are configurable
using Linux GPIO Sysfs interface.

The following example shows how to set bit 4 of the i.MX53 GPIO4 interface
(pin 7) in output mode and write 1 and 0 from linux shell.

```
# echo 100 > /sys/class/gpio/export
# echo out > /sys/class/gpio/gpio100/direction
# echo 1 > /sys/class/gpio/gpio100/value
# echo 0 > /sys/class/gpio/gpio100/value
```
