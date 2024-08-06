---
title: 记一次分析Cloudflare异常流量的过程
slug: remember-to-analyze-the-process-of-cloudflare-abnormal-flow-zs9joq
url: /post/remember-to-analyze-the-process-of-cloudflare-abnormal-flow-zs9joq.html
date: '2024-08-06 20:24:57+08:00'
lastmod: '2024-08-06 21:11:46+08:00'
toc: true
isCJKLanguage: true
---



几个月以来，Cloudflare 的 HTTP Traffic 都有些问题，每天都有8万多个请求，但是我的网站按理说都是自己用，不应该有这么大流量

​![image](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgsimage-20240806203012-29es6y3.png)​

我去查看了流量的来源，发现大部分来自于韩国，联想到我的甲骨文免费服务器都是在韩国，所以大概是我的服务器安装的一些应用在不停得向我代理在Cloudflare上的服务发送请求。

​![image](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgsimage-20240806204440-a9tuxcn.png)​

因此，我开始尝试筛查出到底是哪一台甲骨文服务器上的哪个应用在访问哪个部署在CF上的服务。

理论上最简单的方法是开通CF的Pro功能，就可以在CF的Panel按subdomain查看更详细的日志，但是好贵呜呜（20刀每月），所以还是要从自己的服务器入手。

首先想到的是ARM服务器，因为这台服务器上部署了大量的服务，我咨询了DeepSeek：`我发现我的服务器在向cloudflare段的IP地址持续发送一些流量，有什么办法筛查出到底是哪一个进程、什么端口和协议再发送这些流量呢`​，得到的回答如下：

> 要筛查出服务器上哪个进程、端口和协议在向 Cloudflare 段的 IP 地址发送流量，你可以使用一些网络监控和分析工具。以下是一些常用的方法：
>
> ### 1. 使用 `netstat`​ 或 `ss`​ 命令
>
> ​`netstat`​ 和 `ss`​ 命令可以帮助你查看当前的网络连接和监听端口。你可以结合 `grep`​ 命令来过滤出与 Cloudflare IP 段相关的连接。
>
> ```bash
> sudo netstat -tunap | grep <Cloudflare_IP_Range>
> ```
>
> 或者使用 `ss`​ 命令：
>
> ```bash
> sudo ss -tunap | grep <Cloudflare_IP_Range>
> ```
>
> 将 `<Cloudflare_IP_Range>`​ 替换为具体的 Cloudflare IP 段，例如 `104.16.0.0/12`​。
>
> ### 2. 使用 `iftop`​ 或 `nethogs`​
>
> ​`iftop`​ 和 `nethogs`​ 是实时流量监控工具，可以帮助你查看哪些进程在发送流量。
>
> * ​`iftop`​：
>
>   ```bash
>   sudo iftop -n -F <Cloudflare_IP_Range>
>   ```
> * ​`nethogs`​：
>
>   ```bash
>   sudo nethogs
>   ```
>
> ### 3. 使用 `lsof`​ 命令
>
> ​`lsof`​ 命令可以列出当前打开的文件和网络连接，结合 `grep`​ 可以过滤出特定的连接。
>
> ```bash
> sudo lsof -i | grep <Cloudflare_IP_Range>
> ```
>
> ### 4. 使用 `tcpdump`​
>
> ​`tcpdump`​ 是一个强大的网络抓包工具，可以捕获和分析网络流量。
>
> ```bash
> sudo tcpdump -nn host <Cloudflare_IP_Range>
> ```
>
> ### 5. 使用 `strace`​ 命令
>
> 如果你已经知道某个进程的 PID，可以使用 `strace`​ 命令来跟踪该进程的系统调用，从而查看其网络活动。
>
> ```bash
> sudo strace -p <PID> -e trace=network
> ```
>
> ### 6. 使用 `ss`​ 和 `ps`​ 结合
>
> 你可以结合 `ss`​ 和 `ps`​ 命令来查找具体的进程和端口。
>
> ```bash
> sudo ss -tunap | grep <Cloudflare_IP_Range> | awk '{print $6}' | cut -d':' -f2 | xargs -I {} ps -p {} -o pid,cmd
> ```
>
> ### 7. 使用 `nmap`​ 扫描
>
> 如果你怀疑有未授权的服务在运行，可以使用 `nmap`​ 扫描开放的端口。
>
> ```bash
> sudo nmap -sT -O localhost
> ```
>
> 通过这些工具和命令，你应该能够筛查出服务器上哪个进程、端口和协议在向 Cloudflare 段的 IP 地址发送流量。根据具体情况选择合适的工具和方法进行分析。

最后采用nethogs命令，发现是172.19.0.2这个ip一直在Receive流量，依稀的记得这个IP段一般是docker在用，所以通过这个命令`docker network inspect $(docker network ls -q) | grep -B 10 "172.19.0.2"`​找到了目标容器也就是 nginx-proxy-manager。

那么，接下来就是想办法找到是 nginx-proxy-manager 中的哪个域名在不断被访问，通过在`data/logs`​目录运行`ls -lt`​按时间排序，发现`proxy-host-10_access.log`​文件最大且最近被访问过，那么大概就说明是编号为`10`​的`proxy-host`​是那个不断被访问的服务器，打开文件查看，找到了目标域名`xxxx.example.com`​，并且请求在以每秒一次的频率访问。

> ​`data/logs`​中有很多日志文件，这些文件是 Nginx Proxy Manager 为每个代理主机（proxy host）生成的日志文件。每个代理主机都有两个日志文件：一个用于访问日志（access log），一个用于错误日志（error log）。
>
> 以下是这些日志文件的解释：
>
> 1. **访问日志（Access Log）** ：
>
>     * ​`proxy-host-X_access.log`​：记录了代理主机 X 的访问日志，包括客户端请求的详细信息，如请求的 URL、客户端 IP、请求时间、响应状态码等。
> 2. **错误日志（Error Log）** ：
>
>     * ​`proxy-host-X_error.log`​：记录了代理主机 X 的错误日志，包括 Nginx 在处理请求时遇到的错误信息，如配置错误、连接问题等。
> 3. **压缩的旧日志文件**：
>
>     * ​`proxy-host-X_access.log.1.gz`​ 和 `proxy-host-X_error.log.1.gz`​：这些是压缩的旧日志文件，通常是日志轮转（log rotation）的结果，用于节省磁盘空间。
> 4. **空日志文件**：
>
>     * 文件大小为 0 的日志文件（如 `proxy-host-X_error.log`​）：这些文件表示在相应的时间段内没有生成任何日志记录。
> 5. **Fallback 日志**：
>
>     * ​`fallback_access.log`​ 和 `fallback_error.log`​：这些日志文件记录了 Nginx Proxy Manager 自身的访问和错误信息，而不是特定代理主机的日志。

但是此刻还是不知道导致是哪个服务在访问它，此时又打开了另外两台AMD服务器运行`nethogs`​，发现了是目标服务器AMD2中的某个docker容器，然后`docker ps`​后发现了可疑容器`autospy`​，将其停止后终于恢复正常。
