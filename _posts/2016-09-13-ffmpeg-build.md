---
layout: post
title: "How to build ffmpeg?"
category: prog
tags: ffmpeg
keywords: build
---

출처: http://ffmpeg.org/platform.html#Windows

4.1.1 Native Windows compilation using MSYS2

The MSYS2 MinGW-w64 environment provides ready to use toolchains and dependencies through pacman. 

Make sure to use mingw64_shell.bat or mingw32_shell.bat to have the correct MinGW-w64 environment. The default install provides shortcuts to them under MinGW-w64 Win64 Shell and MinGW-w64 Win32 Shell. 

# normal msys2 packages
pacman -S make pkgconf diffutils

# mingw-w64 packages and toolchains
pacman -S mingw-w64-x86_64-yasm mingw-w64-x86_64-gcc mingw-w64-x86_64-SDL


To target 32 bits replace x86_64 with i686 in the command above. 


* [MSys2 Install](https://msys2.github.io/)


* package install

```
# normal msys2 packages
pacman -S make pkgconf diffutils

# mingw-w64 packages and toolchains
pacman -S mingw-w64-x86_64-yasm mingw-w64-x86_64-gcc mingw-w64-x86_64-SDL
```

* 환경 변수 설정

```
C:\msys64\mingw64\x86_64-w64-mingw32\bin;C:\msys64\mingw64\bin;

or

C:\msys64\mingw32\i686-w64-mingw32\bin;C:\msys64\mingw32\bin;

```


* `mingw32_shell.bat` or `mingw32_shell.bat` 실행


* `build_ffmpeg.sh x86` or `build_ffmpeg.sh x64`


* `build_ffmpeg.sh`에서 64비트 일경우 아래와 같이 수정한다.


```
-            cross_prefix=x86_64-w64-mingw32-
+            cross_prefix=

```

* [LAVFilters](https://github.com/hsccr/LAVFilters)에서는 [ffmpeg](http://git.1f0.de/gitweb?p=ffmpeg.git;a=summary)의 것을 사용하고 있습니다.
