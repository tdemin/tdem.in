---
title: "Simple Mi Fit Login Issue Fix"
date: 2018-06-01T12:00:00+00:00
draft: false
tags: ["hack", "Mi Band"]
---

Some people experience issues with their Mi Bands: Mi Fit, the official Xiaomi's app for wearable electronics like their wristbands, simply doesn't login into the Mi Account, rendering you unable to set up your Mi Band.

<!--more-->

![](/img/mi-fit-play-store-comment.png)

This happened to me as well, though I've managed to work it through.

Apparently the app doesn't work because of the mismatch between your system's display language and the country you're trying to logging in from. Routing the app's traffic through an US-located server fixed the issue for me, and I could finally login.

Hope this helps to someone else wondering what to do about it.
