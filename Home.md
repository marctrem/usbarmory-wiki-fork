Welcome to the USB armory wiki.

The USB armory from [Inverse Path](http://inversepath.com) is an open source
hardware design, implementing a flash drive sized computer.

For a project introduction and information about purchasing an USB armory board
please see information at the [USB armory project page](http://inversepath.com/usbarmory).

## Hardware

The first hardware revision of the USB armory is currently being finalized, for
information and updates see the [project
page](http://inversepath.com/usbarmory).

## Available images

None at this time, this section will be soon updated with links to the
available disk images that, when copied on a microSD, allows to boot the USB
armory board.

The alpha board has been successfully tested with a vanilla Debian
installation, Linux kernel with published Freescale patches for the i.MX53 and
a patched version of U-Boot.

## Applications

## Host configuration

### Mass Storage emulation

No particular host configuration is required.

### CDC Ethernet

When configured for Ethernet operation the USB armory board triggers the host
relevant driver.

The network configuration can then proceed with conventional means against the
virtual network interface, depending on the specific USB armory image typically
static addressing or a DHCP server are required.

Once the network is configured the host can decide to route the USB armory.

Linux example (host: 10.0.0.1, USB armory: 10.0.0.2):
```
# bring the USB virtual Ethernet interface up
/sbin/ip link set usb0 up

# set the host IP address
/sbin/ip addr add 10.1.7.1/24 dev usb0

# enable masquerading for outgoing connections towards wireless interface
/sbin/iptables -t nat -A POSTROUTING -s 10.0.0.1/32 -o wlan0 -j MASQUERADE

# enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

```

## Development

### Boot procedure

The USB armory board boots by default from the micro SD card if present,
otherwise the board gets enumerated as a vendor specific Freescale USB device
to support the serial downloader.

The serial downloader can be found at the GitHub
[imx usb loader repository](https://github.com/boundarydevices/imx_usb_loader) and can be
used to directly download and execute code on the SoC.

[Preparing a bootable microSD image](https://github.com/inversepath/usbarmory/wiki/Preparing-a-bootable-microSD-image)

## Developer Notes

None at this time, this section will be soon updated.

## Useful documentation

[i.MX53 System Development  Guide](http://cache.freescale.com/files/32bit/doc/user_guide/MX53UG.pdf)

[i.MX53 Secure Boot Application Note](http://cache.freescale.com/files/32bit/doc/app_note/AN4581.pdf)

## How to Help

Obtain a USB armory board and start developing! Please see ordering information
at the [USB armory project page](http://inversepath.com/usbarmory).

A list of project ideas is also available on the [project page](http://inversepath.com/usbarmory).

A discussion group is available on [Google Groups](https://groups.google.com/d/forum/usbarmory).

