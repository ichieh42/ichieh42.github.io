---
layout: post
title: "Optimized official archlinux installation process"
---

希望能逼迫养成自己强制使用命令行和查看英文文档的习惯。

## 安装Arch
1. 预安装
    1. 下载arch镜像文件
    2. 验证文件完整性
    3. 使用命令行或者图形工具制作usb启动盘
    4. 从启动盘启动
        - 注意：在主板设置里禁用安全启动（Secure Boot: -> disable）
    5. 调整终端键盘布局和字体（可选）
    6. 确认安装的引导模式（可选）
        ```
        # 执行命令
        ls /sys/firmware/efi/efivars

        # 如果当前结果是BIOS
        # ls: cannot access '/sys/firmware/efi/efivars': No such file or directory
        # 如果当前结果是UEFI
        # 则会显示一些英文字母的单词表示对应的文件
        ```
    
    7. 连接互联网

        如果是使用网线的只需要执行命令验证下⬇
        ```
        # 执行命令
        ping baidu.com -c 4

        # 如果成功返回结果大体应该是这样的
        # PING baidu.com (39.156.66.10) 56(84) bytes of data.
        # 64 bytes from baidu.com (39.156.66.10): icmp_seq=1 ttl=48 time=45.7 ms
        # 64 bytes from baidu.com (39.156.66.10): icmp_seq=2 ttl=48 time=49.1 ms
        # 64 bytes from baidu.com (39.156.66.10): icmp_seq=3 ttl=48 time=45.9 ms
        # 64 bytes from baidu.com (39.156.66.10): icmp_seq=4 ttl=48 time=49.1 ms

        # --- baidu.com ping statistics ---
        # 4 packets transmitted, 4 received, 0% packet loss, time 3006ms
        # rtt min/avg/max/mdev = 45.682/47.423/49.068/1.639 ms
        ```
        
        如果想使用wifi设备连接网络可以使用`iwctl`命令
        ```
        # 执行命令进入iwctl命令模式
        iwctl
        # 在iwctl模式下可以查看自己的网卡设备
        device list
        # 扫描WiFi（使用wlan0这个设备）
        station wlan0 scan
        # 获取扫描之后的结果
        station wlan0 get-networks
        # 输出扫描的WiFi结果
        #                       Available networks
        # ------------------------------------------------------------------
        #        Network name          Security              Signal
        # ------------------------------------------------------------------
        #        WiFi1                 psk                   ****
        #        WiFi2                 psk                   ***
        #        ...
        #

        #使用命令连接想要的WiFi网络，并输入WiFi密码
        station wlan0 connect WiFi1
        ```

    8. 手动更新系统时间（可选）

        一般来说在实时环境中，systemd-timesyncd默认处于启用状态，一旦建立与互联网的连接，时间将自动同步。但你也可以使用命令手动同步
        ```
        timedatectl

        # 执行结果
        # Local time: Sat 2023-06-24 21:46:27 +08
        # Universal time: Sat 2023-06-24 13:46:27 UTC
        # RTC time: Sat 2023-06-24 13:46:27
        # Time zone: Asia/Singapore (+08, +0800)
        # System clock synchronized: no
        # NTP service: inactive
        # RTC in local TZ: no
        ```

    9. 对磁盘进行分区和挂载（略）
        建议新手在其他设备用图形化工具做好分区工作，我自己是使用命令行的
        ```
        # 使用fdisk等相关命令
        fdisk
        ```
        1. BIOS模式的默认是一个SWAP分区`/swap`和主分区`/mnt`
        2. UEFI模式的默认是一个SWAP分区`/swap`，一个主分区`/mnt`，一个EFI分区`/mnt/boot`

    10. 格式化分区（略）
            注意：我使用archlinux启动盘格式化的主分区`/mnt`为`ext4`格式，在其它系统查看分区是损坏的。
    11. 挂载分区（略）
2. 安装过程
    1. 设置软件商店镜像网站（可选）

        由于网络问题我们需要设置一些国内的镜像源优化下载速度
        ```
        # 执行命令
        vim /etc/pacman.d/mirrorlist
        ```

        这里加入一些截图

    2. 安装系统和基本软件

        ```
        pacstrap -K /mnt base linux linux-firmware grub efibootmgr vim iwd dhcpcd
        ```
        - 注意： 这里的`-K`选项表示使用已经安装的内核和驱动程序来构建新的根目录文件系统。可以节省时间和磁盘空间。

3. 配置系统
    1. 设置开机启动自动挂载分区
        ```
        genfstab -U /mnt >> /mnt/etc/fstab
        ```
    2. 进入新系统环境
        ```
        arch-chroot /mnt
        ```
        - 注意：下面的命令都是在新系统环境下执行的
    3. 设置时区
        ```
        ln -sf /usr/share/zoneinfo/Asia/Singapore /etc/localtime

        hwlock --systohc
        ```
    4. 设置语言和地区化（略）
    5. 主机名设置（可选）
    6. Initramfs（可选）
    7. 修改密码（可选）
        ```
        passwd
        # Current password:
        # 输入你想要设定的新密码
        ```
    8. 引导程序
        ```
        grub-install --target=x86_64-efi --efi-directtory=boot --bootloader-id=GRUB

        # 输出为
        # Installing for x86_64-efi platform.
        # Installation finished. No error reported.
        ```

4. 重新启动
```
reboot
```

5. 安装后
