---
title: Windows 实现透明代理或者全局代理的方案总结
slug: >-
  windows-to-implement-a-solution-for-transparent-agents-or-global-agents-z2gvprv
url: >-
  /post/windows-to-implement-a-solution-for-transparent-agents-or-global-agents-z2gvprv.html
date: '2024-06-25 17:19:38+08:00'
lastmod: '2024-06-25 23:25:27+08:00'
toc: true
isCJKLanguage: true
---

# Windows 实现透明代理或者全局代理的方案总结

题中所指的透明代理或全局代理的意思为：无需手动设置 Proxy 的、应用无感知的、可以实现代理所有应用流量的代理方式。

## 透明代理的用处

* 全局抓包（中间人代理）：有些应用不会读取 System Proxy 设置，因此只能通过透明代理来实现此类应用流量的捕获
* 全局科学：懂的都懂

## 方案总结

Windows 上没有 iptables，我调查了目前大部分抓包类软件和科学类应用的透明代理实现方式，按照技术栈分类如下，总体来说推荐<u>基于 Windows Filtering Platform (WFP) 的方案</u>

### 基于 Windows Filtering Platform (WFP) 的方案

这种方案利用了 Windows 提供的 WFP API，需要进行驱动级别的编程，然而有一些项目提供了封装好的易于使用的 API，比较著名的有 [Netfilter SDK](https://netfiltersdk.com/nfsdk.html)，[WinDivert](https://github.com/basil00/Divert)

比如：

* 最负盛名的开源中间人代理工具 [mitmproxy](https://github.com/mitmproxy/mitmproxy) 在2024年初的10.2版本中发布了[基于 WinDivert 的 Local Redirect 模式](https://mitmproxy.org/posts/local-redirect/windows/)，无需用户设置系统代理即可捕捉全局流量
* [HTTP Debugger Pro](https://www.httpdebugger.com/) 则使用 [Netfilter SDK](https://netfiltersdk.com/nfsdk.html) 来实现全局系统代理
* [Proxifier 在 4.0 中也将处理流量的方式从 Inline Hook 迁移到了WFP](https://www.proxifier.com/new/)
* Clash.NET 基于 [Netfilter SDK](https://netfiltersdk.com/nfsdk.html) 实现全局科学
* [Netch ](https://github.com/netchx/netch)和 [nft-game](https://github.com/PinkD/nft-game) 同样基于 [Netfilter SDK](https://netfiltersdk.com/nfsdk.html)。
* [junjiexing/libredirect: 使用WFP重定向socket链接](https://github.com/junjiexing/libredirect)
* [zliu-fd/WinDivertProxy: Proxy any program via WinDivert to a specific server](https://github.com/zliu-fd/WinDivertProxy)
* [xljiulang/WindivertDotnet: 面向对象的WinDivert的dotnet异步封装](https://github.com/xljiulang/WindivertDotnet)

### 基于虚拟网卡接口的方案

这类方案的原理一般为先创建一个虚拟网卡，然后修改路由表将所有流量转发到代理，据说在性能上不如基于WFP的方案。Windows上主流开源方案有以下两个：

* [Wireguard 项目的 Wintun 接口](https://github.com/WireGuard/wintun) (Layer3)
* [OpenVPN 的 Tap 接口](https://github.com/Toney-Sun/openvpn)

具体来说，有以下几个项目：

* [mitmproxy 的 WireGuard 模式](https://docs.mitmproxy.org/stable/concepts-modes/#wireguard-transparent-proxy)
* [tun2socks](https://github.com/xjasonlyu/tun2socks/blob/main/README_ZH.md) 使用 OpenVPN 创建虚拟网卡，然后将流量转发到代理（[[教程] 在 Windows 上使用 tun2socks 进行全局代理 | by Tachyon | Medium](https://tachyondevel.medium.com/%E6%95%99%E7%A8%8B-%E5%9C%A8-windows-%E4%B8%8A%E4%BD%BF%E7%94%A8-tun2socks-%E8%BF%9B%E8%A1%8C%E5%85%A8%E5%B1%80%E4%BB%A3%E7%90%86-aa51869dd0d)）
* Clash 基于 Wintun 来实现全局科学
* [leaf ](https://github.com/eycorsican/leaf)项目使用 Rust 实现了 tun2socks，[Maple ](https://github.com/YtFlow/Maple)则是一个基于 leaf 的 Windows 全局代理方案

### 基于进程注入 Inline Hook 的方案

这类方案通过 hook connect 函数来实现流量的重定向。基于该方案的项目貌似比较少，已知 Proxifier 在 4.0 之间采取该方式实现全局流量捕获。

---

## 参考文章

* [Wintun：一款惊艳的 WireGuard 虚拟网卡接口驱动-技术圈](https://jishu.proginn.com/doc/363964760a2eb4f9d)
* [[教程] 在 Windows 上使用 tun2socks 进行全局代理 | by Tachyon | Medium](https://tachyondevel.medium.com/%E6%95%99%E7%A8%8B-%E5%9C%A8-windows-%E4%B8%8A%E4%BD%BF%E7%94%A8-tun2socks-%E8%BF%9B%E8%A1%8C%E5%85%A8%E5%B1%80%E4%BB%A3%E7%90%86-aa51869dd0d)
* [Windows驱动编程之NetFilter SDK-编程技术-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-268515.htm)
* [junjiexing/libredirect: 使用WFP重定向socket链接](https://github.com/junjiexing/libredirect)
* [WinDivert+MITM实现https流量透明_windivert proxy-CSDN博客](https://blog.csdn.net/qq_37353105/article/details/121281828)

‍

‍
