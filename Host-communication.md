## Host communication

### Mass Storage emulation

No particular host configuration is required, when configured for mass storage
operation the USB armory is detected as a standard USB flash drive.

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

Linux example (host: 10.0.0.1, USB armory: 10.0.0.2):
```
# bring the USB virtual Ethernet interface up
/sbin/ip link set usb0 up

# set the host IP address
/sbin/ip addr add 10.1.7.1/24 dev usb0

# enable masquerading for outgoing connections towards wireless interface
/sbin/iptables -t nat -A POSTROUTING -s 10.0.0.1/32 -o wlan0 -j MASQUERADE

# enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

```

The Ethernet emulation has been successfully tested on Linux, Mac OS X and
Windows.
