---
layout: post
title: "How to open console in visual studio?"
category: prog
tags: visualstudio
keywords: console
---

```
#pragma comment(linker, "/entry:WinMainCRTStartup /subsystem:console")
```

```
속성 -> Linker -> System -> SubSystem => Windows (/SUBSYSTEM:WINDOWS)
```

```
AllocConsole();
...
WriteFile( GetStdHandle( STD_OUTPUT_HANDLE ), str , strlen( str ) , &size , NULL );
...
FreeConsole();
```