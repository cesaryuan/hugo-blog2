---
title: 更纱黑体中各个版本的区别（截图）
slug: the-difference-between-each-version-of-the-gauze-black-body-screenshot-sceew
url: >-
  /post/the-difference-between-each-version-of-the-gauze-black-body-screenshot-sceew.html
date: '2022-02-03 12:30:31'
lastmod: '2023-10-10 14:48:05'
toc: true
tags:
  - 更纱黑体
  - 字体包
  - 编程字体
  - 系统字体
categories:
  - 编程字体
keywords: 更纱黑体,字体包,编程字体,系统字体
isCJKLanguage: true
---

众所周知，更纱黑体[Release页面](https://github.com/be5invis/Sarasa-Gothic/releases)下载的字体包[sarasa-gothic-ttf-&lt;version&gt;.7z](https://github.com/be5invis/Sarasa-Gothic/releases/download/v0.35.8/sarasa-gothic-ttf-0.35.8.7z)解压后足足有12G大小，且其中有480个ttf文件，一般人不知道如何选择安装，这里说一下该如何选择。

## 结论

* 用作编程字体

  安装所有以`sarasa-fixed-sc-`开头的字体
* 用作系统字体

  安装所有以`sarasa-ui-sc-`开头的字体

## 官方说明

* Style dimension

  * Latin/Greek/Cyrillic character set being Inter（拉丁字符集不等宽）

    * Quotes (`“”`) are full width —— **Gothic**
    * Quotes (`“”`) are narrow —— **UI**
  * Latin/Greek/Cyrillic character set being Iosevka（拉丁字符集等宽）

    * Em dashes (`——`) are full width —— **Mono**（破折号全宽）
    * Em dashes (`——`) are half width —— **Term**（破折号半宽）
    * No ligature, Em dashes (`——`) are half width —— **Fixed**（前面没懂破折号半宽）
* Orthography dimension

  * `CL`: Classical orthography
  * `SC`, `TC`, `J`, `K`, `HC`: Regional orthography, following [Source Han Sans](https://github.com/adobe-fonts/source-han-sans) notations. 参见[SourceHanSansReadMe.pdf](assets/SourceHanSansReadMe-20220203163546-dxsv38v.pdf)

> 补充：
>
> * 文件中还有一部分ttf文件名中包含slab字样，意思是[拉丁字母带衬线](https://github.com/be5invis/Sarasa-Gothic/issues/159)
> * ttc == ttf collection（每个字重一个文件，每个文件里包含多种dimension）

## 截图展示

以下截图均基于regular字重

* 非等宽拉丁

  * ui-sc

    ![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310101503050.png)
  * gothic-sc

    ![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310101503123.png)
* 等宽拉丁

  * mono-sc

    ![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310101503890.png)
  * fixed-sc

    ![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310101503414.png)
  * term-sc

    ![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310101503414.png)
  * fixed-cl

    ![image.png](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310101503197.png)
