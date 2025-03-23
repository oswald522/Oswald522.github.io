---
title: "[转载]WSL(Windows Subsystem for Linux) 教程"
description: ""
date: 2021-03-23T12:02:45+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/0fc867/1600/900.webp"
tags: ""
# series: "建站技术"
---

## 前言

本文基于BiliBili [**UP主技术爬爬虾**](https://space.bilibili.com/316183842)的视频 [超详细的WSL教程：Windows上的Linux子系统](https://www.bilibili.com/video/BV1tW42197za) 整理总结而来, 感谢UP的辛苦付出, 欢迎大家一键三连.

## 简介

### **什么是 WSL？**

WSL（Windows Subsystem for Linux）是微软在 Windows 10 及更高版本中引入的功能，允许用户在 Windows 系统上直接运行 Linux 环境。它提供了一个兼容层，使得 Linux 二进制文件能够在 Windows 上无缝运行，而无需虚拟机或双系统启动。

### **WSL 的版本**

1. **WSL 1**：
   - 首次发布，通过兼容层将 Linux 系统调用转换为 Windows 调用。
   - 优点：轻量级，与 Windows 文件系统完全兼容。
   - 缺点：性能较低，不支持所有 Linux 内核功能。

2. **WSL 2**：
   - 基于轻量级虚拟机技术，运行完整的 Linux 内核。
   - 优点：性能大幅提升（尤其是文件 I/O），支持 Docker 等容器化工具。
   - 缺点：需要更多系统资源，Windows 和 Linux 文件系统之间访问速度略有延迟。

### **WSL 的主要优势**

1. **无需虚拟机**：
   - 直接在 Windows 上运行 Linux 环境，无需额外安装虚拟机软件（如 VirtualBox、VMware）。
2. **无缝集成**：
   - 支持在 Windows 和 Linux 之间共享文件系统，方便文件操作。
   - 可以在 Windows 终端（如 PowerShell、CMD）中直接调用 Linux 命令。
3. **开发效率提升**：
   - 支持运行 Linux 开发工具（如 Bash、Python、Node.js、GCC 等），适合跨平台开发者。
4. **支持 Docker**：
   - WSL 2 支持在 Linux 环境中运行 Docker 容器，非常适合 DevOps 和容器化开发。

---

## 安装

1. 开启CPU虚拟化
   - 一般默认打开(可通过任务管理器/性能查看),若未打开,可进入BIOS,启用英特尔VMX虚拟化平台(Intel Virtualization Technology).如果是AMD CPU,需要开启AMD-v的开关.
2. 开启Windows功能下的**适用于Linux的Windows子系统**和**虚拟机平台(Hyper-V)**.
   - 在任务栏搜索"Windows功能"可找到
   - 启用后按提示重启电脑
3. 管理员身份运行CMD. 使用`wsl --install`下载安装Linxu子系统.
4. 在CMD当前标签页右边下拉选箭头下选择要启动的Linux子系统即可启动
5. 若提示缺少字体,可下载字体后双击安装或手动复制到`C:\Windows\Fonts`目录下.

---

## WSL常用命令

1. 安装Linux子系统: `wsl --install [操作系统名称]`[^1]
   - 国内网络可使用`wsl -install --web-download`减少下载失败的可能性
   - os_name不指定将会下载默认系统,当前默认系统为Ubuntu
2. 获取可安装Linux子系统列表: `wsl --list --online`
3. 查询Linux子系统列表: `wsl --list -v`
4. 设置默认子系统: `wsl --set-default <子系统名称>`
5. 启动Linxu子系统: `wsl -d <子系统名称>`
6. 卸载Linux子系统: `wsl --unregister <子系统名称>`
7. 备份Linxu子系统: `wsl --export <子系统名称> <生成的备份文件的名称>`
   - 默认导出到当前操作目录下
8. 导入备份的Linux子系统: `wsl --import <新子系统名称> <新子系统要存放的目录> <子系统备份文件路径>`
   - 导入成功后在"新子系统要存放的目录"下会出现一个.vhdx后缀名的文件,这是Hyperv的镜像文件, 子系统的全部数据都存储在该文件中.
9. 关闭wsl中所有正在运行的Linxu子系统: `wsl --shutdown`

---

## 实用功能

1. 在Windows中查看子系统文件
   - `Win + E`打开文件资源管理器, 左下角`Linxu`菜单下显示的就是各个子系统的文件列表,可在Windows系统中直接对子系统文件进行增删改查操作
   - 使用命令`df -h`显示所有挂载卷, 在子系统中可直接操作挂载卷中的Windows文件
   - 如果子系统中需要执行高性能IO工作,建议将文件直接复制到子系统(文件列表)中,挂载的性能较低.
2. 命令混用
   - 在Windows中直接运行Linux命令
      - 甚至可以混写,如`Get-ChildItem | wsl grep Video`

   - 在Linxu子系统中可直接运行Windows程序
      - 例如使用`notepad.exe <文本文件名>`会直接使用Windows记事本打开目标文本文件
      - 例如使用`explorer.exe <资源目录>`会直接使用Windows文件资源管理器打开资源目录
3. WSLg - 将Linux子系统中带UI的应用程序以Windows窗口的形式打开
   - 在Linxu子系统中利用命令直接打开应用程序即可
4. 显卡直通
   1. 在Linux子系统中使用命令`nvidia-smi`可查看显卡信息, 显卡已经配置好驱动,CUDA. 在子系统中可直接识别显卡并可使用程序调用

---

## 高级配置

在旧版本中,要配置Linxu子系统的网络等信息需要自行编写wsl配置来完成,新版本下可以通过`WSL Setting`应用程序的设置功能来在图形化页面修改.

- 使用Windows搜索功能搜索`WSL`可找到`WSL Setting`应用程序.
- `WSL Setting`应用程序中还包含集成VSCODE和Docker的文档.

关于配置文件简单介绍如下,完整的高级配置信息参考[完整文档](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config):

- `.wslconfig`用于在 WSL2 上运行的所有已安装发行版中配置全局设置
- `wsl.conf`用于为在 WSL1或 WSL2 上运行的每个 Linux 发行版按各个发行版配置本地设置。
- 8秒规则: 修改配置后必须等待Linux子系统完全停止运行(使用`wsl --shutdown`命令)后并等待8秒重启才生效

## WSL2入门配置指南

最近不断看到有网友询问关于WSL2的问题，因此决定写一篇详细的入门配置教程，手把手教你如何配置WSL2，从入门到熟练使用。

通过这篇文章，你将学会：
>
> 1. 在你的Windows电脑上安装Ubuntu到WSL2中；
> 2. 迁移你的WSL2系统镜像，让系统随时可用；
> 3. 优化WSL2系统设置，让其可以后台启动并高效运行；
> 4. 设置Ubuntu内的应用开机自启。
> 5. 其他的一些安全设置选项

### 1.安装Ubuntu到WSL2

默认选择Ubuntu系统，因为它是唯一官方支持GPU加速的发行版，非常方便。

```bash
wsl --install
```

如果你想尝试其他发行版，可以通过以下命令查看并安装：

```bash
wsl --list --online 或 wsl -l -o
wsl --install -d <发行版名称>
```

当然，你也可以安装多个发行版。默认情况下，系统会安装在C盘上，但你可以迁移到其他位置。

### 2.导出和迁移系统

首先，导出现有的系统：

```bash
wsl --export Ubuntu E:\Ubuntu2.tar
```

然后取消挂载当前系统：

```bash
wsl --unregister Ubuntu
```

最后，将系统重新挂载到新的位置：

```bash
wsl --import Ubuntu E:\wsl2\Ubuntu E:\Ubuntu.tar
```

### 3.1.限制WSL2的资源占用

为了防止WSL长期占用系统资源，你可以通过配置.wslconfig文件来限制WSL的内存、CPU和交换分区大小。

1. 打开Windows资源管理器，地址栏输入 `%UserProfile%` 回车。在该目录下创建一个名为.wslconfig的文件，写入以下内容（以8GB内存电脑为例，分配2GB内存和2个CPU线程给WSL，设置4GB交换分区）：

```ini
[wsl2]
memory=2GB
swap=4GB
processors=2
localhostForwarding=true
```

2. 执行以下命令关闭并重新启动WSL：

```bash
wsl --shutdown
```

如果你不想限制WSL2的内存占用，可以通过定时任务定期清理内存。编辑crontab设置每小时释放一次内存：

```bash
crontab -e
0 */1 * * * echo 3 > /proc/sys/vm/drop_caches
```

### 3.2.设置WSL2后台启动

1. 在你的WSL2的Debian或Ubuntu中执行以下命令，允许用户执行所有命令而不需要密码：

```bash
sudo bash -c "echo '$USER ALL=(ALL) NOPASSWD: ALL' >/etc/sudoers.d/$USER"
```

1. 在Windows开机启动文件夹中添加一个以.vbs结尾的文件，内容如下：

```vbs
set ws=wscript.CreateObject("wscript.shell")
ws.run "wsl -d Ubuntu", 0
```

要快速进入开机启动文件夹，可以按Windows+R，然后输入 `shell:startup`。你可能还会用到 `taskschd.msc`。

### 4.设置Ubuntu内的应用开机自启

在WSL系统内新建并添加以下内容：

```bash
vi /etc/wsl.conf
```

内容如下：

```ini
[boot]
systemd=true
```

然后开启Ubuntu的开机启动服务：

```bash
systemctl status rc-local
```

并添加以下内容：

```bash
cat <<EOF >/etc/rc.local
# !/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
exit 0
EOF
```

执行以下命令使脚本可执行并启用：

```bash
chmod +x /etc/rc.local
systemctl enable --now rc-local
```

无视警告即可：

```bash
systemctl status rc-local.service
```

修改 `/etc/rc.local` 里的内容即可让你的Ubuntu应用开机启动。

## 5.其他的一些必要配置

### 设置root密码，开机必备

```bash
sudo passwd root
```

### 更新系统

```bash
su -
apt-get update && apt-get upgrade
```

### WSL2 Ubuntu 安装openssh-server

```bash
sudo apt update
sudo apt install openssh-server
```

### WSL2 启用systemd

在/etc目录下新建wsl.conf配置文件，并编辑该配置文件：

#### Ubuntu

```bash
sudo vi /etc/wsl.conf
```

输入内容：

```ini
[boot]
systemd=true
```

#### Windows

在 Windows PowerShell(管理员)中运行：

```bash
wsl --shutdown
```

再重新打开 Ubuntu，使 WSL 彻底重新启动以便启用 systemd。然后在 WSL 中运行：

### 让你的ssh开机自启

```bash
systemctl enable ssh
systemctl start ssh
```

请注意你的ssh的登录安全，按照你的安全习惯配置即可。

### 进一步学习

学完以上基础内容后，你可能会感兴趣以下内容：

- 如何在WSL2内运行大模型，如部署本地的ollama或吾皇的GPT；
- 如何让你的本地模型在外网随时可访问，如部署Tailscale或ZeroTrust；
- 还有哪些有趣的场景可以探索？

### 参考资料

- [让WSL开机启动，后台运行，以减少唤醒时间](https://www.cnblogs.com/wswind/p/17201979.html)
- [微软WSL安装官方指南](https://learn.microsoft.com/zh-cn/windows/wsl/install)
- [新版本下如何通过外部网络访问wsl](https://www.sxrhhh.top/blog/2023/12/14/connect-to-wsl/#_4)
- [WSL(Windows Subsystem for Linux) 教程](https://linux.do/t/topic/506768)
