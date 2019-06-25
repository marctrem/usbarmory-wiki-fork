## Handling information

The USB armory board is an open system design, which does not include a
shielded enclosure. This may cause interference to other electrical or
electronic devices in close proximity. In a domestic environment, this product
may cause radio interference in which case the user may be required to take
adequate measures. In addition, this board should not be used near any medical
equipment or RF devices.

The board is sensitive to Electrostatic discharge (ESD). Hold the board only by
its edges. After removing the board from its box, place it on a grounded,
static-free surface.

The board can become hot, like any fan-less design, during continuous high CPU
loads, mind its temperature while handling it.

## Terms of Use

The following Terms of Use apply to USB armory Mk I boards manufactured by
Inverse Path and/or F-Secure and sold directly or through one of its resellers.

IMPORTANT – BEFORE INSTALLING OR USING THE USB ARMORY MK I BOARDS MANUFACTURED
BY INVERSE PATH AND/OR F-SECURE, CAREFULLY READ THE BELOW TERMS OF USE
(”TERMS”). BY OPTING TO ACCEPT, BY INSTALLING OR USING THE USB ARMORY YOU
(EITHER AN INDIVIDUAL OR AN ENTITY) WARRANT THAT YOU HAVE READ THESE TERMS,
UNDERSTAND THEM AND AGREE TO BE LEGALLY BOUND BY THEM. IF YOU DO NOT AGREE TO
ALL OF THE TERMS, DO NOT INSTALL OR USE THE USB ARMORY.

* [Terms of Use](https://github.com/inversepath/usbarmory/wiki/Terms-of-Use)

## Getting started

The USB armory requires a valid operating system to boot on either the external
microSD card (Mk I, Mk II) or the internal eMMC (Mk II).

* [Boot modes](https://github.com/inversepath/usbarmory/wiki/Boot-Modes-(Mk-II)) (Mk II)

### Pre-imaged microSD card

If you received a pre-imaged microSD card with your USB armory it almost
certainly includes [Inverse Path](https://inversepath.com) default image,
please see notes and default credentials
[here](https://github.com/inversepath/usbarmory-debian-base_image/releases)
(specifically the **Connecting** section). If you have issues connecting always
make sure that:

1. The LED is blinking (indicates Linux kernel heartbeat)

2. The network interface on your USB host, associated to the USB armory (Linux:
   usbX, macOS/Windows: RNDIS), is correctly set up (either using DHCP or
   manually) with the required host address (typically 10.0.0.2 with netmask
   255.255.255.0).

### Preparing your own bootable image

The following page keeps track of available binary releases:

* [Available images](https://github.com/inversepath/usbarmory/wiki/Available-images)

Alternatively the following instructions can be used to manually prepare and
install a Linux distribution from a Linux computer:

* [Preparing a bootable image](https://github.com/inversepath/usbarmory/wiki/Preparing-a-bootable-image)

## Documentation

For additional information check the available pages on the rest of the
[wiki](https://github.com/inversepath/usbarmory/wiki), especially the
[Frequently Asked Questions (FAQ)](https://github.com/inversepath/usbarmory/wiki/Frequently-Asked-Questions-(FAQ)),
or information available on the [main project page](https://inversepath.com/usbarmory).

## Support

If you think anything is missing or require further support please email us at
support@inversepath.com.
