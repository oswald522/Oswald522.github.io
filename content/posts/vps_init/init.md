---
title: VPS环境部署常用实践及操作命令（以Debian为例）
date: 2025-03-15T16:50:41+08:00
lastmod: 2025-06-16T10:00:00+08:00
draft: false
description: 一份从零开始的VPS（Debian）环境配置指南，涵盖系统安装、安全加固、性能优化到Docker部署的全流程，帮助你打造一个稳定、高效、安全的服务器环境。
featureimage: https://picsum.photos/seed/674acfad/1600/900.webp
showComments: true
tags:
  - 经验技术
  - Linux
  - VPS
  - Debian
  - 服务器部署
# slug: "series"
series: ["Linux部署教程"]
series_order: 1
---

## 👋 前言

VPS (Virtual Private Server)，我们可以亲切地称它为一台永不关机的“云端电脑” ☁️。如何从零开始，一步步将它配置成一个稳定、高效且安全的工作环境？这份笔记总结了我过往的实践经验，既是分享，也是个人备忘。

{{< alert icon="circle-info" >}}
本教程所有命令均在 **Debian 11/12** 系统下测试通过，理论上兼容 Ubuntu。建议在全新的、纯净的操作系统上操作。

约定说明：`{}` 及其中的内容代表您需要根据实际情况替换的变量。例如 `ssh {user}@{server_ip}`，若用户名为 `root`，IP为 `198.56.25.1`，则替换为 `ssh root@198.56.25.1`。
{{< /alert >}}

---

## 🗺️ 部署路线图

我们将按照以下步骤，由浅入深地完成服务器配置：

1. **新机开荒**：重装一个纯净的操作系统。
2. **基础配置**：更新系统、安装必备工具、校准时间。
3. **安全加固**：配置 SSH 密钥登录，修改端口，提高安全性。
4. **性能优化**：开启 BBR 加速，添加 SWAP 虚拟内存。
5. **应用环境**：安装 Docker 和 Docker-compose。
6. **最佳实践**：分享一些重要的安全好习惯。

---

## 🚀 第一步：新机开荒 (可选)

对于部分厂商（尤其国内）提供的系统镜像，可能包含一些不必要的监控或预装软件。通过 DD (Disk Dump) 的方式重装一个纯净的官方系统，是保证服务器干净的第一步。

> 💡 **何时需要DD？**
> 厂商不提供你想要的操作系统版本 (如 Debian 12)。
> 想彻底清除厂商的预装程序。

推荐脚本: {{< github repo="bin456789/reinstall" >}} (脚本内容已做基本审查)。

```shell
# 必须使用 root 账户执行
# 1. 更新源并安装基础工具
apt update && apt install -y curl wget

# 2. 下载DD脚本
# 国外VPS
curl -O https://raw.githubusercontent.com/bin456789/reinstall/main/reinstall.sh || wget -O reinstall.sh $_ 
# 国内VPS (使用镜像加速)
curl -O https://cnb.cool/bin456789/reinstall/-/git/raw/main/reinstall.sh || wget -O reinstall.sh $_

# 3. 运行脚本，例如安装 Debian 12
chmod a+x reinstall.sh && ./reinstall.sh -debian 12
```

**备选脚本**：{{< github repo="leitbogioro/Tools" >}}，可根据需要选用。需要注意的是系统重启安装过程可能需要一定的时间，因此执行命令脚本重启后需要等待时间才能使用ssh连接（具体时间根据服务器性能配置而定）。

---

## 🛠️ 第二步：基础系统配置

系统装好后，我们来做一些基础的初始化设置。

### 1. 更新系统与软件

```shell
# 养成备份好习惯，备份默认软件源
cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 全面更新系统
apt update && apt upgrade -y && apt dist-upgrade -y && apt full-upgrade -y && apt autoremove -y

# 安装常用必备工具
apt install -y sudo curl wget nano zsh git axel vim htop bat
```

### 2. 校准时区与主机名（可选）

```shell
# 将时区设置为上海
sudo timedatectl set-timezone Asia/Shanghai
# 修改主机名 (例如: my-awesome-vps)
hostnamectl set-hostname {my-awesome-vps}
echo "127.0.0.1 {my-awesome-vps}" >> /etc/hosts
# 查看当前时区和时间
timedatectl
```

> 💡 **提示：**
>
> * 查看所有可用时区: `timedatectl list-timezones`
> * 选择一个易于辨识的主机名，方便多台服务器管理。

---

## 🔐 第三步：安全加固 - SSH 配置

这是保障服务器安全**最重要**的一步！强烈推荐使用密钥登录，彻底告别弱密码风险。

### 1. 🔑 生成并配置 SSH 密钥

