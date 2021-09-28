---
title: "On using GORM: first experience"
date: 2021-09-28T21:27:00+05:00
draft: false
tags: ["Go", "golang", "programming", "ORM", "databases", "GORM"]
---

I rarely complain on others' software, but this time I felt compelled to share
my first impression on GORM, since it's quite an example of how you pick
software you need.

Since you usually don't care much about the upsides but rather have blockers
preventing you from using something at all, let's get right to the negative
side.

<!--more-->

A first glimpse at the [docs][docs] reveals you don't `(*DB).Close()` after
`gorm.Open()`. A more thorough examination results in finding out this is an
[expected change][soanswer] the author introduced in v1.20. Maybe the point to
have the connection pool closed automatically on application exit even is valid,
but expecting a Go dev to have to find that out and with a good chance still
write something like:

```go
defer func() {
    sqlDB, _ := db.DB()
    sqlDB.Close()
}()
```

[docs]: https://gorm.io/docs/
[soanswer]: https://stackoverflow.com/a/63817476

still looks weird.

While this on itself doesn't look like much of an issue, this is actually a sign
of GORM not following [semantic versioning][semver], which is baked into the Go
toolchain and is illustrated by its [docs on version numbers][versioning]. This
means users of `go get -u` might have a difficult time having to patch their
code after supposedly minor dependency updates.

[semver]: https://semver.org
[versioning]: https://golang.org/doc/modules/version-numbers

I am still going to use this library, after all, it does what I wanted quite
well; but hey, I have a right to complain! Right?..
