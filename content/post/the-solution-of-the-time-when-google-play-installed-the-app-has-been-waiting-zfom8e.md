---
title: Google Play 安装 APP 时一直等待中的解决办法
slug: >-
  the-solution-of-the-time-when-google-play-installed-the-app-has-been-waiting-zfom8e
url: >-
  /post/the-solution-of-the-time-when-google-play-installed-the-app-has-been-waiting-zfom8e.html
date: '2024-04-18 16:50:16'
lastmod: '2024-04-18 18:29:23'
toc: true
description: >-
  本文介绍了解决Google
  Play安装APP一直等待中的问题。通过分析下载流程，发现问题根源在于Google国内CDN屏蔽了proxy连接，导致下载失败。提出换节点、关掉梯子、调整规则等多种解决方法，以确保访问成功。
isCJKLanguage: true
---

# Google Play 安装 APP 时一直等待中的解决办法

## 先说方法

前提：`*.googleapis.com`​要通过proxy访问

然后

1. 保证`services.googleapis.cn`​和`*.xn--ngstr-lra8j.com`​都是direct国内直连状态

或者

2. 保证`services.googleapis.cn`​和`*.xn--ngstr-lra8j.com`​都是proxy访问状态

## 分析

最近Google Play下载应用一直显示等待中，然后我网站查询了一下，发现几个可用的解决办法

* 换节点（反正就只一直换就完了）
* [等待下载时关掉梯子](https://zhuanlan.zhihu.com/p/68315260)
* 添加规则`DOMAIN-SUFFIX,services.googleapis.cn,PROXY`​
* 添加规则`DOMAIN-SUFFIX,xn--ngstr-lra8j.com,DIRECT`​

这几个都能解决问题，但是勾起了我的兴趣，为什么四个看起来完全不相干的解决措施却都能够解决问题呢，这其中是不是有更深层次的原因呢，经过一番研究，我认为是以下原因：

Google Play点击安装APP时的流程是这样的

1. 点击安装，进入等待中状态；
2. 访问`https://play-fe.googleapis.com/fdfe/delivery`​，返回应用的下载链接`https://services.googleapis.cn/download/by-token/download`​。PS: 这里根据Google Play区域设置的不同返回不同的域名，对于中国，域名为`services.googleapis.cn`​
3. 访问步骤2返回的下载链接`services.googleapis.cn`​，返回302跳转到真正的Google全球CDN下载地址`https://[xxxx].xn--ngstr-lra8j.com/play-apps-download-latchsky/download/by-id/`​。PS: 从国内访问`services.googleapis.cn`​会返回Google国内CDN地址，通过proxy访问`services.googleapis.cn`​会返回Google国外CDN地址。
4. 访问步骤3中返回的Google CDN下载地址`[xxxx].xn--ngstr-lra8j.com`​，若成功访问，则开始下载应用，否则一直显示等待中

所以为什么我们在开启proxy的情况下还是经常遇到等待中的情况呢，问题其实出在步骤4， **<u>大部分机场提供的clash文件会把</u>** ​ **<u>​`services.googleapis.cn`​</u>** ​ **<u>分流到DIRECT国内直连，把</u>** ​  **<u>​`[xxxx].xn--ngstr-lra8j.com`​</u>** ​ **<u>分流到PROXY连接</u>** 。这就导致了这样一种情况，在步骤3中，由于未经过proxy走的国内直连，所以`services.googleapis.cn`​返回的CDN下载地址是国内的CDN，这时在步骤4中，会通过proxy连接这个步骤3返回的国内CDN下载地址，然而Google的这个国内CDN屏蔽了国外服务器的访问（也就是你proxy所在的服务器），这就导致访问不能成功，也就会一直等待中。

了解了问题出在哪，我们就知道四个解决方法是如何解决问题的了：

对于方法1：换节点，只要我们能换到一个不被Google国内CDN屏蔽的国外节点，那么自然就可以成功

对于方法2：等待下载时关梯子，只要我们在步骤4访问Google国内CDN的同时关闭梯子，这时就会直连Google国内CDN，从而下载成功

对于方法3：让`services.googleapis.cn`​通过代理连接，这样步骤3返回的就是Google的国外CDN节点，通过proxy连接国外CDN下载链接自然没有问题，从而下载成功

对于方法4：让`*.xn--ngstr-lra8j.com`​通过国内直连，国内访问步骤3返回的Google国内CDN自然没有问题，从而下载成功

‍

‍
