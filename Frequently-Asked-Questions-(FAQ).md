#### How do I resize the microSD partition on available images smaller than the total available space ?

An excellent tutorial on how to do this can be found [here](http://elinux.org/Beagleboard:Expanding_File_System_Partition_On_A_microSD).

#### Why does 'ps' return no processes and error 'missing btime in /proc/stat' ?

This happens with modern versions of procps when date is not correctly set, this is likely to happen on environments where the USB armory is not routed to the Internet via the USB host (see [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication)) and/or when a NTP client is not installed.

The [fake-hwlock](https://packages.debian.org/wheezy/admin/fake-hwclock) package can be used to store a timestamp in the filesystem at reasonable intervals and restore it at boot, so that time is not reset to the epoch at power loss.

However setting the right time is always advised, see next question for more information.

#### How do I set the time ?

Like the Raspberry Pi, the USB armory has no battery to hold RTC state. In order to address this a valid time must be set at boot, this can either be done manually using 'date' command or by properly routing the USB armory to the Internet and letting a network time client (such as 'ntpclient' or 'tlsdate') to do its job.

For USB armory-specific application development the recommendation is to implement automatic time/date provisioning from the application client in scenarios where Internet routing is not desired/implemented. As an example the [INTERLOCK](https://github.com/inversepath/interlock) application for the USB armory sets the time from the host client at each successful login.

#### How do I address locale related errors when installing packages on Debian images ?

Install package 'locales' and then run 'dpkg-reconfigure locales'.