**本地电脑操作**：打开你的终端（不是VPS），生成密钥对。

```shell
# -t 指定算法, ed25519 是目前推荐的算法
# -C 添加注释，通常用邮箱或用户名，方便识别
ssh-keygen -t ed25519 -C "{your_email@example.com}"
```

命令执行后，会在你的本地电脑 `~/.ssh/` 目录下生成 `id_ed25519` (私钥) 和 `id_ed25519.pub` (公钥) 两个文件。

**服务器操作**：将**公钥**内容添加到服务器的信任列表。

```shell
# 1. 登录你的VPS
# 2. 备份现有的 authorized_keys 文件 (如果有的话)
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak

# 3. 将你的公钥内容完整地粘贴进去
# 方法一：使用 echo (推荐)
# 将 cat ~/.ssh/id_ed25519.pub 在本地电脑查看到的公钥内容替换下面的一整行
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMz2IFBt+XSSXX+FX5FlX your_email@example.com" >> ~/.ssh/authorized_keys

# 方法二：使用编辑器
# nano ~/.ssh/authorized_keys
# (将公钥内容粘贴进去，保存并退出)
```

### 2. 🚪 更改默认 SSH 端口

修改 SSH 端口通常有以下几个主要原因：

* 增强安全性：SSH 服务默认使用的 22 端口是攻击者经常扫描和尝试攻击的目标。通过将端口修改为一个不常见的数值，可以减少自动攻击和暴力破解的风险，因为攻击者通常会首先针对常见的默认端口进行攻击。例如，如果攻击者使用自动化工具扫描大量服务器，这些工具可能主要集中在 22 端口。而修改了端口后，就降低了被这类工具轻易发现和攻击的可能性。

* 减少误连接和非法访问尝试：一些网络环境中可能存在大量的随机连接请求或非法访问尝试，针对默认的 22 端口。更改端口可以减少这类无意义的连接请求。假设您的服务器处于一个公共网络环境中，经常会收到大量的随机连接尝试，其中很多是针对常见端口的。修改 SSH 端口可以减少这类不必要的干扰。默认的 22 端口是自动化扫描和攻击的首要目标。修改为一个不常用的高位端口（如 10000-65535 之间）能有效减少恶意尝试。

```shell
# 将默认的 22 端口修改为你选择的端口，例如 55520
sudo sed -i 's/^#\?Port 22.*/Port {55520}/g' /etc/ssh/sshd_config
```

### 3. 🚫 禁用密码和 Root 登录 (强烈建议)

```shell
# 编辑 SSH 配置文件
sudo nano /etc/ssh/sshd_config
```

找到并修改以下几项，确保它们是如下状态：

```ini
PermitRootLogin no          # 禁止 root 用户直接登录
PasswordAuthentication no   # 禁止使用密码登录
PubkeyAuthentication yes    # 确保公钥认证是开启的
```

### 4. ✅ 重启并验证

以下的命令和操作系统相关，有一些操作系统是 ssh 而不是sshd。

```shell
# 重启 SSH 服务使其生效
sudo systemctl restart sshd
```

{{< alert icon="triangle-exclamation" type="warning" >}}
**🔥 关键操作！**
在断开当前连接之前，**必须**打开一个**新的终端窗口**，使用你的**新端口**和**密钥**尝试登录：
`ssh -p {55520} {user}@{server_ip}`
**确认可以成功登录后**，再关闭旧的终端窗口。否则配置错误会导致你再也无法连接服务器！
{{< /alert >}}

---

## ⚡ 第四步：性能优化

这个功能通过调整各种系统和网络参数来优化服务器的性能。

### 0. 系统参数调优

这个功能通过调整各种系统和网络参数来优化服务器的性能。

实现方法：

1. 内核参数调整：例如，增加 TCP 缓冲区大小、修改系统队列长度等，这些改变有助于提高网络吞吐量和减少延迟。
2. 性能优化：安装和配置 Tuned 和其他系统性能优化工具来自动调整和优化服务器的运行状态。
3. 资源限制：例如，设置文件打开数量的限制，这可以防止某些类型的资源耗尽攻击。

通过这些功能，你的服务器不仅能够更有效地管理资源，还能提高对外部威胁的防护能力，保障系统稳定运行。

`bash <(wget -qO- https://raw.githubusercontent.com/jerry048/Tune/main/tune.sh) -t`

### 1. 🚄 BBR 网络加速

BBR 是 Google 提出的一种新型拥塞控制算法（Bottleneck Bandwidth and RTT），全称为瓶颈带宽和往返传播时间。

在 Linux 系统中，BBR 主要有以下特点和作用：

