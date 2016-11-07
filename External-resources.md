### SoC Documentation

* [i.MX53 Reference Manual](http://cache.nxp.com/files/32bit/doc/ref_manual/iMX53RM.pdf)
* [i.MX53xD Datasheet](http://cache.nxp.com/files/32bit/doc/data_sheet/IMX53CEC.pdf)
* [i.MX53 System Development User's Guide](http://cache.nxp.com/files/32bit/doc/user_guide/MX53UG.pdf)
* [i.MX53 Secure Boot Application Note](http://cache.nxp.com/files/32bit/doc/app_note/AN4581.pdf)

### Contributions

* [ckuethe](https://github.com/ckuethe) is contributing [scripts, configurations and much more](https://github.com/ckuethe/usbarmory) as well as additional [documentation](https://github.com/ckuethe/usbarmory/wiki)

* [Collin Mulliner](https://github.com/crmulliner) repository [hidemulation](https://github.com/crmulliner/hidemulation) provides HID emulation helpers to support composite CDC Ethernet and HID gadget.

* [Quentin Young](http://qlyoung.net) repository [armory keyboard](https://github.com/qlyoung/armory-keyboard) provides HID keyboard emulation utilities with partial DuckyScript compatibility.

* [Rudd-O](https://github.com/Rudd-O) repository [usbarmory-debian-imagebuilder](https://github.com/Rudd-O/usbarmory-debian-imagebuilder) provides an alternative creation script for Debian images

* [Yann GUIBET](https://github.com/yann2192) published a helpful [guide](https://gist.github.com/yann2192/f989143c86567237460e) on setting up full disk encryption with Arch Linux

* [Carl Smith](https://twitter.com/base16io) published a [guide](http://base16.io/?p=124) on Linux kernel module compilation

* [Christopher](https://yawnbox.com/index.php/about) published a [guide](https://yawnbox.com/index.php/2015/12/02/configuring-a-usb-armory-as-a-reverse-ssh-server-via-tor-hidden-service/) on configuring the USB armory as a reverse SSH server via Tor hidden service.

* [Julien Desfossez](https://github.com/jdesfossez) published a [guide](https://github.com/jdesfossez/usbarmory/wiki/Preparing-a-Ubuntu-FDE-microSD-image) on manually preparing an Ubuntu image for the USB armory with a full disk encryption and signed boot image.

* [Vince Cali](https://twitter.com/0x56) [ported](https://github.com/darwin-on-arm/xnu/commit/410a687039bbbd35b703e1ead996080fae51a887) Darwin on the USB armory, a detailed writeup with build instruction is available [here](http://embeddedideation.com/2016/02/08/darwin-on-armory/).

* [ftorn](https://github.com/ftorn) repository [Docker for usbarmory](https://github.com/ftorn/usbarmory) provides Dockerfile configurations for automatic creation of several application images (e.g. Tor Anonymizing Middlebox, Electrum Bitcoin wallet images).

* Andrea Covello of [scip AG](https://www.scip.ch/en/?) published two excellent tutorials on setting up INTERLOCK on the USB armory: [part 1](https://www.scip.ch/en/?labs.20160811), [part 2](https://www.scip.ch/en/?labs.20160922)

### Papers, Articles

* Jeroen van Kessel and Nick Petros Triantafyllidis authored a paper titled [USB Armory as an Offensive Attack Platform](contrib/USB Armory as an Offensive Attack Platform Jeroen_van_Kessel-and-Nick_Triantafyllidis.pdf), exploring implementation of DNS hijacking and traffic diversion attacks...BadUSB style.

* [Genode Labs](http://www.genode-labs.com) - [The story behind Genode's TrustZone demo on the USB Armory](http://genode.org/documentation/articles/usb_armory)

* Roland Schilling, Frieder Steinmetz authored a paper titled [USB devices phoning home](https://doi.org/10.15480/882.1279), exploring data leaks through rogue HTTP channels established by malicious USB devices and using the USB armory for attack implementation. The repository with the PoC is available [here](https://github.com/willnix/usbpoc).

* Matthias Neugschwandtner, Anton Beitler, Anil Kurmus authored [A Transparent Defense Against USB Eavesdropping Attacks](http://static.securegoose.org/papers/uscramble_cr.pdf), a research which explores passive USB sniffing and countermeasures using the USB armory as evaluation platform.

* [Mubix “Rob” Fuller](https://room362.com/about/) wrote a step by step guide titled ["Snagging creds from locked machines"](https://room362.com/post/2016/snagging-creds-from-locked-machines/) that illustrates how to use the USB armory to capture credential hashes on certain Windows and OS X versions, even when locked out.

### Presentations

* [Inverse Path](https://inversepath.com) - [Forging the USB armory](https://dev.inversepath.com/download/usbarmory/forging_the_usb_armory.pdf)
