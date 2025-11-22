---
layout: post
title: "Docker desktop on windows can not modify the settings problem"
---

在过去两天里我被一个奇怪的问题折磨，在Windows11机器上安装了docker desktop后，每次进入设置页面都会有一个错误横幅提示我docker遇到了一个error。
实际上在命令行和ui上是可以正常执行命令的。但是这个红色的错误横幅非常影响我的心情，我决定找一找原因。

首先我重新卸载并安装了docker desktop，过程十分顺利，但这并不起作用。然后寻找docker desktop的运行日志或者错误日志。最后我决定使用之前的版本。

https://docs.docker.com/desktop/release-notes/

解决方案:
1. v4.20.0 -> v4.19.0
