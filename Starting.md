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

## Getting started

The USB armory requires a microSD card with a valid operating system to boot.

**IMPORTANT**: in order to remove/insert the microSD card (with or without enclosure) please check this [FAQ entry](https://github.com/inversepath/usbarmory/wiki/Frequently-Asked-Questions-(FAQ)#how-can-i-removeinsert-the-microsd-card).

### Pre-imaged microSD card

If you received a pre-imaged microSD card with your USB armory it almost certainly includes [Inverse Path](https://inversepath.com) default image, please see notes and default credentials [here](https://github.com/inversepath/usbarmory-debian-base_image/releases) (specifically the **Connecting** section). If you have issues connecting always make sure that:

1. The LED is blinking (indicates Linux kernel heartbeat)

2. The network interface on your USB host, associated to the USB armory (Linux: usbX, macOS/Windows: RNDIS), is correctly set up (either using DHCP or manually) with the required host address (typically 10.0.0.2 with netmask 255.255.255.0).

### Preparing your own microSD card

The following instructions provide information on preparing a bootable microSD.

While every modern microSD card should be supported, the following
compatibility reference keeps track of successfully tested cards:

* [microSD compatibility](https://github.com/inversepath/usbarmory/wiki/microSD-compatibility)

The microSD card can be manually installed, on a computer using a microSD
reader/write (with or without an adapter depending on the reader type), with
either a pre-existing image or from scratch.

The following page keeps track of pre-made images that are available:

* [Available images](https://github.com/inversepath/usbarmory/wiki/Available-images)

Alternatively the following instructions can be used to manually install a
Linux distribution on the microSD from a Linux computer:

* [Preparing a bootable microSD image](https://github.com/inversepath/usbarmory/wiki/Preparing-a-bootable-microSD-image)

## Documentation

For additional information check the available pages on the rest of the [wiki](https://github.com/inversepath/usbarmory/wiki), especially the [Frequently Asked Questions (FAQ)](https://github.com/inversepath/usbarmory/wiki/Frequently-Asked-Questions-(FAQ)), or information available on the [main project page](https://inversepath.com/usbarmory).

## Support

If you think anything is missing or require further support please email us at support@inversepath.com.
