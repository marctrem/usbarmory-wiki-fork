#### How do I resize the microSD partition on available images smaller than the total available space ?

An excellent tutorial on how to do this can be found [here](http://elinux.org/Beagleboard:Expanding_File_System_Partition_On_A_microSD).

#### Why does 'ps' return no processes and error 'missing btime in /proc/stat' ?

This happens with modern versions of procps when date is not correctly set, this is likely to happen on environments where the USB armory is not routed to the Internet via the USB host (see [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication)) and/or when a NTP client is not installed.

See next question.

#### How do I set the time ?

Like the Raspberry Pi, the USB armory has no battery to hold RTC state. In order to address this a valid time must be set at boot, this can either be done manually using 'date' command or by properly routing the USB armory to the Internet and letting a network time client (such as 'ntpclient' or 'tlsdate') to do its job.

For USB armory-specific application development the recommendation is to implement automatic time/date provisioning from the application client in scenarios where Internet routing is not desired/implemented.