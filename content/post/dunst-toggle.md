---
title: Toggling dunst notifications in i3
date: 2020-08-12T18:02:33+05:00
tags: [Linux, dunst, i3]
---

In case you always wanted to write a script that temporarily turns off dunst
notifications in i3wm but have never taken the time to write one yourself,
here, have a working implementation in bash.

<!--more-->

`${HOME}/.local/bin/dunst_toggle.sh`:

```sh
#!/usr/bin/env bash

notify="notify-send -u low dunst"

case `dunstctl is-paused` in
    true)
        dunstctl set-paused false
        $notify "Notifications are enabled"
        ;;
    false)
        $notify "Notifications are paused"
        # the delay is here because pausing notifications immediately hides
        # the ones present on your desktop; we also run dunstctl close so
        # that the notification doesn't reappear on unpause
        (sleep 3 && dunstctl close && dunstctl set-paused true) &
        ;;
esac
```

Now let's add a hotkey that toggles notifications in i3 config:

`${HOME}/.config/i3/config`:

```
bindsym $mod+F5 exec --no-startup-id ~/.local/bin/dunst_toggle.sh
```

That's it, pressing Win + F5 will now toggle notifications on/off. Any
notifications received during downtime will be shown as soon as dunst
is unpaused.
