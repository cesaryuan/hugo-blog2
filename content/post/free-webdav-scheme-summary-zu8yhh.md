---
title: 免费WebDAV的方案总结
slug: free-webdav-scheme-summary-zu8yhh
url: /post/free-webdav-scheme-summary-zu8yhh.html
date: '2023-10-09 21:01:12'
lastmod: '2023-10-10 21:00:22'
toc: true
tags:
  - WebDAV
categories:
  - 云存储
keywords: WebDAV
description: >-
  本文介绍了坚果云、TeraCloud、Koofr等多个提供WebDAV云存储服务的限制和特点，以及利用其他网盘服务结合挂载软件实现多云存储的方法。同时还介绍了如何让OneDrive实现WebDAV服务以及使用Cloudreve搭建支持多家云存储的云盘系统。
isCJKLanguage: true
---

1. 坚果云，限制：每月1G上传3G下载
2. TeraCloud，免费10G空间，90天没有登录过，可能会封号
3. 【目前使用】[Koofr](https://app.koofr.net/app/) 自带10G空间，还支持链接OneDrive（相当于拥有1T空间！），每日限制上传流量 1G。永久空间购买链接：[Koofr Cloud Storage: Lifetime Subscription (1TB) | StackSocial](https://stacksocial.com/sales/koofr-cloud-storage-plans-lifetime-subscription-1tb)
4. [MultCloud | 最佳的免费多云存储管理器](https://www.multcloud.com/cn/)每个月5GB流量

    ![image](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310102101134.png)​
5. 利用其他网盘服务结合挂载软件，参考[让OneDrive实现WebDAV服务 - 七米蓝](https://www.chirmyram.top/archives/onedrivewebdav)

    1. [WebDAV | AList文档](https://alist.nn.ci/zh/guide/webdav.html#webdav-%E9%85%8D%E7%BD%AE)
    2. [cloudreve/Cloudreve: 🌩支持多家云存储的云盘系统 (Self-hosted file management and sharing system, supports multiple storage providers)](https://github.com/cloudreve/Cloudreve)
    3. Rclone
    4. [messense/aliyundrive-webdav: 阿里云盘 WebDAV 服务](https://github.com/messense/aliyundrive-webdav)
