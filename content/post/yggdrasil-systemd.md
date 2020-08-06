---
title: "Configuring Yggdrasil with systemd-networkd"
date: 2020-07-23T13:15:08+05:00
draft: false
tags: ["Yggdrasil", "systemd", "systemd-networkd", "systemd-resolved", "Linux"]
---

[Yggdrasil][ygg], while being a great mesh networking software, doesn't have
that many learning resources on it. The docs on its website and a few
enthusiast-established wikis are probably everything you'll ever find on how to
set it up.

[ygg]: https://yggdrasil-network.github.io

This is a simple recipe on how to configure Yggdrasil with systemd-networkd and
systemd-resolved while providing other devices on your local network with Ygg
addresses and allowing them to use Ygg.

Consider the following setup:

+ an always-on PC
+ a router to which the PC is connected
+ all other devices in LAN

We want to deploy Yggdrasil on the PC and delegate the `300:XXXX:XXXX:XXXX::/64`
subnet provided by Ygg to other devices. This guide assumes you have already
[set up Yggdrasil][yggdoc] on your Linux box.

[yggdoc]: https://yggdrasil-network.github.io/configuration.html

We configure our usual connection with `/etc/systemd/network/10-eth0.network`:

```
[Match]
# wired connection device name
Name=eth0

[Network]
# Address/Gateway, or DHCP=yes, or whatever else you might have configured
# your wired connection with
...
# the address inside the 300::/8 subnet; the host will use this address inside
# the wired network
Address=300:XXXX:XXXX:XXXX::1/64
# enable IPv6 router
IPv6PrefixDelegation=static
IPForward=ipv6
# the DNS we want to use for clearnet connections
DNS=...

[IPv6Prefix]
# the prefix advertised to other devices by the machine
Prefix=300:XXXX:XXXX:XXXX::/64

[IPv6PrefixDelegation]
EmitDNS=yes
# should be an Ygg DNS address, you might prefer to unset this or use your own
# DNS server inside Yggdrasil
DNS=301:2522::53
RouterLifetimeSec=3600 # should always be set

[IPv6RoutePrefix]
# the route to Ygg to propagate to devices
Route=200::/7
```

Now let's configure Yggdrasil TUN device with
`/etc/systemd/network/40-tun0.network`:

```
[Match]
# Yggdrasil TUN device name, same as IfName in /etc/yggdrasil.conf if set
Name=tun0

[Network]
# useless with Yggdrasil
LinkLocalAddressing=no
# some DNS inside Yggdrasil to resolve .ygg addresses and alike; you might want
# to specify your own address here
DNS=301:2522::53
DefaultRouteOnDevice=no

[Address]
# the /128 address Ygg provides us with
Address=200:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX/128

[Route]
# route packets to Yggdrasil
Destination=200::/7
Scope=global
```

In this schema systemd-resolved will only use Ygg DNS for sites we browse from
Ygg, and whatever else we configure in `/etc/systemd/resolved.conf` by default.
This allows the machine to properly resolve `.ygg` domains while falling back
to system default DNS for everything outside Yggdrasil.

Every other IPv6-enabled device on your LAN will now receive Ygg addresses and
will be able to connect to Yggdrasil while your machine is on. However, be aware
that end-to-end encryption Yggdrasil provides is terminated at your Yggdrasil
router. You should also consider setting up a firewall to protect your other
devices.
