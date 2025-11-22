---
layout: post
title: "Xiaomi Mi Router 3G Installation OpenWrt Guide"
---

最近入手了一台小米路由器R3G，处理器型号为`ramips/mt7621`，于是尝试安装一直心心念念的OpenWrt系统。

使用硬件厂商搭载的系统总觉得在隐私方面无法保证，并且也限制了很多功能，所以一个开源的系统是有必要的。

上网翻阅了OpenWrt的官方文档和一些中文社区的资料后，决定整理一下方便其他人也做同样的事情。

进入[OpenWrt官网](https://openwrt.org/start)，此时右上角有选择语言选项，选择中文后可以获得经过翻译校对的残篇。

## 下载
在官方的[OpenWrt下载器链接](https://firmware-selector.openwrt.org/)可以使用模糊搜索快速查找到我们需要的系统映像文件，一般来说有4种：
- KERNEL （具有最少文件系统的Linux内核. 对于首次安装或恢复很有用.）
- KERNEL1 （独立的 Linux 内核映像。）
- ROOTFS0 （根文件系统作为单独的映像.）
- SYSUPGRADE （使用 Sysupgrade 映像以更新现有运行 OpenWrt 的设备。该映像可以在 LuCI 界面或终端中使用。）

我们从小米原厂系统直接安装OpenWrt的话需要的是[KERNEL1](https://downloads.openwrt.org/releases/22.03.5/targets/ramips/mt7621/openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-kernel1.bin)和[ROOTFS0](https://downloads.openwrt.org/releases/22.03.5/targets/ramips/mt7621/openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-rootfs0.bin)

两个映像文件可以提前下载到U盘然后挂载到路由器上，或者直接命令行下载到路由器上

## 安装过程

首先安装R3G的开发者版本，到[小米路由器官网](http://miwifi.com/miwifi_download.html)下载[miwifi_r3g_firmware_12f97_2.25.124.bin](https://bigota.miwifi.com/xiaoqiang/rom/r3g/miwifi_r3g_firmware_12f97_2.25.124.bin)

在小米路由器管理页面上传`miwifi_r3g_firmware_12f97_2.25.124.bin`并刷机

过渡到开发版后即可获取ROOT权限，这里我使用[acecilia/OpenWRTInvasion](https://github.com/acecilia/OpenWRTInvasion)仓库的python一键脚本（当然你也可以手动操作）

```
pip3 install -r requirements.txt # Install requirements
python3 remote_command_execution_vulnerability.py # Run the script

# 获取权限后连接上去会有banner出现
BusyBox v1.19.4 (2018-10-29 07:52:03 UTC) built-in shell (ash)
Enter 'help' for a list of built-in commands.

 -----------------------------------------------------
       Welcome to XiaoQiang!
 -----------------------------------------------------
  $$$$$$\  $$$$$$$\  $$$$$$$$\      $$\      $$\        $$$$$$\  $$\   $$\
 $$  __$$\ $$  __$$\ $$  _____|     $$ |     $$ |      $$  __$$\ $$ | $$  |
 $$ /  $$ |$$ |  $$ |$$ |           $$ |     $$ |      $$ /  $$ |$$ |$$  /
 $$$$$$$$ |$$$$$$$  |$$$$$\         $$ |     $$ |      $$ |  $$ |$$$$$  /
 $$  __$$ |$$  __$$< $$  __|        $$ |     $$ |      $$ |  $$ |$$  $$<
 $$ |  $$ |$$ |  $$ |$$ |           $$ |     $$ |      $$ |  $$ |$$ |\$$\
 $$ |  $$ |$$ |  $$ |$$$$$$$$\       $$$$$$$$$  |       $$$$$$  |$$ | \$$\
 \__|  \__|\__|  \__|\________|      \_________/        \______/ \__|  \__|

```

准备插入U盘，使用命令行刷入两个映像文件

```
# 进入U盘挂载的sda目录，一般都是这个sda1
root@XiaoQiang:~# cd /extdisks/sda1/

root@XiaoQiang:/extdisks/sda1# ls
System Volume Information                                               
openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-rootfs0.bin
TDDOWNLOAD                                                              xiaomi_config
ThunderDB                                                               
openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-kernel1.bin

# 刷入kernel1映像
root@XiaoQiang:/extdisks/sda1# mtd write openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-kernel1.bin kernel1
Unlocking kernel1 ...

Writing from openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-kernel1.bin to kernel1 ... 

# 刷入rootfs0映像
root@XiaoQiang:/extdisks/sda1# mtd write openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-rootfs0.bin rootfs0
Unlocking rootfs0 ...

Writing from openwrt-22.03.5-ramips-mt7621-xiaomi_mi-router-3g-squashfs-rootfs0.bin to rootfs0 ...     

# 设置nvram环境变量flag_try_sys1_failed
root@XiaoQiang:/extdisks/sda1# nvram set flag_try_sys1_failed=1
root@XiaoQiang:/extdisks/sda1# nvram commit

# 重启路由器
root@XiaoQiang:/extdisks/sda1# reboot
```

至此路由器开始工作灯闪烁，知道蓝灯常亮代表结束刷机过程。

最后使用网线将路由器LAN口和PC连接，即可通过`http://192.168.1.1`登入OpenWrt的管理页面。
