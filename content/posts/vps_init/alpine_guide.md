---
title: Alpine基础安装教程
description: "Alpine的基础安装配置教程，包含各部分的操作和存储持久化。"
date: 2025-04-22T15:05:13+08:00
Lastmod: 2025-04-22T15:05:13+08:00
draft: false
showComments: true
featureimage: https://picsum.photos/seed/7a639a/1600/900.webp
tags:
  - 操作系统
  - 服务器
  - 技术教程
series: ["Alpine操作系统"]
series_order: 1
seriesOpened: true
---

## Alpine基础环境配置

版本的选择，默认使用 standard，extend 版本可做便携版本使用。
用户手册：[Alpine User Handbook](https://docs.alpinelinux.org/user-handbook/0.1a/index.html)  
官方WIKI：[Alpine Linux WIKI](https://wiki.alpinelinux.org/wiki/Installation)

### 安装

安装的实际逻辑是通过 `setup-alpine` 脚本去调用其他功能的脚本进行配置，可以通过 vi 查看脚本。如果某个部分安装失败，可退出后单独再次执行。通过镜像文件，进入系统引导，默认用户名 root，密码为空。

1. 运行 `setup-alpine` 进行安装，提示键盘选择，键入：`us us` 即可；如果键入一个 us ，会再次提示，因为键盘设置需要地区和键位两个值。
2. 设置主机名，默认 localhost ，可自定义
3. 网络初始化（推荐使用DHCP，安装后手动修改）
    1. 默认网卡 eth0，默认即可，选用 eth0
    2. dhcp 服务，默认即可，启用 DHCP，自动获取 IP
    3. 是否配置静态IP，默认no，与上一步 dhcp 互斥，默认即可
4. root 密码配置
5. 配置时区，键入 `？` 可查看可选参数，键入 `PRC` ；PRC 是国内的简称，与 Asia/Shanghai 效果一致
6. 配置代理，默认 none 即可
7. 配置NTP，默认 chrony
8. 配置源，键入数字即可，49 阿里云，52 北邮，60 东软
9. 创建本地用户，默认no
10. 配置 ssh 服务，默认 openssh
    1. root 登录配置，默认使用密钥，键入 `yes` ，允许密码登录
11. 硬盘安装，对话前会打印出硬盘列表，默认 none ，键入 `sda` 选择硬盘
    1. 根据磁盘格式，选用需要的文件系统或方案，键入 `sys`
    2. 是否写入变更，键入 `y`
12. 安装完成，进行重启

### 安装后配置

#### 配置源，安装时仅配置了主要仓库

```bash
# 仓库配置路径 /etc/apk/repositories
http://mirrors.aliyun.com/alpine/v3.18/main# 默认配置，主要的源
http://mirrors.aliyun.com/alpine/v3.18/community# 社区源，默认未开启
# edge 源，拥有很多第三方应用
http://mirrors.aliyun.com/alpine/edge/community
http://mirrors.aliyun.com/alpine/edge/testing
```

#### 网络配置

- `setup-interfaces` 配置网络，setup-interfaces wlan0 配置无线网卡，setup-interfaces -a 使用 dhcp 获取IP；手动配置，配置文件路径 `/etc/network/interfaces`

    ```bash
    # DHCP 模式配置信息
    auto lo
    iface lo inet loopback
    
    auto eth0
    iface eth0 inet dhcp
    # 静态模式配置信息
    auto lo
    iface lo inet loopback
    
    auto eth0
    iface eth0 inet static
        address 192.168.0.147
        netmask 255.255.255.0
        gateway 192.168.0.1
    # 启用网络
    rc-service networking start
    # 开机自启
    rc-update add networking boot
    # 查看所有服务状态
    rc-status
    ```

- 配置 dns，`setup-dns`，与手动编辑 `/etc/resolv.conf` 效果一致

    ```bash
    nameserver 223.5.5.5
    ```

- 配置时区 `setup-timezone`，安装时已经配置，可略过；亦可手动编辑

    ```bash
    # 安装时区，可能需要 tzdata 包
    install -Dm 0644 /usr/share/zoneinfo/Asia/Shanghai /etc/zoneinfo/Asia/Shanghai
    # 配置环境变量
    export TZ='Asia/Shanghai' 
    echo "export TZ='$TZ'" >> /etc/profile.d/timezone.sh
    ```

- 配置 ntp 服务 `setup-ntp`，系统安装时，已经安装应用；手动变更ntp服务器

    ```bash
    # 配置文件 /etc/chrony/chrony.conf
    sed -i "s|pool.ntp.org|ntp.aliyun.com|g" /etc/chrony/chrony.conf
    # 重启服务，默认已开机启动
    rc-service chronyd restart
    ```

#### 配置 bash

默认使用的是 ash

```bash
# 安装 bash
apk add bash bash-completion
# 替换 ash
sed -i 's/ash/bash/g' /etc/passwd
# 按需配置 bashrc，需手动初始化，所以写成系统变量，以遍开机加载
cat > /etc/profile.d/self.sh <<eof
alias update='apk update && apk upgrade'
export HISTTIMEFORMAT="%d/%m/%y %T "
export PS1='\u@\h:\W \\$ '
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
source /usr/share/bash-completion/bash_completion
eof
```

### 常用软件的配置

1. Docker安装：直接使用具有管理员权限的用户`curl -fsSL https://get.docker.com | sh`实现安装
2. K8S安装

## 参考资料

1. [Alpine官方教程](https://wiki.alpinelinux.org/wiki/Installation)
2. 虫祇，[Linux从0到X / Alpine](https://www.cnblogs.com/chongxs/p/17982225/alpine-docker-env)
