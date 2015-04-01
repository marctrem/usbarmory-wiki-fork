### Mass Storage emulation

No particular host configuration is required, when configured for mass storage
operation the USB armory is detected as a standard USB flash drive.

Entry in /etc/modules:
```
g_mass_storage file=disk.img
```

### CDC Ethernet

This mode allows to interact with the USB armory just like a standard TCP/IP
server, additionally this also allows the USB armory to contact the USB host
and its services.

When configured for Ethernet operation the USB armory board automatically
triggers the relevant host driver to enable TCP/IP communication.

The network configuration can then proceed with conventional means against the
virtual network interface, depending on the specific USB armory image typically
static addressing or a DHCP server are required.

Once the network is configured the host can decide to route the USB armory.

The Ethernet emulation has been successfully tested on Linux, Mac OS X and
Windows.

Entry in /etc/modules:
```
g_ether use_eem=0 dev_addr=aa:bb:cc:dd:ee:f1 host_addr=aa:bb:cc:dd:ee:f2
```

#### Connection Sharing: Linux

Routing example (host: 10.0.0.2 with wlan0 as Internet interface, USB armory: 10.0.0.1):
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

This is a command line example that assumes no interference from running Network Managers, in general favor following the predefined configuration files and/or UIs for your specific Linux distribution.

#### Connection Sharing: Mac OS X (Yosemite)

1. For connection sharing ensure that the USB armory image assigns an IP
address between the range 192.168.2.2-192.168.2.254 to CDC/RNDIS interface, using
192.168.2.1 as default gateway. (**NOTE**: the official pre-imaged microSD card for
the USB armory does not assign an address on this subnet, please follow its [instructions](http://dev.inversepath.com/download/usbarmory/) for the first connection)

2. On the Mac choose Apple menu > 'System Preferences' and click Sharing.

3. Select Internet Sharing.

4. Choose the Internet connection you want to share in 'Share your connection
from'

5. Depending on the chosen USB armory Ethernet emulation tick the RNDIS or
CDC Ethernet checkboxes in 'To computers using'.

#### Connection Sharing: Windows 7

1. For connection sharing ensure that the USB armory image assigns an IP
address between the range 192.168.137.2-192.168.137.254 to CDC/RNDIS interface, using
192.168.137.1 as default gateway. (**NOTE**: the official pre-imaged microSD card for
the USB armory does not assign an address on this subnet, please follow its [instructions](http://dev.inversepath.com/download/usbarmory/) for the first connection)

2. On the Control Panel choose 'Network and Sharing Center' and click 'View network connections'.

3. Select the Internet connection to be shared, right click to Properties.

4. Choose the Sharing tab and select 'Allow other network users to connect through this computer’s Internet connection'.

5. Depending on the interface assigned to the USB armory Ethernet emulation check its relevant entry in the "Home networking connection" selector.

#### Connecting

When using CDC Ethernet emulation any standard TCP/IP server can be used to communicate with the USB armory board.

On Linux installations the typical interaction would be via OpenSSH server, using an SSH client on the USB host, or web server, using a standard browser as a client.

The installation of the [shellinabox](https://code.google.com/p/shellinabox/) software (available on Debian with apt) provides an easily accessible web terminal emulator (default port: 4200), however using a proper SSH client/server is highly recommended over this method.