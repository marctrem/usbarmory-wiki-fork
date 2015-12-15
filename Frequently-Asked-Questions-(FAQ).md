#### How can I remove/insert the microSD card?

1. (official enclosure only, if present) To remove the microSD sliding cover push it outwards with your thumb, on the side of the memory card, and pull it simultaneously with your other hand index fingernail, on the opposite side.

2. Please note that the microSD slot is with "hinge" type insertion and not a pull-push slot. In order to open it gently slide the closed metal hinge towards the opposite direction of the little arrow engraved on it, a small "click" indicates successful opening. Lift the hinge upwards and place the microSD card from the top, once inserted close it by putting the hinge back in position and by sliding it towards the arrow direction.

#### Help, I can't connect to my USB armory.

If you received a pre-imaged microSD card with your USB armory it almost certainly includes [Inverse Path](https://inversepath.com) default image, please see notes and default credentials [here](https://dev.inversepath.com/usbarmory) (specifically the **CONNECTING** section). If you have issues connecting always make sure that:

1. the LED is blinking (indicates Linux kernel heartbeat)

2. the network interface on your USB host, associated to the USB armory (Linux: usbX, OSX, Windows: RNDIS), is correctly set up with the required host address (typically 10.0.0.2 with netmask 255.255.255.0)

At this time all available images, with the exception of Kali Linux, configure the USB armory with IP address 10.0.0.1 (gateway/host address 10.0.0.2). The Kali Linux image configures the USB armory with IP address 10.42.0.3 (gateway/host address 10.42.0.1).

#### How do I resize the microSD partition on available images smaller than the total available space ?

Some tutorials on how to do this can be found [here](http://base16.io/?p=61) and [here](http://elinux.org/Beagleboard:Expanding_File_System_Partition_On_A_microSD).

#### Why does 'ps' return no processes and error 'missing btime in /proc/stat' ?

This happens with modern versions of procps when date is not correctly set, this is likely to happen on environments where the USB armory is not routed to the Internet via the USB host (see [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication)) and/or when a NTP client is not installed.

The [fake-hwlock](https://packages.debian.org/wheezy/admin/fake-hwclock) package can be used to store a timestamp in the filesystem at reasonable intervals and restore it at boot, so that time is not reset to the epoch at power loss.

However setting the right time is always advised, see next question for more information.

#### How do I set the time ?

Like the Raspberry Pi, the USB armory has no battery to hold RTC state. In order to address this a valid time must be set at boot, this can either be done manually using 'date' command or by properly routing the USB armory to the Internet and letting a network time client (such as 'ntpclient' or 'tlsdate') to do its job.

For USB armory-specific application development the recommendation is to implement automatic time/date provisioning from the application client in scenarios where Internet routing is not desired/implemented. As an example the [INTERLOCK](https://github.com/inversepath/interlock) application for the USB armory sets the time from the host client at each successful login.

#### How do I address locale related errors when installing packages on Debian images ?

Install package 'locales' and then run ```dpkg-reconfigure locales```.

#### I get the following Linux kernel errors on the host: "Dual-Role OTG device on non-HNP port", "can't set HNP mode"

Remove the CONFIG_USB_OTG kernel option on your host.
