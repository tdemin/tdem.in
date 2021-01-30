---
title: "Using Restic with systemd on Linux"
date: 2021-01-29T20:30:00+05:00
draft: false
tags: ["restic", "systemd", "Linux"]
---

[Restic][restic], the simple backup program, is a fairly well-known piece of
software. Designed to be simple to both use and script on any system, it doesn't
include any OS-specific setup examples, which is precisely what this post
describes.

So what we're trying to achieve here:

1. Automated backup runs daily at 12:00 AM (or any configurable time).
2. The backup includes only important configuration files and data stores.
3. The backup also includes all PostgreSQL databases, restorable with `psql -f`.
4. The backup expands to an unlimited number of repos on need.

<!--more-->

This guide assumes we're backing up to a [rest-server][rest-server] instance
running at `192.168.1.200`. The configuration should still be trivial to adapt
to pretty much any storage provider. This guide also assumes you have already
initialized a rest-server repo with
`restic init -r rest:http://192.168.1.200/your-repo/`.

For this we will need two systemd services, two correspondent timers, and a
single helper script.

## Backing up files/directories

We'd rather not want to run restic as root for obvious reasons, so let's create
a dedicated user:

```
# sudo useradd -m -N -s /usr/sbin/nologin restic
```

To backup files, we'll need this service along with its timer:

`/etc/systemd/system/restic@.service`:

```
[Unit]
# this unit can be activated with a parameter, e.g. in
#   systemctl start restic@your-repo.service
# %I is "your-repo"
Description=Restic backup on %I
After=syslog.target
After=network-online.target

[Service]
Type=oneshot
User=restic
# runs restic backup on the files listed in /etc/restic/your-repo.files
ExecStart=/usr/local/bin/restic backup --files-from /etc/restic/%I.files
# source repo and password from /etc/restic/your-repo.env
EnvironmentFile=/etc/restic/%I.env
AmbientCapabilities=CAP_DAC_READ_SEARCH

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/restic@.timer`:

```
[Unit]
# the timer, enabled as restic@your-repo.timer, will trigger
# restic@your-repo.service
Description=Run Restic at 12:00 AM

[Timer]
OnCalendar=*-*-* 0:00:00

[Install]
WantedBy=timers.target
```

The repo name is passed through an environment file in `/etc/restic`, read by
systemd (systemd does this as root, and `/etc/restic` should really be only
readable as root, so set permissions accordingly):

`/etc/restic/your-repo.env`

```
RESTIC_PASSWORD=your_repo_password
RESTIC_REPOSITORY=rest:http://192.168.1.200/your-repo/
```

We also have to supply restic with a file/directory list to back up:

`/etc/restic/your-repo.files`

```
/var/lib/docker
/etc/postgresql
/etc/restic
...
```

## Backing up databases

This one is just a little less trivial.

Restic supports backing up data provided through stdin, so we can feed it with
the output of `pg_dumpall`. The only limitation is that systemd runs whatever
you specify in `ExecStart` with `execve(3)`, and, to use output redirection,
we'll need a separate bash script:

`/usr/local/bin/pgdump.sh`:

```
#!/usr/bin/env bash

set -euo pipefail

/usr/bin/sudo -u postgres /usr/bin/pg_dumpall --clean \
    | gzip --rsyncable \
    | /usr/local/bin/restic backup --host $1 --stdin \
        --stdin-filename postgres-$1.sql.gz
```

To run `pg_dumpall` as `postgres`, we'll need the following sudo rule in
`/etc/sudoers`:

```
restic ALL=(postgres) NOPASSWD:/usr/bin/pg_dumpall --clean
```

The unit file is as trivial as this:

`/etc/systemd/system/restic-pg@.service`:

```
[Unit]
Description=Restic PostgreSQL backup on %I
After=syslog.target
After=network-online.target
After=postgresql.service
Requires=postgresql.service

[Service]
Type=oneshot
User=restic
ExecStart=/usr/local/bin/pgdump.sh %I
EnvironmentFile=/etc/restic/%I.env

[Install]
WantedBy=multi-user.target
```

The timer isn't really any different from the one we've already seen above:

`/etc/systemd/system/restic-pg@.timer`

```
[Unit]
Description=Run Restic on PostgreSQL at 12:00 AM

[Timer]
OnCalendar=*-*-* 0:00:00

[Install]
WantedBy=timers.target
```

## Finishing up

Start the timers and enable their autostart at boot. Remember that `your-repo`
is used to expand file paths in `/etc/restic`:

```
# systemctl enable --now restic@your-repo.timer
# systemctl enable --now restic-pg@your-repo.timer
```

Test whether the whole backup system is working:

```
# systemctl start restic@your-repo.service
# systemctl start restic-pg@your-repo.service
```

These unit files allow backing up to an unlimited number of repos as long as the
relevant configuration is provided through `/etc/restic/repo-name.{env,files}`.

## Links / sources

* [Recipe to backing up PostgreSQL in a Docker container][link1], originally
    used for the PostgreSQL backup script.
* systemd docs: [systemd.service][systemd.service],
    [systemd.timer][systemd.timer].

[restic]: https://restic.net
[rest-server]: https://github.com/restic/rest-server
[link1]: https://forum.restic.net/t/recipe-to-snapshot-postgres-container/1707
[systemd.service]: https://www.freedesktop.org/software/systemd/man/systemd.service.html
[systemd.timer]: https://www.freedesktop.org/software/systemd/man/systemd.timer.html
