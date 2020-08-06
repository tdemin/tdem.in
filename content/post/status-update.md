---
title: "Status Update: July 2020"
date: 2020-08-02T00:52:05+05:00
draft: false
tags: ["status update"]
---

This has been a busy time.

I'm finally finished with my internship at Nefteavtomatika, getting my
hands on an internal web service managing MAC address allocation
designed to provide an API for flashing software. Thanks to that I'm now
a lot more familiar with building web in Go, and am eager to build more
things with it. <3

I've started looking at lot more into [Yggdrasil][ygghome], a mesh
networking software. As of right now I'm mainly using it as a VPN, as it
provides every node with a static /64 subnet, so IPv6 exploration is a
lot easier (as most of Russian ISPs don't even bother with providing
static prefixes to end users). The positive side effects of this are the
new [guide][yggdoc] on configuring it with systemd-networkd and routing,
and a [simple address miner][yggminer] I wrote in Go. If you want a
neatly-looking Yggdrasil address, you now know what to do. :)

The work on Amber continues, this time mainly with
[amber_cli][amber_cli], who has never gotten the love it deserved since
the very beginning of its development and now gets a complete
refactoring as at the time of starting the project I wasn't really
familiar with Go at all. There are also few corrections I'd like to make
to API v0 and related algos before considering it done. [Amber][amber]
itself needs a huge overhaul, so does [Amber Web][amber_web], who's
never seen a single update the last few months.

This blog itself has got an update—it is now hosted on my main domain,
[tdem.in](/), and orphaned links at [blog.tdem.in](https://blog.tdem.in)
now lead here. The home page is now managed by Hugo as well, making the
effort in keeping it updated less of a time waste.

Looking good so far, looking forward to the last month of summer!

![Photo by Safar Safarov on [Unsplash](https://unsplash.com/photos/MSN8TFhJ0is)](/img/coding-xps.jpg)

[ygghome]: https://yggdrasil-network.github.io
[yggdoc]: /post/yggdrasil-systemd/
[yggminer]: https://git.tdem.in/tdemin/syg_go
[amber_cli]: https://git.tdem.in/tdemin/amber_cli
[amber]: https://git.tdem.in/tdemin/amber
[amber_web]: https://git.tdem.in/tdemin/amber_web
