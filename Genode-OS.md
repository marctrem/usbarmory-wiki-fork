The [Genode OS Framework](http://genode.org/index) has basic support for the USB armory since version [15.02](http://genode.org/documentation/release-notes/15.02#Support_for_the_USB-Armory_board). Genode OS version [15.11](http://genode.org/documentation/release-notes/15.11#Improved_TrustZone_support_on_USB_Armory) introduces a TrustZone Secure [virtual-machine monitor](http://genode.org/documentation/articles/trustzone) (VMM) supervising Linux running in the Normal world.

To give it a try you have to first install the [Genode toolchain](http://genode.org/download/tool-chain).

Then download the Genode sources and create a build directory for the USB armory:

```
git clone https://github.com/genodelabs/genode
cd genode
./tool/create_builddir hw_usb_armory
```

The Genode 'tz_vmm' scenario integrates a Secure Genode VMM and a sample Linux guest with initrd image. A tutorial on how to compile it, and bring it to a bootable microSD card, can be found [here](https://github.com/genodelabs/genode/blob/master/repos/os/run/tz_vmm.run).

The Linux kernel requires minimal patching to be executed in the Normal world. The 'tz_vmm' scenario tutorial downloads a pre-built Linux images for that purpose, if you are interested in building a custom Linux guest a separate tutorial is [available](https://github.com/m-stein/genode_binaries/blob/master/tz_vmm/usb_armory/README).

### External documentation

* https://github.com/genodelabs/genode/blob/master/repos/os/run/tz_vmm.run
* https://github.com/m-stein/genode_binaries/blob/master/tz_vmm/usb_armory/README
* [Genode Labs](http://www.genode-labs.com) - [The story behind Genode's TrustZone demo on the USB Armory](http://genode.org/documentation/articles/usb_armory)