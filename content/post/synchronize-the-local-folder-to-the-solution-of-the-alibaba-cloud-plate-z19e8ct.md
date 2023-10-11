---
title: 将本地文件夹同步到阿里云盘的方案
slug: >-
  synchronize-the-local-folder-to-the-solution-of-the-alibaba-cloud-plate-z19e8ct
url: >-
  /post/synchronize-the-local-folder-to-the-solution-of-the-alibaba-cloud-plate-z19e8ct.html
date: '2022-06-12 15:00:12'
lastmod: '2023-10-11 16:37:15'
toc: true
tags:
  - 同步
  - 阿里云盘
keywords: 同步,阿里云盘
description: >-
  本文介绍了使用GoodSync和Alist将阿里云盘映射为WebDav服务，实现文件同步的方法。通过这种方式，可以方便地将文件夹同步到云端，并保持时间信息的一致性。
isCJKLanguage: true
---

## 准备工作

* 同步软件：GoodSync  
  这里我选择的是GoodSync，目前主流的此类同步软件我知道的有以下几种

  > |官网|特色（个人愚见）|
  > | ---------| ----------------------------------------------------------------------------------------------------|
  > |[GoodSync](https://www.goodsync.com/cn)|优点：功能最全面，UI比较好<br />缺点：同步文件不支持忽略文件时间只比较文件大小|
  > |[FreeFileSync](https://freefilesync.org/download.php)|优点：免费<br />缺点：不支持同步到WebDav|
  > |[FileGee](http://cn.filegee.com/product.html)|优点：国产，支持百度网盘/Dropbox/OneDrive/阿里云OSS/AmazonS3<br />缺点：UI较丑，扫描完成后没有同步树的展示|
  > |[odrive](https://www.odrive.com/homepage5b)|未使用|
  > |[微力同步](http://www.verysync.com/)|简单看了下好像不符合我的需求，|
  > |syncthing||
  > |nextcloud||
  >

  经过一些考虑，我选择了GoodSync
* 将阿里云盘进行处理以使其支持GoodSync的软件：[Alist](https://github.com/alist-org/alist)

  这里我没有使用 [messense/aliyundrive-webdav: 阿里云盘 WebDAV 服务](https://github.com/messense/aliyundrive-webdav)，因为我扫了一眼Release，发现没有exe（狗头）

  不使用CloudDrive的原因是CloudDrive会丢失文件的时间信息
* 云服务：阿里云盘

  可能有同学说为什么不使用OneDrive或者其他云服务，因为我觉得阿里云盘目前访问速度比较快

## 同步方法

1. 首先使用Alist将阿里云盘映射为WebDav服务

    具体步骤见Alist官网
2. 在GoodSync中将文件夹同步到webdav

    这里要注意有一个坑是必须关闭图中的选项，不然同步到

    ​![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310102144093.png)​

简单的小分享，大家也可以分享自己的方案
