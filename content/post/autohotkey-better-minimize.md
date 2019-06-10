---
title: "AutoHotkey: Scratchpad"
date: 2019-06-10T01:28:30+05:00
draft: false
---

Another day, another hack. This [AutoHotkey][AHK] snippet preserves the last
window you minimized with `Win + Down` and allows you to quickly unminimize it
with `Win + Shift + Up`:

    global lastWindow
    ToScratchpad()
    {
        WinGetTitle, lastWindow, A
        WinMinimize, %lastWindow%
    }
    FromScratchpad()
    {
        WinActivate, %lastWindow%
    }
    #Down::ToScratchpad()
    #+Up::FromScratchpad()

_Update 06/11/2019_: the script was slightly tweaked to handle apps with more
than one window better.

[AHK]: https://autohotkey.com "a brilliant program for system automation"
