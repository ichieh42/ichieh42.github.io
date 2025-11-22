---
layout: post
title: "Started btsync but cannot discover peer nodes"
---

最近在新的机器上安装了`BTSync-1.4.111.exe`，但是配置完代理后依然不能搜索到节点。

## 排查过程

从代理的工具日志查看是正常发送了请求的，查看具体的请求列表中有一个403的请求比较特别

```
# http://config.usyncapp.com/sync.conf

403 Forbidden
Code: AllAccessDisabled
Message: All access to this object has been disabled
RequestId: 0XJJKQ58XVR3HHZ6
HostId: j0P/pB91qCCYqq7lXByUkEXrIJFBg4bUz5m8BqelN0qyoKpfebhA+M+ui15n1fesAxUItodiwTg=
```

点击启用BTSync的配置页面可以勾选启用调试日志选项方便我们确认这个错误

在`%APPDATA%\Roaming\BitTorrent Sync`目录中有一个运行日志文件`sync.log`

观察日志文件可以看到影响同步的问题

```
...
[2023-12-18 14:45:48] API: --> getsyncfolders(discovery=1&t=1702881948381)
[2023-12-18 14:45:49] API: --> getsyncfolders(discovery=1&t=1702881949412)
[2023-12-18 14:45:50] API: --> getsyncfolders(discovery=1&t=1702881950432)
[2023-12-18 14:45:51] Requesting folder config from config.usyncapp.com
[2023-12-18 14:45:51] API: --> getsyncfolders(discovery=1&t=1702881951470)
[2023-12-18 14:45:52] API: --> getsyncfolders(discovery=1&t=1702881952523)
[2023-12-18 14:45:52] Failed to parse folder config. Error: lexical error: invalid char in json text.
                                        <html> <head><title>403 Forbid
                     (right here) ------^

[2023-12-18 14:45:53] API: --> getsyncfolders(discovery=1&t=1702881953545)
[2023-12-18 14:45:54] API: --> getsyncfolders(discovery=1&t=1702881954562)
[2023-12-18 14:45:55] API: --> getsyncfolders(discovery=1&t=1702881955587)
[2023-12-18 14:45:56] API: --> getsyncfolders(discovery=1&t=1702881956605)
...
```

因响应结果不能正常转换导致程序一直停留在这一步，这就是我们要人工干预解决的问题

## 解决方法

我们试图找到`sync.conf`源文件，先在互联网搜索

从编程随想的博客`https://program-think.blogspot.com/2017/08/GFW-Resilio-Sync.html`可以看到

```
◇Tracker Server（追踪服务器）

　　BT 下载的老用户应该听说过 Tracker 这个概念。打个比方，Tracker Server 类似于节点（peer）之间的中介，互不认识的节点通过 Tracker Server 来获得对方的信息（IP 和端口）。
　　那么，BTsync 的客户端是如何知道 Tracker Server 的 IP 地址捏？通常有两种方法：
　　方法1
　　客户端软件本身已经内置了若干个 Tracker Server 的 IP 地址和端口。
　　方法2
　　客户端软件先连上 BTsync 公司官方的某个网站（通过域名的方式），然后得到某个配置文件，这个配置文件中会包含 Tracker Server 的 IP 地址和端口。比如下面这个网址，就是某些版本的 BTsync 用来获取配置文件的网址。
https://config.resilio.com/sync.conf

⬆上面的网址是我们的重点
```

在这里我推测原来的配置连接因为是http的不安全，于是最新的版本已经更新成了`https://config.resilio.com/sync.conf`。我们只需要使用该文件内容喂给旧版本程序即可。

解决办法五花八门，我这里仅提供几个思路

1. 配置应用服务器如`nginx`和本地host文件，使请求发往本地下载好的`sync.conf`文件
2. 将`http://config.usyncapp.com/sync.conf`请求转发到`https://config.resilio.com/sync.conf`获取最新的配置
3. ……
