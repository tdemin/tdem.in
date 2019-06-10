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
        WinGetClass, lastWindow, A
        WinMinimize, ahk_class %lastWindow%
    }
    FromScratchpad()
    {
        WinActivate, ahk_class %lastWindow%
    }
    #Down::ToScratchpad()
    #+Up::FromScratchpad()

[AHK]: https://autohotkey.com "a brilliant program for system automation"
