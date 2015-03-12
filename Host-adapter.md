### Enabling host mode

The USB armory is primarily meant to be attached to a USB host, such as a
laptop or desktop computer. However, it can be also used as a standalone
computer by using a custom host adapter.

The host adapter leverages the fact that it is possible, by changing the Linux
kernel device tree configuration, to invert the role of the USB On-The-Go port
currently used as main plug for the board. This allows USB Armory to be used
independently as a host.

The role change can be enabled by using a device tree source file (dts) with the following configuration:

```
&usbotg {
        dr_mode = "host";
        status = "okay";
};
```

The official USB armory repository provides an example [here](https://github.com/inversepath/usbarmory/blob/master/software/kernel_conf/imx53-usbarmory-host_mode.dts).

The device tree binary file can then be compiled as shown in the [microSD image preparation instructions](https://github.com/inversepath/usbarmory/wiki/Preparing-a-bootable-microSD-image), taking in account the different dts file.

The resulting dtb can be copied to /boot/imx53-usbarmory.dtb on the USB armory when host mode functionality is desired (it is recommended to keep the standard device dtb file around to switch it over when desired).

### Hardware adapter

In order to use host mode an adapter is needed to perform the following functions:

 * Bridging the USB Armory male plug to a USB Type A receptacle (gender changer).
 * Accepting power from a micro-USB input.

This simple conversion enables connection between the USB armory, power supply,
and a USB device or hub. Therefore, it is only a matter of compiling the right
Linux kernel modules and the USB armory can independently use a keyboard, USB
display, USB mass storage devices, USB WiFi dongle and more.

Connecting a powered USB hub to the adapter ensures that all the connected USB
devices have enough power to perform their tasks. Additionally, a micro-USB
cable we can power the USB armory itself. Alternatively, a passive USB hub can
be used and a micro-USB charger (such as ones used for most mobile phones) can
provide power.

The host adapter can be easily built using a breadboard as follows (only
recommended for people with experience in DIY hardware and electronics):

  * D+, D- bridged from a Type A receptacle to the other
  * VBUS, GND shorted between the two Type A receptacles and a micro-USB one

The adapter is available for purchase along with the USB armory
board, please see information at the [USB armory project page](http://inversepath.com/usbarmory).

![USB armory host adapter](http://inversepath.com/images/usbarmory_host_adapter.jpg)
