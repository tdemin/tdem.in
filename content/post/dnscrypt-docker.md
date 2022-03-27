---
title: Making dnscrypt-proxy and Docker play well together
date: 2020-08-03T17:16:37+05:00
tags: [Docker, dnscrypt-proxy, DNS, Linux, systemd, systemd-resolved]
---

This post is available in Russian [here][orig].

[orig]: https://habr.com/en/post/496142/

DNSCrypt is a fairly popular way of protecting DNS traffic that is usually left
unencrypted from other people. [dnscrypt-proxy][dnscrypt-proxy], a client
program that implements DNSCrypt, also supports the DNS-over-HTTPS protocol,
allowing name resolution over DoH.

[dnscrypt-proxy]: https://github.com/DNSCrypt/dnscrypt-proxy

Unfortunately, leaving dnscrypt-proxy with its default settings while setting it
as the default resolver breaks name resolution in Docker containers. Fixing this
while not exposing a DNS resolver on the LAN is what's described below.

<!--more-->

This post in short:

1. Create a dummy adapter, assign a private network IP to it.
2. Make dnscrypt-proxy listen on this interface, change system DNS settings
   accordingly.

## The problem

Docker sets up DNS in containers by [copying host DNS settings][DockerDNS],
reusing `/etc/resolv.conf` from the host.

dnscrypt-proxy and other custom resolvers usually bind to `127.0.0.1:53`; the
corresponding `/etc/resolv.conf` entry is `nameserver 127.0.0.1`. This setting
is propagated to the containers, but, as the containers belong to a different
network namespace, host's `127.0.0.1` and container's mean two different things.

[DockerDNS]: https://docs.docker.com/config/containers/container-networking/#dns-services

## The fix

The easiest solution is to run dnscrypt-proxy on host's public IP address and
then add this address to `/etc/resolv.conf`. This means we expose a DNS
resolver to the network, and we'd rather not.

Instead we'll create a `dummy` network adapter that is routable yet doesn't
actually send any packets; it is routable from the container since Docker
containers use the host machine as their default gateway.

### Creating the adapter

If `dummy` kernel module is not loaded yet (`% lsmod | grep dummy` displays
nothing), load it and enable its autostart:

```
# modprobe dummy
# echo "dummy" >> /etc/modules-load.d/net_dummy.conf
```

Creating and setting up a dummy adapter is as simple as running these two
commands on any modern Linux system with iproute2 installed:

```
# ip link add type dummy name dummy0
# ip addr add dev dummy0 10.0.197.1/24
```

Making this permanent will vary between network configuration software. With
systemd-networkd you'll need two config files:

`/etc/systemd/network/50-dummy0.netdev`:

```
[NetDev]
Name=dummy0
Kind=dummy
Description=Dummy network for dnscrypt-proxy
```

`/etc/systemd/network/50-dummy0.network`:

```
[Match]
Name=dummy0

[Network]
DHCP=no
Address=10.0.197.1/24
DefaultRouteOnDevice=false
```

### Changing DNS settings

To bind dnscrypt-proxy to a new address, edit `listen_addresses` in
`/etc/dnscrypt-proxy/dnscrypt-proxy.toml` and make it look like this:

```
listen_addresses = ['127.0.0.1:53', '[::1]:53', '10.0.197.1:53']
```

Restart dnscrypt-proxy and then replace the text in `/etc/resolv.conf`
(or wherever your network configurator stores DNS settings) with this:

```
nameserver 10.0.197.1
```

### Checking

Run a new container:

```
% docker run -it --rm alpine:3.12
# cat /etc/resolv.conf
nameserver 10.0.197.1
# ping -c 1 ya.ru
```

## Additional info

If you use a firewall (and you should be), then allow incoming traffic to
`10.0.197.1:53` from the subnets Docker uses for containers.

If you use systemd-resolved as your caching resolver of choice with
dnscrypt-proxy set as its upstream, then you're still fine even though
systemd-resolved won't allow you to listen on anything besides `127.0.0.53`:
[Docker detects the use of systemd-resolved][DockerResolved] and copies
`/run/systemd/resolve/resolv.conf`, which is generated from resolved settings,
instead of using `/etc/resolv.conf`.

[DockerResolved]: https://github.com/moby/libnetwork/blob/master/resolvconf/resolvconf.go

