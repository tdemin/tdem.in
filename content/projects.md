---
title: Projects
layout: static
---

My code:

* [gmnhg][gmnhg], a Go tool that converts your [Hugo][hugo] site to
  [Gemini][gemini]
* [md2gmn][gmnhg], a Go tool that converts Markdown to Gemtext
* [Find Latest Tag][flt], a GitHub Action to find latest tag of any Git
  repo on the internet
* [akm][akm], an [age(1)][age] key manager and wrapper in bash

I maintain the following Docker images:

* [`ghcr.io/tdemin/docker-satellite`][docker-satellite], nightly builds
  of the [Satellite][satellite] Gemini server
* [`ghcr.io/tdemin/gmnhg`][gmnhg], builds of `gmnhg` along with `md2gmn`
  for use in CI pipelines

I sometimes contribute to [nixpkgs][nixpkgs]. I used to maintain a
number of packages on AUR, the Arch Linux user repository, as
[@tdemin][aur-tdemin]:

* [airsonic-advanced][airsonic-advanced], the self-hostable music
  streaming software
* ...and packages for self-maintained software mentioned above/below,
  see respective links

Historical code (mostly migrated from the now-defunct `git.tdem.in`):

* [librespot-builds][librespot-builds], fresh Librespot builds available
  as GitHub releases
* [i3-gaps-desktop][i3-gd], a fork of i3 window manager that supports
  desktop managers ([AUR package][i3-gdaur])
* [syg_go][syg_go], an Yggdrasil address miner written in Go ([AUR
  package][sgaur])
* [opmodbus][opmodbus], an optimizing Modbus master library written in
  Go
* [scarlet_export][scarlet_export], a Python program to export notes
  from Scarlet Notes
* [backup][backup], a simple backup script in bash
* Project Amber, a task list app with variable task nesting:
    + [Amber][amber], the API server in Python/Flask/SQLAlchemy
    + [Amber Web][amber_web], the web client built with TypeScript and
      React
    + Amber CLI, the CLI client written in Go, unfinished

Miscellaneous projects:

* [Guidebook on computer networking][gon] with Linux for Ufa State
  Petroleum Technological University (in Russian).

[nixpkgs]: https://github.com/NixOS/nixpkgs/commits?author=tdemin
[age]: https://age-encryption.org/
[aur-tdemin]: https://aur.archlinux.org/account/tdemin
[librespot-builds]: https://github.com/tdemin/librespot-builds
[docker-satellite]: https://github.com/tdemin/docker-satellite
[satellite]: https://git.sr.ht/~gsthnz/satellite
[opmodbus]: https://github.com/tdemin/opmodbus
[flt]: https://github.com/marketplace/actions/find-latest-tag-of-git-repository
[akm]: https://github.com/tdemin/akm
[amber]: https://github.com/tdemin/amber
[amber_web]: https://github.com/tdemin/amber_web
[amber_cli]: https://github.com/tdemin/amber_cli
[syg_go]: https://github.com/tdemin/syg_go
[syg_go-aur]: https://aur.archlinux.org/packages/syg_go
[sgaur]: https://aur.archlinux.org/packages/syg_go/
[scarlet_export]: https://github.com/tdemin/scarlet_export
[backup]: https://github.com/tdemin/backup
[emdl]: https://aur.archlinux.org/packages/emdl/
[ygg]: https://yggdrasil-network.github.io
[gmnhg]: https://github.com/tdemin/gmnhg
[hugo]: https://gohugo.io
[gemini]: https://gemini.circumlunar.space/
[airsonic-advanced]: https://aur.archlinux.org/packages/airsonic-advanced-bin/
[i3-gd]: https://github.com/tdemin/i3
[i3-gdaur]: https://aur.archlinux.org/packages/i3-gaps-desktop/
[gon]: /files/guidebook_networking.pdf
