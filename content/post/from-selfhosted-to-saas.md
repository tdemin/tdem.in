---
title: "From Self-Hosted FOSS to Proprietary PaaS: A Migration Story"
date: 2024-01-27T17:01:00+06:00
tags: [SaaS, iCloud, Nextcloud, Tiny Tiny RSS, TTRSS, GitHub, Gitea, Git, Gemini, gmnhg, Satellite, NetNewsWire, CalDAV, CardDAV, Cloudflare]
---

First time to blog in three years!

As a person too busy with `${DAYJOB}` the last few years, I wanted my
remaining servers hosting groupware, file sharing, and news reading
software to ideally manage themselves. 12 EUR/mo doesn't mean you get a
system administrator to do chores for you, instead you get to do the
routines yourself. That could only continue for as much before we would
get fed up and attempt the move back to being truly serverless™, which
is what the post you're reading is all about.

No servers for us would mean:

* getting a smaller monthly bill thankfully to most personal cloud
  options costing less than a six EUR/mo VPS
* doing nothing whenever another Debian / Red Hat / `$DISTRONAME`
  Security Advisory reports of a critical vulnerability
* having nothing to maintain or update (which is a serious win,
  considering `${DAYJOB}` involves enough system maintenance by itself)

<!-- more -->

## Planning the move

Our self-hosted service inventory to migrate contains the following:

* [Gitea][gitea], a Git repository hosting in Go
* [Nextcloud][nextcloud], a groupware suite backend server that works as
  a private cloud, in PHP
