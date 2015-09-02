The following example security application ideas illustrate the flexibility of
the USB armory concept:

* Hardware Security Module (HSM)
* file storage with advanced features such as automatic encryption, virus scanning, host authentication and data self-destruct
* OpenSSH client and agent for untrusted hosts (kiosk)
* router for end-to-end VPN tunnelling, Tor
* password manager with integrated web server
* electronic wallet (e.g. pocket Bitcoin wallet)
* authentication token
* portable penetration testing platform
* low level USB security testing

This section is meant to track available software PoC, projects and/or
procedures oriented towards implementing such application ideas and any other
interesting USB armory usage. If you have tested an interesting use case for
the USB armory please submit it at info@inversepath.com for inclusion.

See also [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication)
for interfacing options.

### File encryption

The [INTERLOCK](https://github.com/inversepath/interlock) application is a file encryption front-end developed, but not limited to, usage with the USB armory.

### Password manager

A PoC from [Michael Weissbacher](http://mweissbacher.com) is available at [https://github.com/mweissbacher/armory-pass](https://github.com/mweissbacher/armory-pass).

### Bitcoin wallet

The Electrum (https://electrum.org/) Bitcoin wallet works out of the box on the
USB armory, it has been tested with X11 forwarding from Linux as well as
Windows hosts.


### Tor Anonymizing Middlebox

Assumptions:
* host main interface on network 192.168.1.0/24 with default gateway 192.168.1.1
* USB armory on network 10.0.0.0/24 with IP address 10.0.0.1
* USB armory masqueraded and routed as described in [Host communication](https://github.com/inversepath/usbarmory/wiki/Host-communication)

#### USB armory setup

Launch Tor with the following configuration (/etc/tor/torrc):

```
VirtualAddrNetworkIPv4 10.192.0.0/10
AutomapHostsOnResolve 1

# Transparent proxy
TransPort 9040
TransListenAddress 127.0.0.1
TransListenAddress 10.0.0.1

# DNS
DNSPort 53
DNSListenAddress 127.0.0.1
DNSListenAddress 10.0.0.1
```


Launch the following script (based on
[Transparent Proxy Tor wiki page](https://trac.torproject.org/projects/tor/wiki/doc/TransparentProxy), *IMPORTANT*: change the _tor_uid variable accordingly):

```
#!/bin/sh

### set variables
#destinations you don't want routed through Tor
_non_tor="10.0.0.0/24"

#the UID that Tor runs as (varies from system to system)
_tor_uid="104"

#Tor's TransPort
_trans_port="9040"

#your internal interface
_int_if="usb0"

### flush iptables
iptables -F
iptables -t nat -F

### set iptables *nat
iptables -t nat -A OUTPUT -o lo -j RETURN
iptables -t nat -A OUTPUT -m owner --uid-owner $_tor_uid -j RETURN
iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 53

#allow clearnet access for hosts in $_non_tor
for _clearnet in $_non_tor; do
   iptables -t nat -A OUTPUT -d $_clearnet -j RETURN
   iptables -t nat -A PREROUTING -i $_int_if -d $_clearnet -j RETURN
done

#redirect all other pre-routing and output to Tor
iptables -t nat -A OUTPUT -p tcp --syn -j REDIRECT --to-ports $_trans_port
iptables -t nat -A PREROUTING -i $_int_if -p udp --dport 53 -j REDIRECT --to-ports 53
iptables -t nat -A PREROUTING -i $_int_if -p tcp --syn -j REDIRECT --to-ports $_trans_port

### set iptables *filter
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#allow clearnet access for hosts in $_non_tor
for _clearnet in $_non_tor 127.0.0.0/8; do
 iptables -A OUTPUT -d $_clearnet -j ACCEPT
done

#allow only Tor output
iptables -A OUTPUT -m owner --uid-owner $_tor_uid -j ACCEPT
iptables -A OUTPUT -j REJECT
```



#### Host setup

Define the "rt_usbarmory" routing table identifier in /etc/iproute2/rt_tables:

```
#
# reserved values
#
255     local
254     main
253     default
0       unspec
#
# local
#
#1      inr.ruhep
1 rt_usbarmory
```

Launch the following commands:

```
# ip rule add from 10.0.0.1/32 table rt_usbarmory
# ip route add default via 192.168.1.1 table rt_usbarmory
# ip route del default
# ip route add default via 10.0.0.1
```

A successful setup can be tested as follows:

```
$ curl https://check.torproject.org | grep -E "Sorry|Congratulations"
      Congratulations. This browser is configured to use Tor.
```