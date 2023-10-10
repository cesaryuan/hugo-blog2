---
title: conda太慢怎么办，使用mamba代替龟速conda
slug: >-
  what-should-i-do-if-conda-is-too-slow-use-mamba-instead-of-turtle-speed-conda-z1oondz
url: >-
  /post/what-should-i-do-if-conda-is-too-slow-use-mamba-instead-of-turtle-speed-conda-z1oondz.html
date: '2023-10-08 10:04:17'
lastmod: '2023-10-10 21:26:01'
toc: true
tags:
  - Python
  - conda
  - mamba
categories:
  - Coding
keywords: Python,conda,mamba
description: >-
  本文介绍了使用mamba替代conda进行包管理的方法，并分析了mamba相较于conda在速度和内存占用方面的优势。通过对比实验，发现mamba在安装和移除包时具有更快的速度，并且解决依赖关系的效率更高。同时，文章提供了迁移至mamba的步骤和注意事项，以及在新环境中设置mamba的方法。
isCJKLanguage: true
---

## 介绍

conda无论是在remove还是install的时候，速度都太慢了，有时候甚至能长达半小时，还有内存泄漏问题（有一次执行`conda remove`​的时候内存占用到了15G），因此经过我的一番冲浪，发现了mamba这个东西，[甚至连伯克利都在推荐mamba而不是conda](https://statistics.berkeley.edu/computing/conda)，下面是mamba的一段介绍：

> ​`mamba`​ is a reimplementation of the conda package manager in C++.
>
> * parallel downloading of repository data and package files using multi-threading
> * libsolv for **much faster dependency solving**, a state of the art library used in the RPM package manager of Red Hat, Fedora and OpenSUSE
> * core parts of `mamba`​ are implemented in C++ for **maximum efficiency**

因此，我从conda迁移到了mamba，下面迁移过程的一些记录和总结

## 使用

- 迁移：

  这是经过速度对比之后选择的，这种方式不需要卸载原有的ananconda或者miniconda，只需创建一个`.condarc`​​然后将`env_dirs`​​指向原来conda的env位置即可

  然后在pwsh的profile里面加上`(& env:MAMBA_EXE 'shell' 'hook' -s 'powershell' -p env:MAMBA_ROOT_PREFIX) | Out-String | Invoke-Expression`​​即可
* 新环境：

  我选择先安装miniconda3然后再安装micromamba

## 速度对比

对比时使用的condarc：

```js
channels:
  - conda-forge
  - defaults
auto_activate_base: false
show_channel_urls: true
envs_dirs:
  - E:\scoop\apps\miniconda3\current\envs
pkgs_dirs:
  - E:\scoop\apps\miniconda3\current\pkgs

```

|命令|速度|备注|
| -----------------------------------| ------| -------------|
|conda install -y matplotlib=3.3.1|365s||
|**micromamba install -y matplotlib=3.3.1**|25s||
|mamba install -y matplotlib=3.3.1|70s|[Existing conda install](https://mamba.readthedocs.io/en/latest/mamba-installation.html#existing-conda-install-not-recommended)<br />|
|mamba install -y matplotlib=3.3.1|90s|[Fresh install](https://mamba.readthedocs.io/en/latest/mamba-installation.html#fresh-install-recommended) mambaforge|

|命令|速度|备注|
| ----------------------------------| ------| -------------|
|conda remove -y matplotlib=3.3.1|217s||
|**micromamba remove -y matplotlib=3.3.1**|11s||
|mamba remove -y matplotlib=3.3.1|64s|[Existing conda install](https://mamba.readthedocs.io/en/latest/mamba-installation.html#existing-conda-install-not-recommended)<br />|
|mamba remove -y matplotlib=3.3.1|141s|[Fresh install](https://mamba.readthedocs.io/en/latest/mamba-installation.html#fresh-install-recommended) mambaforge|

‍
