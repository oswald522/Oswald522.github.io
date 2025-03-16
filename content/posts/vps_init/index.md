---
title: "VPS环境部署常用实践及操作命令"
date: 2025-03-15T16:50:41+08:00
lastmod: 2024-03-15T16:50:41+08:00
draft: false
description: ""
featureimage: "https://picsum.photos/seed/674acfad/1600/900.webp"
showComments: true
---

<!-- > {{< alert >}}adfasdf{{< /alert >}} -->

## 前言

VPS或者说服务器，可以理解为一台永不停机的"电脑"。如何配置部署好 VPS 环境，前期积累一些经验，现在将这些经验总结分享，同时也用于个人记录。

{{< alert icon="circle-info" >}}
教程设计的命令均在 Debian 11/12 下运行，理论上兼容 Ubuntu。建议在全新的操作系统下执行，仅仅针对于小白。
此处约定: {} 及其括起来的内容为根据你实际情况需要替换的文本内容，括起来的内容为说明，如 `ssh {user}@{server ip}` 为 ssh 连接服务器的命令，假设用户为 `root`, 服务器 IP 为 `198.56.25.1`, 则为 `ssh root@198.56.25.1`.
{{< /alert >}}

## 具体步骤

### 新机器开荒（可选）

1. **DD 纯净系统** (可选):对于国内厂商的机器，DD 纯净系统几乎是必须做的，以彻底删除阿里云等的机器里面的监控程序.其余的，如果厂商不提供 Debian 11 / 12 镜像或者镜像过于老旧，也可以通过 DD 安装上. 推荐脚本: GitHub - leitbogioro/Tools: Something about tools, 我已简单审查脚本内容.

```shell
# 安装操作系统属于高级权限，必须使用root账户。
apt update && apt install curl wget
# 国外VPS
wget --no-check-certificate -qO InstallNET.sh 'https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh'  
# 国内VPS
wget --no-check-certificate -qO InstallNET.sh 'https://gitee.com/mb9e8j2/Tools/raw/master/Linux_reinstall/InstallNET.sh' && chmod a+x InstallNET.sh
chmod a+x InstallNET.sh && ./InstallNET.sh -debian
```

### 更新配置源

1. 配置更新源

```shell
# 养成备份的好习惯:
mv /etc/apt/sources.list /etc/apt/sources.list.bak
```

1. 配置更新源

```shell
apt update && apt upgrade -y && apt dist-upgrade -y && apt full-upgrade -y && apt autoremove -y
apt install zsh git curl axel vim htop bat 
hostnamectl set-hostname {new name}
echo "127.0.0.1 {new name}" >> /etc/host
```

### 配置SSH

推荐使用**公钥验证登录**，抛弃密码和 fail2ban 吧～当然，不方便用公钥的，密码一定要足够强.

```shell
ssh-keygen -t ed25519 -C "{密钥注释}"
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak
cat <<'EOF' > ~/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMz2IFBt+FWJc750FHf5hFt7UeXqF4gCet7td+FX5FlX Test
EOF
```

其他需要修改的配置，如修改高位端口号，禁用Root用户登录，禁用密码登录等。
然后重启SSHD服务`systemctl restart ssh.service`

> 注意事项: (千万不要关掉当前终端)请务必另外启动一个终端使用公钥尝试登录进行测试，以验证配置是否成功,成功连接再执行下一步.  

## 其他注意⚠️事项

- 尽量不要日用 root, 养成使用普通账户然后必要时 sudo 的习惯 (TODO)
- 不要使用 PHP 等 , 以及相关项目，如 Wordpress, 实在太多安全问题了，非要使用请使用 Docker 隔离。个人对于非自己编写或审查的东西一律 Docker 伺候，最大限度减少攻击面.
- 谨慎使用第三方脚本 , 脚本里埋雷不是没见过，执行前务必详细检查，让 AI 帮忙也是可以的.
- 尽量不要使用服务器管理面板，尤其是宝塔等闭源产品，扩大攻击面，自身权限过高.
- 尽量不使用密码登录 SSH, 请使用密钥登录，且私钥务必设置密码以免被盗用.

## 结语

到这里，新机器就开荒完毕了。有关各类常用应用，如 Docker 等的部署也是常见的，但本文篇幅已经很长了，让我们下期再见！

## 附录

- [Linux 部署及安全实践指南](https://linux.do/t/topic/468841)
