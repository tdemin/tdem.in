---
title: "Running a Gemini site with gmnhg and Hugo"
date: 2020-11-26T03:45:04+05:00
tags: ["gemini", "Go", "gmnhg"]
---

This site has recently got its [Gemini][gemini] version thanks to the
tool I wrote, [gmnhg][gmnhg], available at `gemini://tdem.in`. The tool
has already met its first milestone, [v0.1.0][gmnhg010], so this is
likely a good moment to recap on the whole idea of working in a Hugo
site into the Geminispace.

[gemini]: https://gemini.circumlunar.space
[gmnhg]: https://git.tdem.in/tdemin/gmnhg
[gmnhg010]: https://git.tdem.in/tdemin/gmnhg/releases/tag/v0.1.0

<!--more-->

Markdown, even though being quite complex as the [CommonMark][cmark]
specification shows (the CommonMark spec was originally designed to
clean up the ambiguities of the original 2004 spec that resulted in
quirky behaviour across various renderers), is simple enough to be
converted to reasonable Gemini text (named Gemtext here) if you only use
a subset of it. This means source Markdown can be converted to native
Gemtext content just fine as long as some editor discretion is used when
running a Gemini-friendly blog.

[cmark]: https://commonmark.org

As most simple Markdown formatting can be converted to Gemtext using
either the [tools Gemtext provides][GemtextSpec] or just left unmodified
as plain text, it can be used as a Gemtext source, with following
exceptions:

[GemtextSpec]: https://gemini.circumlunar.space/docs/specification.html

* links or any formatting that requires a whole new line for it are not
    used in lists;
* text formatting cannot be used inside link descriptions;
* absolutely no inline HTML can be included in the document.

Testing your Markdown is made simple with `md2gmn`, another tool I wrote
that is distributed alongside `gmnhg` that only renders Markdown text to
Gemtext, to see if any of your Markdown fails to properly render to the
medium Gemini provides.

`gmnhg`, on the other hand, does the uplifting of processing your Hugo
content files into an output dir suitable for being served by a Gemini
server such as [satellite][satellite] while also taking care of
generating index pages and such. The resulting Gemtext is then passed to
Go's `text/template`, and you can customize the templates by adding the
ones relevant to you to the `gmnhg/` dir. The algorithm is already
summarized quite well in its [godoc page][gmnhggodoc], so there's no
real point in going over it again.

[satellite]: https://sr.ht/~gsthnz/satellite/
[gmnhggodoc]: https://pkg.go.dev/git.tdem.in/tdemin/gmnhg/cmd/gmnhg

![Markdown site converted to Gemini, index page opened in Amfora](/img/gmnhg_2020-11-20.png)

`gmnhg` respects stuff you'd probably expect a Hugo companion to obey:
it takes care about copying static files to the output dir, it won't
render draft posts, etc; yet, `gmnhg` at this time doesn't support much
of the rich functionality Hugo provides to your blog (taxonomies, etc),
but, to be fair, a lot of it is complex enough to not fit well in a
simple Gemini blog. Yet, I am completely open to patches that implement
certain Hugo stuff that works fine with Gemini.

This site's sources can [serve as a reference][blog] for those who need
an example on how to use gmnhg to serve their site to Gemini.

[blog]: https://git.tdem.in/tdemin/blog.tdem.in

Gemini is too escapist / privacy-focused in the tools it provides (and I
still think mandatory TLS in a braindead-simple protocol is a dumb
idea), but no one has come up with a decent enough of a Web replacement,
so I guess this blog will stay available on Gemini (and having coded a
few tools for its support counts for enough effort to make leaving
Gemini retrospectively costly).  This new medium is simple and
hacker-friendly, so I advise you to explore it as well!
