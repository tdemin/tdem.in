---
title: "AutoHotkey: Scratchpad"
date: 2019-06-10T01:28:30+05:00
draft: false
tags: ["AutoHotkey", "hack", "Windows"]
---

Another day, another hack. This [AutoHotkey][AHK] snippet preserves the last
window you minimized with `Win + Down` and allows you to quickly unminimize it
with `Win + Shift + Up`:

<!--more-->

```ahk
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
```

Now let's go even further and use a stack of windows for that purpose! The next
snippet saves the list of windows you minimized that way in a stack, and
restores them in order:

```ahk
global lastWindows := Array() ; this line needs to be at the top of the file
ToScratchpad(windows)
{
    WinGetTitle, lastWindow, A
    WinMinimize, %lastWindow%
    windows.Push(lastWindow)
}
FromScratchpad(windows)
{
    lastWindow := windows.Pop()
    WinActivate, %lastWindow%
}
#Down::ToScratchpad(lastWindows)
#+Up::FromScratchpad(lastWindows)
```

The first line [has to be placed before the first return/hotkey][docs], or it
won't be executed (and the script will not work).

[AHK]: https://autohotkey.com "a brilliant program for system automation"
[docs]: https://www.autohotkey.com/docs/Scripts.htm#auto "AHK docs on auto-execute"
