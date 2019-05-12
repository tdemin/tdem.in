---
title: "AutoHotkey Run-or-Raise"
date: 2019-05-12T18:12:17+05:00
draft: false
---

It is a common practice for some to use a Quake-style terminal. For those who
have never played the game: it's a single terminal window which is hidden by
default and is toggled with the grave accent key. Those who like the behaviour
use a terminal emulator program like Tilda on Linux or ConEmu on Windows.

Sometimes you might want the same behaviour for the utilities that do not
support Quake-style windows by themselves. Luckily, you can still achieve this
with a simple [AutoHotkey][AHK] script I've come up with:

    RunOrRaise(class, run)
    {
        if WinActive(class) {
            WinMinimize, %class%
        }
        else if WinExist(class) {
            WinActivate, %class%
        }
        else {
            Run, %run%
        }
    }
    ; run PowerShell on Win + Z
    #z::RunOrRaise("ahk_exe powershell.exe", "powershell -NoLogo")

The first parameter is the [AHK window class][AHKWinTitle], and the second one
is the command to be run if the window doesn't exist yet (usually the path to
the program).

If you aren't familiar with AutoHotkey yet, I stronly recommend you to check it
out, it's a lifesaver for those who are still running Windows for whatever
reason: this program allows scripting system actions as complex as
[tiling window management][bug.n], for instance.

[AHK]: https://autohotkey.com "a brilliant program for system automation"
[AHKWinTitle]: https://autohotkey.com/docs/misc/WinTitle.htm "AutoHotkey WinTitle docs"
[bug.n]: https://github.com/fuhsjr00/bug.n "bug.n, a tiling WM for Windows"
