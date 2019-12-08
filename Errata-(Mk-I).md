USB armory Mk I (all revisions)
===============================

Errata: Secure Boot feature is deprecated
-----------------------------------------

The NXP i.MX53 System-on-Chip, main processor used in the USB armory Mk I board
design, suffers from vulnerabilities that allow bypass of the optional High
Assurance Boot (HABv4) function.

The USB armory Mk II design is recommended for anyone needing
[Secure boot](https://github.com/inversepath/usbarmory/wiki/Secure-boot-(Mk-II))
capabilities.

Detail information can be found in the related
[security advisory](https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/Security_Advisory-Ref_QBVR2017-0001.txt).
