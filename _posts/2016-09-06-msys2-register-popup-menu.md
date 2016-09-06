---
layout: post
title: "MSYS2 'Open Here' registry settings"
category: prog
tags: msys2
keywords: registry
---

`registry.reg`

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\Background\shell\open_msys2]
@="Open MSYS2 here"

[HKEY_CLASSES_ROOT\Directory\Background\shell\open_msys2\command]
@="c:\\msys64\\usr\\bin\\mintty.exe /bin/sh -lc 'cd \"$(cygpath \"%V\")\"; exec bash'"

[HKEY_CLASSES_ROOT\Folder\shell\open_msys2]
@="Open MSYS2 here"

[HKEY_CLASSES_ROOT\Folder\shell\open_msys2\command]
@="c:\\msys64\\usr\\bin\\mintty.exe /bin/sh -lc 'cd \"$(cygpath \"%V\")\"; exec bash'"

```