* [Tiny Tiny RSS][ttrss], an RSS feed reader and aggregator in PHP
* [Satellite][satellite], a Gemini server that only serves static content, in Go
* (if you count this one as a service, that is) nginx, a web server
  hosting [https://tdem.in][tdem_in], aka the website you're reading
  this at, that also needs to serve a few occasional redirects, like
  `music.tdem.in` to the [Last.fm page][lastfm]

[gitea]: https://gitea.io
[nextcloud]: https://nextcloud.com
[ttrss]: https://tt-rss.org/
[satellite]: https://git.sr.ht/~gsthnz/satellite
[tdem_in]: https://tdem.in
[lastfm]: https://www.last.fm/user/l0548942

The hardest challenge to solve with migrating off these three would be
keeping exactly zero servers at hand. This would mean finding some other
way to handle HTTP redirects from domains previously served at
`git.tdem.in`, `storage.tdem.in`, and others, which a simple CNAME
record doesn't handle. Gladly for us, Cloudflare provides redirects as a
feature of its web edge for free (if you're fine with only having up to
10 of those), and the sole requirements to use them is to have
Cloudflare handle DNS for your domain (which was already the case for
`tdem.in`, and proved useful).

Finding a replacement to `git.tdem.in` was easy: GitHub was long the
home to most of our public code. TTRSS would have to go in favor of
something that runs locally, like [NetNewsWire][netnewswire]. Nextcloud
has a pretty obvious alternative for Apple users.

[netnewswire]: https://netnewswire.blog/

As long as we were fine with stopping hosting the Gemini capsule (which
was bound to happen at some point due to reducing interest and lack of
energy to maintain [gmnhg][gmnhg]) and agreeing to the status quo of
this website being our only home page, Satellite needed no replacement.

[gmnhg]: https://github.com/tdemin/gmnhg

Therefore, our final listing of free or cheaper services to use for
syncing consists of the following:

* iCloud to handle files / calendar / contacts / tasks syncing / other
  groupware (only really works for Mac/iPhone users, but we're rocking
  both anyway, so why not)
* GitHub to host our ever-shrinking set of public projects
* GitHub Pages to host this website
* NetNewsWire to read RSS feeds locally and sync subscriptions with the
  built-in iCloud support
* none for Gemini hosting, which was bound to be gone at some point
  anyway due to lack of energy to maintain [gmnhg][gmnhg]
* Cloudflare to handle redirects

## Cloudflare as the ultimate redirector

Performing redirects from a hostname to elsewhere is doable with
Cloudflare in two simple steps:

1. Creating a CNAME record pointing anywhere (we use `@` as the target
   to prevent any unwanted side effects). The important bit is that the
   "Proxied by Cloudflare" toggle needs to be ON, so the traffic reaches
   the Cloudflare edge.

![CNAME records at Cloudflare](/img/cloudflare_cnames.png)

What actually ends up on public DNS looks like this:

![CNAME records, transformed by Cloudflare into A records onto its edge](/img/cloudflare_cnames_transformed.png)

2. Creating a dynamic redirection rule that responds with 302 whenever
   traffic matching your hostname reaches Cloudflare:

![Cloudflare dynamic redirection rule matcher](/img/cloudflare_dynamic_redirect_matcher.png)
![Cloudflare rule target, responding with a 302](/img/cloudflare_dynamic_redirect_target.png)

The end result is that our traffic ends up where destined:

![Cloudflare redirects git.tdem.in to GitHub, as advertised](/img/cloudflare_redirect_git.tdem.in.png)

## Gitea

Relatively easy to migrate, as long as most of the repos are private,
feature no CI/CD, no content to migrate besides code, etc. The list of
repos is trivially obtainable through `<profile> -> Admin Panel ->
Content -> Repositories`.

This small snippet automates most of the routine, except creating the
repository at GitHub:

```sh
git_mirror () {
    git clone --mirror "https://git.tdem.in/${1}.git" $1
    cd $1
    git remote add github "https://github.com/${1}.git"
    git push -u github --all
    git push github --tags
    cd -
}
```

Invoking it, as repositories at GitHub get created and reconfigured to
be private and contain no wiki / Actions / etc, is as easy as:

```
git_mirror tdemin/repository1
git_mirror tdemin/repository2
```

> Memo: Gitea uses the same wiki Markdown hosting scheme as GitHub, with
> `${REPOSITORY_NAME}.wiki.git` [being the URL][github_wiki] for the Git
> repository containing all of your pages. Migrating that would simply
> mean an extra `git_mirror`.

[github_wiki]: https://docs.github.com/en/communities/documenting-your-project-with-wikis/adding-or-editing-wiki-pages#adding-or-editing-wiki-pages-locally

The last bit remaining would redirecting everyone[^1] hitting
`git.tdem.in` from history or links on the public web, which is
configurable with Cloudflare as shown above.

[^1]: This means pages like links to commits, issues, etc, are likely to
    break, since GitHub might not necessarily do the URI scheme Gitea's
    web UI does. This also includes Google Search, as Gitea's
    workarounds to get non-canonical pages (random commit pages in
    random languages, etc) out of search engine indexes are imperfect to
    this day. YMMV.

## Nextcloud

The amount of time spent on migrating from a groupware software piece to
another directly depends on the volumes of information stored inside the
former. The scenario for `storage.tdem.in` included the following, for
one user:

* calendar events
* reminders (technically fancy calendar events)
* contacts
* files

Files are taken care of by copying them to the iCloud drive folder. To
make syncing `~/Desktop` and `~/Documents` seamless like Nextcloud
offers OOB[^2] you might want to enable the `Desktop & Documents
folders` in [iCloud sync settings][icloud_desktop]. As an added bonus,
you get file URL sharing directly from Finder.

[icloud_desktop]: https://support.apple.com/en-us/HT206985

[^2]: Out-of-the-box

![That is, until it works](/img/icloud_web_crash.png)

Calendar events are trivially moved to the relevant calendar in two
clicks per event. While sounding like a tedious job, that's doable in a
few minutes even for the largest agendas thanks to the built-in snappy
macOS app. Reminders get moved into iCloud-hosted lists with the help of
mass drag-and-drop (unless you care about lots of historically completed
entries. I do not).

![As easy as it gets with the right app for the job](/img/icloud_calendar.png)

> There's an option to extract your calendar entries through CalDAV and
> then import it using https://caldav.icloud.com, but I didn't care
> enough to do that. The events in my calendar were trivial enough to
> click through to move to iCloud-provided calendars. The Apple's
> calendar app is friction-free enough.

Contacts proved to be a simple task for a mass vCard export/import. A
few records needed fixing after the import, but that's a five to ten
minute-job even for a contact book of hundreds of entries.

## Tiny Tiny RSS

There's no real data export option other than using OPML, but that one
just gets imported into your client of choice, and you're done. I
recommend [NetNewsWire][netnewswire] as a brilliant macOS / iOS
implementation that can also sync feeds across devices with iCloud, but
the choice is ultimately up to you.

You might still want to keep the dump of your TTRSS database for
archival purposes.

![That's... quite a number of entries to get rid of. :(](/img/ttrss_entries_count.png)

## GitHub Pages

Migrating an existing Hugo blog with a well-defined CI setup mostly
concludes to following the instructions for [setting up GitHub
Pages][ghp_quickstart] and then [doing the routine][ghp_custom_domain]
to get it work with a custom domain. My sole nitpick was that GitHub
Pages can be served from any domain, not just `<username>.github.io`,
which helped with keeping the [repository][blog_repo] in its canonical
place.

[ghp_quickstart]: https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site
[ghp_custom_domain]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site

In the end, the whole chore concluded to the following GitHub Actions
workflow [diff][ghp_diff]:

[ghp_diff]: https://github.com/tdemin/tdem.in/commit/0453579bfd53b47d77d333688e7fbfd62f552dfc

```diff
-      - name: Build Gemini site with gmnhg
-        uses: docker://ghcr.io/tdemin/gmnhg:v0.4.2
+      - name: Deploy to GitHub Pages branch
+        uses: peaceiris/actions-gh-pages@373f7f263a76c20808c831209c920827a82a2847
         with:
-          args: gmnhg -output ./gmnhg
-      - name: Deploy web site
-        # v6.0.0, strict commit pinning due to handling secrets
-        uses: Burnett01/rsync-deployments@45d84ad5f6c174f3e0ffc50e9060a9666d09c16e
-        with:
-          switches: -avzr --delete
-          path: hugo/
-          remote_path: ${{ secrets.DEPLOY_ROOT_PATH }}/blog.tdem.in
-          remote_host: ${{ secrets.DEPLOY_HOST }}
-          remote_user: ${{ secrets.DEPLOY_USER }}
-          remote_key: ${{ secrets.DEPLOY_KEY }}
-      - name: Deploy Gemini site
-        # GitHub Actions doesn't allow anchors in workflow YAML, so
-        # duplicating this is required
-        uses: Burnett01/rsync-deployments@45d84ad5f6c174f3e0ffc50e9060a9666d09c16e
-        with:
-          switches: -avzr --delete
-          path: gmnhg/
-          remote_path: ${{ secrets.DEPLOY_ROOT_PATH }}/gemini
-          remote_host: ${{ secrets.DEPLOY_HOST }}
-          remote_user: ${{ secrets.DEPLOY_USER }}
-          remote_key: ${{ secrets.DEPLOY_KEY }}
+          github_token: ${{ secrets.GITHUB_TOKEN }}
+          publish_dir: ./hugo
+          cname: tdem.in
```

Setting up redirects for both `www.tdem.in` and `blog.tdem.in` as
outlined in the Cloudflare chapter above helps for the remainders of
`blog.tdem.in` on search engine databases.

> As `tdemin.github.io` is now served from [this repository][blog_repo],
> this required importing content we might have had in there. Gladly,
> this only meant [copying a single blog post][vpn_translated] for the
> archive.

[blog_repo]: https://github.com/tdemin/tdem.in
[vpn_translated]: https://github.com/tdemin/tdem.in/commit/d83927d3f1e37ff709f9d0e92d4bd5b27abf8989

## Conclusions

Whether or not we won or lost more by migrating depends on the
priorities. Pros are as follows:

* less costly monthly bill (60% of the original costs after one adds the
  VPN server costs, that of two 6 EUR/mo VPS instances)
* virtually no maintenance required
* better groupware OS integration for Apple devices, where syncing
  reminders whenever they change/get added just works™ and
  doesn't get stuck from time to time which is what occasionally
  happened on both latest iOS 17 and macOS 14 with Nextcloud CalDAV
  entries installed with a configuration profile

What we lost:

* the ability to have a background always-on RSS updater that would
  ensure we lose no entries in-between connections (which is important
  for high-volume collective blogs and news outlets)
* freedom in picking groupware client software, which never really
  existed on Apple hardware anyway

I stand by the option that was a net positive.
