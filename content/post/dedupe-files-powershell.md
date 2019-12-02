---
title: "Remove duplicate files with PowerShell"
date: 2019-12-02T22:02:37+05:00
draft: false
---

Well, I finally decided to clean up the files from my storage drive.

I found a lot of duplicate files unnecessarily taking up my storage space. As
looking through every single file on your hard disk is a tedious job, I wrote a
little PowerShell script to assist me with that.

```ps1
[CmdletBinding()]
param (
    [Parameter()]
    [string]
    $dir
)
if (!$dir) {
    $dir = Get-Location
};

class File {
    [string]$Hash
    [string]$FileName
};
class Duplicate {
    [string]$Orig
    [string]$Dupe
};

$files = @()
foreach ($file in Get-ChildItem -Path $dir -Recurse -File | Select-Object -ExpandProperty FullName) {
    $f = [File]::new()
    $f.FileName = $file
    $f.Hash = Get-FileHash -Path $file -Algorithm "SHA1" | Select-Object -ExpandProperty Hash
    $files += $f
}

$dupes = @()
for ($i = 0; $i -lt $files.Count; $i++) {
    for ($j = $i; $j -lt $files.Count; $j++) {
        if ($files[$i].Hash -eq $files[$j].Hash -And $files[$i].FileName -ne $files[$j].FileName) {
            $f = [Duplicate]::new()
            $f.Orig = $files[$i].FileName
            $f.Dupe = $files[$j].FileName
            $dupes += $f
        }
    }
}

# print the file names
$dupes
```

This script calculates a checksum for every file in the directory, recursively
passing through every folder inside the dir it was launched at (it accepts the
target through the first command line parameter as well).

As you might want to inspect the files before deleting them, this script simply
prints their names to the standard output. For a quick oneliner that would purge
every duplicate file found automatically, issue:

    PS > Remove-Duplicates.ps1 C:\Some\Path | Select-Object -ExpandProperty Dupe | Remove-Item
