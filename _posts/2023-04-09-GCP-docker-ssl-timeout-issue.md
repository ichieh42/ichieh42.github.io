---
layout: post
title: "GCP's ubuntu VM docker container ssl handshake timeout issue"
---

这是在谷歌的数据中心注册的虚拟机中出现的问题。

``` log
root@docker:/root $ curl https://www.github.com
curl: (35) Recv failure: Connection reset by peer
```

``` log
root@docker:/root $ openssl s_client -host github.com -port 443
CONNECTED(00000003)

```

复现步骤：
1. 创建一个ubuntu镜像的linux虚拟机
2. 安装docker并建立一个容器（容器镜像不重要）
3. 在容器里进行任意的ssl握手操作都会超时失败

解决方案：
1. 更换为debian镜像

在事件的开始我注意到docker容器里的nginx配置会转发失败，我尝试在宿主机curl访问对应容器端口的服务，这给了我正常的反馈。一度让我以为nginx配置错误了。后来经过两天的排查，终于定位到容器内的ssl握手请求会失败。

但是，在比较了正常机器和报错机器的证书和各种环境后，最终我也没有找到真正的问题。本来我还想直接求助谷歌支持的工作人员，但是因为语言问题和试用账号问题无法建立工单所以放弃了。

本人的网络底层知识和运维经验不够，所以你有什么建议吗？
