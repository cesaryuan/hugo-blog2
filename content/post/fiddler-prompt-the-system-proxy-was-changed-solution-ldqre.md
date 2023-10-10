---
title: Fiddler 提示 The system proxy was changed 的解决办法
slug: fiddler-prompt-the-system-proxy-was-changed-solution-ldqre
url: /post/fiddler-prompt-the-system-proxy-was-changed-solution-ldqre.html
date: '2022-02-15 11:53:21'
lastmod: '2023-10-10 16:05:19'
toc: true
tags:
  - fiddler
  - sangfor
  - 抓包
categories:
  - 网络
keywords: fiddler,sangfor,抓包
description: >-
  解决了Fiddler一直以来老是出现the system proxy was changed click to reenable capturing
  的问题，原因是未删除的Sangfor VPN软件导致。通过排查进程和服务，卸载Sangfor VPN后重启Fiddler问题得到解决。
isCJKLanguage: true
---

终于解决了Fiddler一直以来老是 The system proxy was changed. Click to reenable capturing 的问题了

原因在于一个VPN软件（深信服Sangfor），好像是本科时使用学校的VPN软件安装上的，结果卸载了学校的VPN后sangfor却没删除。

# 排查过程

首先通过Google，其中[一个帖子](https://blog.csdn.net/weixin_44010756/article/details/115016604)提到了四种可能的原因

1. 电脑上安装了银联控件
2. 开启了防火墙或者是VPN
3. 拨号和虚拟专用网络设置下面框中有不用的代理服务器

第一点我经过在进程中和程序中搜索后排除了，接着是第二种，我先关闭了防火墙，不管用，接着我去服务service中搜索vpn，还真搜到了一个sangfor正在运行，接着我就卸载然后重启fiddler，就好了！
