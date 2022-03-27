---
title: Filtering Friendica posts in an RSS feed by tag
date: 2020-02-14T02:00:38+05:00
tags: [Friendica, fediverse, RSS, Flask, Python, programming]
---

The current version of Friendica [doesn't appear to support filtering posts by
tag](https://libranet.de/display/0b6b25a8-185e-3d67-ae42-cb8788243360) in an RSS
feed. Appending `?tag=XXX` to an RSS feed URI seems to be silently ignored.

As I wanted a custom feed that only has music posts in it, I wrote a quick and
dirty feed crawler in Python/Flask that removes all posts that do not have a
single mention of the specified tag in it. The whole program only took 50
minutes to write.

<!--more-->

After deployment the feed is available at
`/tag?token=some_token` where `tag` is the text we're searching for in the feed
content and `some_token` is whatever you set with `FRIENDICA_TOKEN` (only needed
so that Googlebot doesn't accidentally flood Friendica with requests).

```py
from os import getenv

import requests
from flask import Flask, request, Response
from bs4 import BeautifulSoup

app = Flask(__name__)

TOKEN = getenv("FRIENDICA_TOKEN", "some_token")
URI = getenv("FRIENDICA_URI", "https://some.friendica.server.tld/dfrn_poll/username")

@app.route("/<text>", methods=["GET"])
def home_route(text: str):
    if request.args.get("token") != TOKEN:
        return "unauthorized", 401
    res = requests.get(URI)
    if not res.ok:
        res.close()
        return "failed to retrieve data", 500
    data = BeautifulSoup(res.content.decode("utf8"), features="lxml")
    res.close()
    for entry_tag in data.find_all("entry"):
        content = entry_tag.find("content")
        if content is not None:
            if not text in str(content):
                entry_tag.extract()
    return Response(data.encode().decode().replace("</body></html>", "").replace("<html><body>", "").encode(), mimetype="application/rss+xml")
```
