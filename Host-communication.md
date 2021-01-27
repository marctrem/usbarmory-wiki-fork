
The USB armory Mk II supports USB 2.0 connectivity over USB-C connectors, one
dual role port (DRP) receptacle and one Upstream Facing Port (UFP) plug.

The USB armory Mk I supports USB 2.0 connectivity over a single One-The-Go
(OTG) Type-A plug.

The following instructions assume that a Linux distribution is running on the
USB armory, mind that alternatively host communication can also be achieved
in a similar manner when using [TamaGo](https://github.com/f-secure-foundry/tamago)
unikernels ([example](https://github.com/f-secure-foundry/tamago-example)).

# CDC Ethernet

This mode allows to interact with the USB armory just like a standard TCP/IP
server, additionally this also allows the USB armory to contact the USB host
and its services.

When configured for Ethernet operation the USB armory board automatically
triggers the relevant host driver to enable TCP/IP communication.

The network configuration can then proceed with conventional means against the
virtual network interface, depending on the specific USB armory image typically
static addressing or a DHCP server are required.

Once the network is configured the host can decide to route the USB armory.

The Ethernet emulation has been successfully tested on Linux, macOS and
Windows.

Entry in /etc/modules:
```
g_ether
```

Options in /etc/modprobe.d/usbarmory.conf:
```
options g_ether use_eem=0 dev_addr=aa:bb:cc:dd:ee:f1 host_addr=aa:bb:cc:dd:ee:f2
```

## Setup & Connection Sharing: Linux

**NOTE**: The USB armory standard Debian image configures it with IP address
10.0.0.1/24 and default gateway 10.0.0.2, the following setup instructions
reflect these settings. The image comes with a DHCP server enabled by default,
for this reason static IP address configuration should not be required on most
configurations.

**NOTE**: This is a command line example that assumes no interference from
running Network Managers, in general favor following the predefined
configuration files and/or UIs for your specific Linux distribution.
The interface names usb0 and wlan0 should be adjusted to reflect your
configuration.
```
# bring the USB virtual Ethernet interface up
/sbin/ip link set usb0 up

# set the host IP address
/sbin/ip addr add 10.0.0.2/24 dev usb0

# enable masquerading for outgoing connections towards wireless interface
/sbin/iptables -t nat -A POSTROUTING -s 10.0.0.1/32 -o wlan0 -j MASQUERADE

# enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

```

## Setup & Connection Sharing: macOS

### Without connection sharing

**NOTE**: The USB armory standard Debian image configures it with IP address
10.0.0.1/24 and default gateway 10.0.0.2, the following setup instructions
reflect these settings. The image comes with a DHCP server enabled by default,
for this reason static IP address configuration should not be required on most
configurations.

1. Open 'System Preferences' -> 'Network'.

2. Select the 'RNDIS/Ethernet Gadget' interface.

3. Set IPv4 configuration to Manual with IP Address 10.0.0.2, Netmask 255.255.255.0.

### With connection sharing

The Internet Connection Sharing on macOS requires an IP range on a
**different** subnet, this requires re-configuring the USB armory with a static
IP address between the range 192.168.2.2-192.168.2.254 on its usb0 interface,
using 192.168.2.1 as default gateway.

Example interface entry for `/etc/network/interfaces`:

```
iface usb0 inet static
  address 192.168.2.2
  netmask 255.255.255.0
  gateway 192.168.2.1

```

Once the USB armory is re-configured the macOS interface must be then set as
follows:

1. Open 'System Preferences' -> 'Network'.

2. Select the 'RNDIS/Ethernet Gadget' interface.

3. Set IPv4 configuration to Manual with IP Address 192.168.2.1, Netmask 255.255.255.0.

Once these steps are performed Internet Connection Sharing can be enabled as
follows:

1. On the Mac choose Apple menu > 'System Preferences' and click Sharing.

2. Select Internet Sharing.

3. Choose the Internet connection you want to share in 'Share your connection
from'

4. Depending on the chosen USB armory Ethernet emulation tick the RNDIS or
CDC Ethernet checkboxes in 'To computers using'.

Alternatively Internet Connection Sharing can also be manually enabled without
having to modify the default USB armory usb0 address, the following example
assumes en0 as the Internet interface and en5 as the RNDIS/CDC USB armory
interface on the host machine (your assignment might vary):

```
# enable IP forwarding
sudo sysctl -w net.inet.ip.forwarding=1

# enable PF firewall
sudo pfctl -e

# Option 1: add NAT rule after en5 is up (USB armory already plugged and started)
echo "nat on en0 from en5:network to any -> (en0)" | sudo pfctl -f -

# Option 2: add NAT rule before USB armory is plugged, requires specifying its network
echo "nat on en0 from 10.0.0.0/8 to any -> (en0)" | sudo pfctl -f -
```

## Setup & Connection Sharing: Windows 7, 8, 10

**NOTE**: The USB armory standard Debian image configures it with IP address
10.0.0.1/24 and default gateway 10.0.0.2, the following setup instructions
reflect these settings. The image comes with a DHCP server enabled by default,
for this reason static IP address configuration should not be required on most
configurations.

1. Plug in the USB armory and let Windows install the network adapter driver. The driver installation is usually automatic, however on certain Windows installations it has been reported that this is not the case. To manually install a driver, success has been reported with the [Linux USB Ethernet/RNDIS Gadget](https://www.kernel.org/doc/Documentation/usb/linux.inf) or the [Acer USB Ethernet/RNDIS Gadget](http://catalog.update.microsoft.com/v7/site/ScopedViewRedirect.aspx?updateid=37e35bd4-d788-4b83-9416-f78e439f90a2) (in case of driver issues some tips can be found in this [thread](https://groups.google.com/forum/#!topic/usbarmory/bFZBrTKq2Mg)) (it has also been reported that adding options `idVendor=0x04b3 idProduct=0x4010` to g_ether can trigger successful usage of an IBM driver that is typically included in Windows installations).

2. Once installation is complete go on the Control Panel, choose 'Network and Internet' -> 'Network and Sharing Center' ->  'Change adapter settings'.

3. You should see a 'USB Ethernet/RNDIS Gadget' adapter, select it, right click to select 'Properties'.

4. Select 'Internet Protocol Version 4' (ensure that its check-box remains ticked) and then click on 'Properties'.

5. Select 'Use the following IP Address', set IP address to 10.0.0.2, netmask to 255.255.255.0.

The Internet Connection Sharing on Windows requires by default an IP range on a **different** subnet than the one shown in the previous section. This requires re-configuring the USB armory with an IP address between the range 192.168.137.2-192.168.137.254 on its usb0 interface, using 192.168.137.1 as default gateway (which must be set as static IP address on the 'RNDIS/Ethernet Gadget' interface). Once this is performed Internet Connection Sharing can be enabled as follows:

1. Go on the Control Panel, choose 'Network and Internet' -> 'Network and Sharing Center' ->  'Change adapter settings'.
2. Select the Internet connection to be shared, right click to Properties.

3. Choose the 'Sharing' tab and select 'Allow other network users to connect through this computer’s Internet connection'.

4. Depending on the interface assigned to the USB armory Ethernet emulation check its relevant entry in the "Home networking connection" selector.

## Connecting

When using CDC Ethernet emulation any standard TCP/IP server can be used to communicate with the USB armory board.

The typical, and first, interaction would be via OpenSSH server, using an SSH client on the USB host.

# Mass Storage emulation

No particular host configuration is required, when configured for mass storage
operation the USB armory is detected as a standard USB flash drive.

Entry in /etc/modules:
```
g_mass_storage
```

Options in /etc/modprobe.d/usbarmory.conf:
```
options g_mass_storage file=disk.img
```

# Combined CDC Ethernet and Mass Storage emulation

It is possible to have simultaneous ethernet and mass storage emulation with the Multifunction Composite Gadget (g_multi).

Entry in /etc/modules:
```
g_multi
```

Options in /etc/modprobe.d/usbarmory.conf:
```
options g_multi use_eem=0 dev_addr=aa:bb:cc:dd:ee:f1 host_addr=aa:bb:cc:dd:ee:f2 file=disk.img
```

# Mouse/Keyboard emulation via USB Human Interface Device (HID) Gadget

The following respository, contributed by [Collin Mulliner](https://github.com/crmulliner) provides HID emulation helpers to support composite CDC Ethernet and HID gadget.

[https://github.com/crmulliner/hidemulation](https://github.com/crmulliner/hidemulation)


# Serial Download Protocol (SDP)

For the USB armory Mk I and Mk II the [imx usb loader repository](https://github.com/boundarydevices/imx_usb_loader)
provides a tool that can be used to directly download and execute code on the SoC via USB.

The ```verify``` and ```debugmode``` flags can also be used as debugging aid to verify correct SoC operation.

```
sudo imx_usb -v -d image.imx
```

Only for the [USB armory Mk II SDP protocol](https://github.com/f-secure-foundry/usbarmory/wiki/Boot-Modes-(Mk-II))
there is also our own equivalent [armory-boot-usb loader](https://github.com/f-secure-foundry/armory-boot/tree/master/cmd/armory-boot-usb).

```
sudo armory-boot-usb -i image.imx
```

# USB VID/PID

For custom drivers, or whenever Linux gadget default USB VID/PID values are not desired, the USB armory has been assigned a USB Product ID through the [pid.codes](http://pid.codes/) project: ID [1209:2702](http://pid.codes/1209/2702/).