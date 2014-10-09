## Development

### Boot procedure

The USB armory board boots by default from the micro SD card if present,
otherwise the board gets enumerated as a vendor specific Freescale USB device
to support the serial downloader.

The serial downloader can be found at the GitHub
[imx usb loader repository](https://github.com/boundarydevices/imx_usb_loader) and can be
used to directly download and execute code on the SoC.

* [Preparing a bootable microSD image](https://github.com/inversepath/usbarmory/wiki/Preparing-a-bootable-microSD-image)
* [External documentation](https://github.com/inversepath/usbarmory/wiki/External-documentation)
