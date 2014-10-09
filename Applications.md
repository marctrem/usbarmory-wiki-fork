The following example security application ideas illustrate the flexibility of
the USB armory concept:

* mass storage device with advanced features such as automatic encryption, virus scanning, host authentication and data self-destruct
* OpenSSH client and agent for untrusted hosts (kiosk)
* router for end-to-end VPN tunnelling
* password manager with integrated web server
* electronic wallet (e.g. pocket Bitcoin wallet)
* authentication token
* portable penetration testing platform
* low level USB security testing

This section is meant to track available software PoC, projects and/or
procedures oriented towards implementing such application ideas and any other
interesting USB armory usage. If you have tested an interesting use case for
the USB armory please submit it at info@inversepath.com for inclusion.

See also [Host configuration](https://github.com/inversepath/usbarmory/wiki/Host-configuration)
for communication/routing options.

### Bitcoin wallet

The Electrum (https://electrum.org/) Bitcoin wallet works out of the box on the
USB armory, it has been tested with X11 forwarding from Linux as well as
Windows hosts.
