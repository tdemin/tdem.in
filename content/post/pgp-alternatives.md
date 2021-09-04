---
title: "GnuPG and its alternatives: the state in 2021"
date: 2021-07-26T02:00:00+05:00
draft: false
tags: ["PGP", "GnuPG", "age", "minisign", "signify", "sigtool", "encryption", "cryptography", "privacy"]
---

The market of the software to replace GnuPG and the OpenPGP infrastructure
appears to be quite a topic on itself, the precondition to replace GnuPG being
its complexity (which has gone to levels so high the official library to
interface with GnuPG, [gpgme][gpgme], is literally a command-line
wrapper[^1] to GnuPG), it being the de-facto implementation of
OpenPGP, and its other problems, as commonly illustrated by [this post by
Latacora][latacora].

[^1]: See section 3.1 of the [GPGME reference][gpgmereference] and others: a large number of functions, like `gpgme_get_dirinfo`, only make sense if GPGME was a command-line wrapper.

<!--more-->

[gpgme]: https://gnupg.org/software/gpgme/index.html

Theoretically, the program (or programs, thinking the Unix way) to replace GnuPG
as a cryptography software suite should do these two things at the very minimum
(GnuPG is capable of a lot more, including keyservers, Web of Trust, etc, but
for simplicity let's talk about the bare minimum):

+ encrypt / decrypt data using (a)symmetric cryptography;
+ generate digital signatures for data and be able to verify the generated
signatures with public keys.

## Encryption

One of the most well-known tools to do encryption is [age][age], which, as of
July 2021, isn't out of release candidates just yet. age doesn't even raise many
concerns on its own, but it lacks even the simplest public/private key
management capabilities (like key aliases or "use this private key as the
default").

It goes along well with the Unix way (one program per job to do it well), but
can you seriously imagine an encryption tool where you provide your full
recipient public key every time with a command-line key? Me neither. `age`
seriously assumes you're fine with entering something like this every time[^2]:

```
% age -r age1h0ea8x07z33fc5lzkqj3kghup4g6jkzsaj6v86pgngwn7gs8q9es56lpwv \
    -o file.enc file.txt 
```

[^2]: To be completely fair, you can more or less solve it with a wrapper. For this job I made [akm][akm], a shell script wrapper that will read recipient keys for you from `~/.akm` and pass them to `age`. It can also set your default encryption key for you unless you specify one on the command line.

`age` also raises concerns by letting trivially discoverable bugs such as [this
one][age-bug] slip in the release candidates. This one is for sure easily
discoverable, as I ran into it while writing [akm][akm] in my first hours of
trying `age`, and its presence seriously raised my eyebrows, as it's been one of
the tools [recommended][latacora] by supposedly competent cryptographers at
Latacora long before the bug has been fixed.

[reop][reop] can also do encryption for you, it will be discussed later.

[age]: https://age-encryption.org "Actually Good Encryption"
[age-bug]: https://github.com/FiloSottile/age/issues/263

## Digital signatures

With digital signatures and the tools to verify them we have quite a number of
options, with different problems / concerns surrounding them (remember, we want
our tool to be at least somewhat reputable):

* The already mentioned [reop][reop] is quite good on itself, as to be expected
from the people around the OpenBSD project, but appears to be abandoned by its
author, and so does [its portable version][reop-portable] for other operating
systems.
* [signify][signify]'s been made by the same person as reop, Ted Unangst, but is
only available under OpenBSD, and Linux folks are supposed to enjoy one of the
many available source ports ([1][signify1], [2][signify2], etc). Which of those
"sucks less" and will actively be maintained? Hard to tell. The quality of ports
of doas, neither of which remember passwords under Linux, doesn't enthuse much.
* There's a portable alternative to signify called [minisign][minisign] by the
author of libsodium, which is written in C. There's little concerns about
it, aside from the only random commit I noticed which got [unrelated changes
slipped into it][minisign-commit] (well, Git has revision history management
tools for a reason?).
* There's also [sigtool][sigtool], which for whatever reason has gone with YAML
for its signature format and isn't really easy to stumble upon, thanks in part
to its name colliding with a [ClamAV tool of the same name][sigtool-clamav]. The
number of its users also leaves to be desired (remember, reputability).

[signify]: https://man.openbsd.org/signify.1
[signify1]: https://github.com/aperezdc/signify
[signify2]: https://github.com/leahneukirchen/outils/tree/master/src/usr.bin/signify
[minisign]: https://github.com/jedisct1/minisign
[minisign-commit]: https://github.com/jedisct1/minisign/commit/0137cd75af4b1188cb01385724fd3c3c1fc2b4ba
[sigtool]: https://github.com/opencoff/sigtool
[sigtool-clamav]: https://www.systutorials.com/docs/linux/man/1-sigtool/
[reop]: https://flak.tedunangst.com/post/reop "Reasonable Expectation of Privacy"
[reop-portable]: https://github.com/tedu/reop/

## Other problems

While out of these described above I have already gone with a [pair of
age/minisign keys][keys] for testing, there's an inherent problem of having them
integrated in my workflow. For instance, while you can sign commit/tag data with
Git with GnuPG, there's no way to sign those with the [current set of Git hooks
available][git-hooks] (although there are [hacks to do so][git-signify] using
current Git GnuPG support).

Other tools, like NeoMutt, also appear to lack support for arbitrary digital
signature / encryption tools unless you're fine with hacking / gluing them
together in ugly ways.

[git-hooks]: https://git-scm.com/docs/githooks
[git-signify]: https://seankhliao.com/blog/12020-10-31-git-signing-signify/

## Conclusion

Unless Git and other software projects start abandoning GnuPG, GnuPG and OpenPGP
are unlikely to go anywhere, as being tied too deep into software codebases and
seeing no perfect simple alternatives just yet.

There's also a new OpenPGP library in Rust, [Sequoia][sequoia], which solves a
number of issues associated with GnuPG (namely the monolith problem), but it
doesn't look like any of the applications using GnuPG integrate with it just
yet.

Anyways, you can now encrypt me messages with age (or [akm][akm]), and verify my
signatures with minisign with those [keys][keys]:

```
age: age1h0ea8x07z33fc5lzkqj3kghup4g6jkzsaj6v86pgngwn7gs8q9es56lpwv
minisign: RWRltlKLStovfiGdhWNzla+GyANAL9ok1Bg15qCAq8oRPCGN6G4fjLj1
```

[latacora]: https://latacora.micro.blog/2019/07/16/the-pgp-problem.html "The PGP problem"
[akm]: https://github.com/tdemin/akm "age key manager"
[keys]: /announcements/#2021-05-27-age-%2f-minisign-public-keys
[gpgmereference]: https://gnupg.org/documentation/manuals/gpgme.pdf
[sequoia]: https://sequoia-pgp.org/
