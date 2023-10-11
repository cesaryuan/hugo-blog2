---
title: 关于通过CMD命令睡眠Windows10的研究
slug: research-on-windows-10-through-cmd-commands-z17flp5
url: /post/research-on-windows-10-through-cmd-commands-z17flp5.html
date: '2023-07-21 14:17:48'
lastmod: '2023-10-11 16:45:23'
toc: true
tags:
  - SetSuspendState
  - 睡眠
categories:
  - Windows
keywords: SetSuspendState,睡眠
description: >-
  本文探讨了在使用CMD命令睡眠系统时的常见错误方法，并提供了正确的方式，解释了为什么这些错误方法会导致系统休眠而不是睡眠，最后介绍了通过powershell调用Winform函数来实现睡眠的方法。
isCJKLanguage: true
---

网络上能找到的睡眠命令几乎都使用的`rundll32.exe`​来实现，然而经过检索发现，这似乎是不正确的，这种方法下实现的是休眠（hibernate）而不是睡眠（suspend）。

> 主流的实现方式：​`rundll32.exe powrprof.dll,SetSuspendState 0,1,0`​

下面我们分析一下，首先看`SetSuspendState`​的[函数签名](https://learn.microsoft.com/en-us/windows/win32/api/powrprof/nf-powrprof-setsuspendstate)：

​![](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310102150130.png)​

可以看到MSDN中提到，如果第一个参数为True，则系统会休眠，False则系统会睡眠

看到这里，仿佛该方法是正确的，因为其看起来是分别把0,1,0传入了方法的三个参数，0代表第一个参数是False，然而实际上，rundll32不是这样传参的，这种用法是对rundll32的错误使用。

为什么说这种用法是对rundll32的错误使用呢？根据[How to use Rundll32 to execute DLL Function? - Stack Overflow](https://stackoverflow.com/questions/3207365/how-to-use-rundll32-to-execute-dll-function/3207411#3207411)，rundll32仅支持签名为`void CALLBACK EntryPoint(HWND hwnd, HINSTANCE hinst, LPSTR lpszCmdLine, int nCmdShow);`​的函数，因此`0,1,0`​并不会按照你预想的方式挨个传递进去。相反，在这种运行方式下，`SetSuspendState`​函数实际接收到的参数是：`[SomePtr1], [SomePtr2], "0,1,0", ...`​，这就导致了第一个参数一定不为0，也就是一定为True，也就是实际会使得系统休眠而不是睡眠。

那么为什么网络上有很多人反馈这个命令好使呢？这里猜测是因为他们把休眠功能关掉了（`powercfg -hibernate off`​或者控制面板手动关掉）

## 正确方式

分析完了主流方法为什么是错误的，我们来谈一下如何正确的通过CMD睡眠系统呢？这里的核心思路还是调用`SetSuspendState`​这个函数

1. 自己写一个ahk或者C#或者C++小程序，通过DllCall之类的技术正确的调用`SetSuspendState`​来实现睡眠。这种方法的缺点是需要一个单独的程序。
2. 通过powershell来调用Winform中的`SetSuspendState`​函数：`powershell "[Void][System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms');[System.Windows.Forms.Application]::SetSuspendState('Suspend', $false, $false);"`​

## 感想

有时候互联网上方法也不是那么可信，还是要自己去探究各中原理😝

‍
