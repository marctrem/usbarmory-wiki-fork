### How can I remove/insert the microSD card? (Mk I)

1. (official enclosure only, if present) To remove the microSD sliding cover push it outwards with your thumb, on the side of the memory card, and pull it simultaneously with your other hand index fingernail, on the opposite side.

2. Please note that the microSD slot is with "hinge" type insertion and not a pull-push slot. In order to open it gently slide the closed metal hinge towards the opposite direction of the little arrow engraved on it, a small "click" indicates successful opening. Lift the hinge upwards and place the microSD card from the top, once inserted close it by putting the hinge back in position and by sliding it towards the arrow direction.

### How can I remove/insert the microSD card? (Mk II)

The microSD slot is a "Push-Push" kind.

### Help, I can't connect to my USB armory.

If you received a pre-imaged microSD card with your USB armory it almost certainly includes the default image, please see notes and default credentials [here](https://github.com/f-secure-foundry/usbarmory-debian-base_image/releases) (specifically the **Connecting** section). If you have issues connecting always make sure that:

1. the LED is blinking (indicates Linux kernel heartbeat)

2. the network interface on your USB host, associated to the USB armory (Linux: usbX, macOS/Windows: RNDIS), is correctly set up with the required host address (typically 10.0.0.2 with netmask 255.255.255.0)

### How do I resize the microSD/eMMC partition on available images smaller than the total available space ?

A tutorial on how to do this can be found [here](http://elinux.org/Beagleboard:Expanding_File_System_Partition_On_A_microSD).

### Why does 'ps' return no processes and error 'missing btime in /proc/stat'?

This happens with modern versions of procps when date is not correctly set, this is likely to happen on environments where the USB armory is not routed to the Internet via the USB host (see [Host communication](https://github.com/f-secure-foundry/usbarmory/wiki/Host-communication)) and/or when a NTP client is not installed.

The [fake-hwlock](https://packages.debian.org/wheezy/admin/fake-hwclock) package can be used to store a timestamp in the filesystem at reasonable intervals and restore it at boot, so that time is not reset to the epoch at power loss.

However setting the right time is always advised, see next question for more information.

### How do I set the time?

Like the Raspberry Pi, the USB armory has no battery to hold RTC state. In order to address this a valid time must be set at boot, this can either be done manually using 'date' command or by properly routing the USB armory to the Internet and letting a network time client (such as 'ntpclient' or 'tlsdate') to do its job.

For USB armory-specific application development the recommendation is to implement automatic time/date provisioning from the application client in scenarios where Internet routing is not desired/implemented. As an example the [INTERLOCK](https://github.com/f-secure-foundry/interlock) application for the USB armory sets the time from the host client at each successful login.

### How do I address locale related errors when installing packages on Debian images?

Install package 'locales' and then run ```dpkg-reconfigure locales```.

### I get the following Linux kernel errors on the host: "Dual-Role OTG device on non-HNP port", "can't set HNP mode"

Remove the CONFIG_USB_OTG kernel option on your host.

### Why is LUKS not working? (Mk I)

There is a known issue in modern Linux kernels which breaks cryptsetup aes-xts-plain64 when the SAHARA driver (`sahara` kernel module) is loaded. For this reason it is recommended to blacklist the module when using LUKS on the USB armory.

### No serial console with debug accessory (Mk II)

The connection between the debug accessory and the target is supported only
with the same orientation for both top layers (side with components for the
accessory, side with LEDs for the USB armory).