* 提高网络性能：它可以显著提高吞吐量和降低 TCP 连接的延迟，使数据传输更加高效。
* 适应不同网络环境：适合高延迟、高带宽的网络链路，以及慢速接入网络的用户，能在一定丢包率的网络链路上充分利用带宽，并降低网络链路上的缓冲区占用率从而降低延迟。
* 优化拥塞控制：BBR 改变了传统基于丢包反馈的拥塞控制机制，通过精确测量往返传播时间（RTT）和瓶颈带宽等参数来更有效地控制数据发送速率，避免了传统算法中因单纯丢包判断拥塞而导致的带宽利用率不高和端到端延迟大等问题。
* 提升网络稳定性：有助于减少网络拥塞和数据包丢失，提高网络的稳定性和可靠性。

> PT 玩家力荐的 **BBRx** 版本，在原版 BBR 基础上调整了关键参数，实测效果更佳。

```shell
# 一键安装 BBRx 并优化系统内核
bash <(wget -qO- https://raw.githubusercontent.com/jerry048/Tune/main/tune.sh) -x

# 安装完成后，重启 VPS 使新内核生效
sudo reboot
```

**验证 BBR 是否开启成功**：

```shell
# 重启后重新登录，执行命令
lsmod | grep bbr

# 理想情况下，应看到 tcp_bbrx 和 tcp_bbr
# 如果只看到 tcp_bbr，可以稍等几分钟再次重启 sudo reboot 试试
```

**备选 BBR 方案** (如果你不偏好 BBRx):

```shell
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
# 同样需要重启生效
```

### 2. 💾 添加 SWAP 虚拟内存

SWAP 就像是物理内存(RAM)的“备用空间”。当 RAM 不足时，系统会将不常用的数据临时存放到 SWAP（一块硬盘空间）中，避免因内存耗尽而导致程序崩溃。对于内存较小的 VPS 来说非常必要。

```shell
# 使用一键脚本添加 SWAP
wget -O swap.sh https://raw.githubusercontent.com/yuju520/Script/main/swap.sh && chmod +x swap.sh && ./swap.sh

# 查看当前内存和 SWAP 情况
free -m
```

---

## 📦 第五步：应用环境 - 安装 Docker

Docker 是现代应用部署的基石，它能将应用及其依赖打包成一个轻量、可移植的“容器”，实现环境隔离和快速部署。

### 1. 安装 Docker

Docker 从 18.06.0-ce 版本就开始自带 Docker Compose 工具，因此，安装docker会直接安装docker compose插件。以后看到的相关教程出现的命令`docker-compose`调整为`docker compose`，即中间去掉`-`符号。

```shell
# 🌐 非大陆服务器 (官方脚本)
curl -fsSL https://get.docker.com | bash

# 🇨🇳 大陆服务器 (使用国内镜像加速)
curl https://install.1panel.live/docker-install | bash
```

### 2. 配置与验证

```shell
# 设置 Docker 开机自启动
sudo systemctl enable docker

# 查看 Docker 版本
docker -v

# 验证 Docker Compose (新版 Docker 已内置)
docker compose version
```

### 3. (可选) 卸载 Docker

如果需要卸载，可以使用以下命令：

```shell
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo apt-get remove docker docker-engine
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

---

## 🧑‍💻 第六步：安全与最佳实践 (重要!)

养成良好的使用习惯，能让你的服务器免受许多不必要的风险。

* ✅ **使用普通用户**：日常操作避免直接使用 `root`，创建一个普通用户，在需要管理员权限时使用 `sudo`。 (TODO: 待补充创建用户教程)
* 🐳 **隔离未知应用**：对于非自己编写或不熟悉的应用（如 WordPress），一律使用 Docker 部署，最大限度地隔离，减少攻击面。
* 🕵️ **审查第三方脚本**：执行任何来路不明的一键脚本前，务必先用 `cat` 或 `nano` 查看其内容，警惕埋雷。也可以让 AI 帮忙初步审查。
* 🛡️ **慎用管理面板**：尽量避免安装宝塔等闭源或权限过高的服务器管理面板，它们会显著扩大攻击面。
* 🔑 **保护私钥**：为你的 SSH 私钥文件设置一个强密码，即使私钥文件泄露，没有密码也无法使用。

---

## 🎉 结语

恭喜！到这里，你的新 VPS 已经完成了从“毛坯房”到“精装修”的全部过程。它现在是一个相对安全、高效且配置了现代化应用环境的平台。

关于如何使用 Docker 部署各类常用应用，篇幅所限，我们下期再见！

---

## 📚 附录：参考资料

* [Linux 部署及安全实践指南](https://linux.do/t/topic/468841)
* [【配置优化】我拿到VPS服务器必做的那些事](https://linux.do/t/topic/160305)
