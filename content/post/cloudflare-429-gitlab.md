---
title: "Advice: CloudFlare, GitLab Pages, 429"
date: 2018-10-25T12:00:00+00:00
draft: false
tags: ["CloudFlare", "GitLab"]
---

To all those people having their Cloudflare requests to GitLab Pages being 429-ed: setting the security policy to “Full” instead of “Flexible” seems to do the trick. Still have no clue to the cause of this issue though.
