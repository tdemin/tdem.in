---
title: "Running restic with AWS S3"
date: 2021-10-22T00:55:00+05:00
draft: false
tags: ["AWS", "S3", "restic", "backups", "Ansible"]
---

It turns out [restic][restic] is a bit less trivial to set up with Amazon's S3
due to some incoherency in URI handling by Amazon S3 leading to subtle
[problems][ghbug] with repo initialization where `restic init` is only fine with
a specific variant of the S3 bucket URI and all other `restic` commands would
work with another one.  While the linked bug is about Scaleway's object storage,
the problem encountered also applied to AWS S3.

[restic]: https://restic.net
[ghbug]: https://github.com/restic/restic/issues/2971

This post documents an [Ansible][ansible] sample variable setup to automatically
generate proper restic repository paths.

[ansible]: https://ansible.com

<!--more-->

When setting up backups with, say, an Ansible role, one might want to set up two
different URI variables pointing to the same bucket for init and rest of the
work:

`vars/main.yml`:

```
aws_region: us-east-1
aws_bucket: tdemin-backups
restic_repo: {{ ansible_fqdn }}
restic_backup_path: "s3:s3.{{ aws_region }}.amazonaws.com/{{ aws_bucket }}/{{ restic_repo }}"
restic_init_path: "s3:{{ aws_bucket }}.s3.{{ aws_region }}.amazonaws.com/{{ restic_repo }}"
```

If you've followed my previous restic backup guide using systemd, then you could
then roughly use the last two variables in a playbook as follows:

`tasks/main.yml`:

```
- name: Install the configuration file
  template:
    mode: 0600
    owner: root
    group: root
    dest: /etc/restic/{{ restic_repo }}.env.conf
    content: |
      AWS_ACCESS_KEY_ID=...
      AWS_SECRET_ACCESS_KEY=...
      RESTIC_REPOSITORY={{ restic_backup_path }}
      RESTIC_PASSWORD=...
- name: Initialize the repository
  command:
    creates: /etc/restic/{{ restic_repo }}.created
    cmd: /usr/local/bin/restic init
  environment:
    AWS_ACCESS_KEY_ID: ...
    AWS_SECRET_ACCESS_KEY: ...
    RESTIC_REPOSITORY: {{ restic_init_path }}
    RESTIC_PASSWORD: ...
- name: Create the repository initialization notice
  file:
    path: /etc/restic/{{ restic_repo }}.created
    state: touch
    mode: "0600"
```

The playbook will non-interactively initialize a repository once, using the
appropriate URIs to access AWS S3 where needed.
