---
title: "[转载]Linux 部署及安全实践指南"
description: "Linux常见部署命令记录"
date: 2024-03-30T08:16:25+08:00
Lastmod: 2025-03-30T08:16:25+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/629600/1600/900.webp"
tags: ["经验技术","转载教程"]
series: "Linux部署"
---

{{< alert >}}
以下内容转载来自于 [https://linux.do/t/topic/468841](https://linux.do/t/topic/468841)
{{< /alert>}}


> 本贴源自 <https://www.nodeseek.com/post-25170-1> , 但不能修改比较麻烦, 过了快两年也有需要更新的地方, 故重写.

- Current version: v0.1.0
- Last updated: 2025.3.3 19:11:00, UTC+8

## 前言

接触 VPS 也五年多了, 对于如何部署 VPS 环境, 同时保护好自己的 VPS, 也总结出一些经验, 于是写个帖子分享一下, 集思广益.

个人相对排斥第三方闭源工具, Nginx 等的配置文件熟悉后基本都是手搓然后机机相传.
值得提一嘴: 很多新手用的**宝塔**, **1Panel**, etc, 包括我一开始也是, 方便是方便, 代价也很大. 面板本身权限太高, 炸个零日漏洞就是大问题, 更不用说前者闭源, 背后干啥也不知道, 会收集服务器信息上传之类的传闻也是有的, ~~更不用说所谓开心版了~~.

本文当中的命令, 建议在全新的系统中执行, 特别是新手吃不准命令的含义的时候. 当然, 如果是老手, 一看就懂了, 我也只是总结一下.

要完全做到安全是不可能的, 绝大多数人也不是网络安全从业者, 本帖主要起扫盲作用, 别犯会被脚本小子利用的问题就好.

以下教程涉及的命令均在 **Debian** 11 / 12 内验证, 理论兼容 Ubuntu. ~~不会吧, 你还在用 CentOS?~~ 全部命令非特别说明均默认在 **root** 用户下执行.

**此处约定**: `{}` 及其括起来的内容为根据你实际情况需要替换的文本内容, 括起来的内容为说明, 如 `ssh {user}@{server ip}` 为 ssh 连接服务器的命令, 假设用户为 `root`, 服务器 IP 为 `114.5.1.4`, 则为 `ssh root@114.5.1.4`.

## 0. 新机器开荒

### 0.1. DD 纯净系统 (可选)

对于国内厂商的机器, DD 纯净系统几乎是必须做的, 以彻底删除阿里云等的机器里面的监控程序.
其余的, 如果厂商不提供 Debian 11 / 12 镜像或者镜像过于老旧, 也可以通过 DD 安装上.

另: Debian 12 资源占用并不多, 384M 的内存够了.

推荐脚本: <https://github.com/leitbogioro/Tools>, 我已简单审查脚本内容.

```sh
# 不会有系统没装 wget 吧, 没有就 `apt update && apt install wget` 一下
wget --no-check-certificate -qO InstallNET.sh 'https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh'
# 如果是中国大陆的机器, 用下面这个
# wget --no-check-certificate -qO InstallNET.sh 'https://gitee.com/mb9e8j2/Tools/raw/master/Linux_reinstall/InstallNET.sh' && chmod a+x InstallNET.sh

chmod a+x InstallNET.sh

# 不加参数 `-debian` 时默认安装当前系统, 但本文基于 Debian
# 不管你原来的系统是什么, 一律安装 Debian 最新稳定版本 (当前为 Debian 12 Bookworm)
./InstallNET.sh -debian
```

基本上脚本会处理好所有事情, 等待提示你 `reboot` 后输入 `reboot` 回车重启即可. 期间可打开 VNC 看进度.

安装后默认密码见官方说明: [default-configurations-of-ssh-or-rdp-service](https://github.com/leitbogioro/Tools?tab=readme-ov-file#default-configurations-of-ssh-or-rdp-service)

[color=red]**🥰注意事项**[/color]

- 安装完毕, 重新 SSH 连接机器时, 提示 SSH 公钥不匹配

  自行打开文件 "C:\Users\{你的用户名}\.ssh\known_hosts", 删除原来的 (搜索你的服务器 IP, 删掉那行就好).

- [color=red]**务必更改默认 root 密码**[/color]

  ```sh
  passwd root
  ```

[color=red]**☠风险提示**[/color]

- 部分服务商 **不允许** DD 系统.
- 如果你的机器网络配置相当奇怪脚本没认出来的话, 可能导致机器失联.
- 部分机器硬盘配置比较怪的, 也可能出问题, 例如大盘鸡之类的. 不怕折腾就相信脚本, 就算出问题控制台重装一下就行, 否则将就用吧. 欢迎分享折腾后的经验!

### 0.2. 更新

#### 0.2.1. 配置更新源

养成备份的好习惯:

```sh
mv /etc/apt/sources.list /etc/apt/sources.list.bak
```

<details>
<summary>对于 Debian 12, 境外机器, 使用官方源</summary>

```sh
cat <<'TEXT' > /etc/apt/sources.list
deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
deb http://deb.debian.org/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-backports main contrib non-free non-free-firmware
deb https://deb.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src https://deb.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
TEXT
```

</details>

<details>
<summary>对于 Debian 12, 阿里云机器, 使用内网域名 (腾讯云之类的依葫芦画瓢)</summary>

```sh
cat <<'TEXT' > /etc/apt/sources.list
deb http://mirrors.cloud.aliyuncs.com/debian/ bookworm main contrib non-free non-free-firmware
deb-src http://mirrors.cloud.aliyuncs.com/debian/ bookworm main contrib non-free non-free-firmware
deb http://mirrors.cloud.aliyuncs.com/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src http://mirrors.cloud.aliyuncs.com/debian/ bookworm-updates main contrib non-free non-free-firmware
deb http://mirrors.cloud.aliyuncs.com/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src http://mirrors.cloud.aliyuncs.com/debian/ bookworm-backports main contrib non-free non-free-firmware
deb http://mirrors.cloud.aliyuncs.com/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src http://mirrors.cloud.aliyuncs.com/debian-security bookworm-security main contrib non-free non-free-firmware
TEXT
```

</details>

<details>
<summary>对于 Debian 12, 境内机器, 使用清华大学镜像源</summary>

```sh
cat <<'TEXT' > /etc/apt/sources.list
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb-src http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
# 除非官方源死活连接不上, 否则还是用官方的, 因为镜像站往往有同步延迟。参考 https://www.debian.org/security/faq.en.html#mirror
deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
TEXT
```

</details>

#### 0.2.2. 更新并安装常用工具

```sh
# 在 apt 2.1.9 及以后的版本中, apt 的 HTTP Pipelining 特性
# 与 Nginx 服务器疑似存在一定的不兼容问题，可能导致出现偶发
# 的 Connection reset by peer 错误, 可选择性关闭之 (取消注释下面这行)
# echo "Acquire::http::Pipeline-Depth \"0\";" > /etc/apt/apt.conf.d/99nopipelining

apt update && apt upgrade -y && apt dist-upgrade -y && apt full-upgrade -y && apt autoremove -y

# 安装常用工具
apt install sudo curl wget build-essential apt-transport-https ca-certificates dnsutils zip unzip iftop htop zsh bat autojump fontconfig git mtr-tiny lsof -y
```

#### 0.2.3. 给机器重命名 (可选)

```sh
hostnamectl set-hostname {new name}
```

[color=red]**🥰注意事项**[/color]

- 命名后记得去 `/etc/hosts/` 加一行 `127.0.0.1 {new name}`, 否则 sudo 会埋怨.

### 0.3. 配置 SSH

推荐使用公钥验证登录, 抛弃密码和 `fail2ban` 吧~ 当然, 不方便用公钥的, 密码一定要足够强.

#### 0.3.1 创建密钥文件

[color=red]**🥰温馨提醒**[/color]

- 一般在本地操作, [color=red]**尽量不要在远程机器操作**[/color], 以免你把私钥文件忘在机器里. 系统带 OpenSSH 套件就行.

执行:

```sh
ssh-keygen -t ed25519 -C "{密钥注释}"
```

![图片|690x94](upload://sHkBdYxVVygh0MroELjkCoA3ea5.png)

提示密钥保存位置 (Enter file in which to save the key), 回车默认存在用户根目录下的 `.ssh` 文件夹下. 我以当前目录下为例, 注意不要忘了文件名.

![图片|690x84](upload://erixgAIGlcm9PU2iLPYjvodb2CL.png)
提示输入密码, 建议设置密码, 防止密钥被盗用. 输入密码时不会显示. 回车确认.

![图片|690x106](upload://rPcN8dcjqOCaYkA76HqC6mbImnE.png)
再输一次密码, 回车确认, 完成密钥生成.

![图片|690x421](upload://bo5BdHfQBDYymLzF9SSupe03awS.png)
![图片|690x52](upload://9YODGGm6cme4w89mt2XnaX8VQoq.png)

`id_ed25519_test` 就是私钥文件, `id_ed25519_test.pub` 就是公钥文件, 纯文本格式打开公钥文件 (电脑装了微软的 Publisher 会误识别文件后缀哦), 实例内容长这样 (下文也拿这个当例子):

```plaintext
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMz2IFBt+FWJc750FHf5hFt7UeXqF4gCet7td+FX5FlX Test
```

#### 0.3.2. 在 VPS 上配置公钥

```sh
# 备份一下, 养成习惯
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak

# 改成自己的公钥!!!
cat <<'EOF' > ~/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMz2IFBt+FWJc750FHf5hFt7UeXqF4gCet7td+FX5FlX Test
EOF
```

[color=red]**🥰注意事项**[/color]

- 请务必使用公钥尝试登录, 以验证配置是否成功.

  **本地执行**:

  ```sh
  ssh root@{机器 IP} -i {私钥文件路径}
  ```

  成功连接再执行下一步.

#### 0.3.3. 配置 SSH config

```sh
# 备份一下, 养成习惯
mv /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

cat <<'TEXT' > /etc/ssh/sshd_config
Include /etc/ssh/sshd_config.d/*.conf

# 配置 SSH 端口, 默认 22
# Port 22
AddressFamily any
# ListenAddress 127.0.0.1
# ListenAddress ::

# Ciphers and keying
# RekeyLimit default none

# Log facility
SyslogFacility AUTH
LogLevel INFO

# Authentication
# Must login within 2 minutes
LoginGraceTime 2m
# Allow root
PermitRootLogin yes
# Check home dir and config files' permission before connection
StrictModes yes
# Max auth attempt
MaxAuthTries 6
# Max concurrent sessions
MaxSessions 16
# Pubkey auth
PubkeyAuthentication yes
# Disallow password login
# 注意: 如果你要密码登陆, 请改为 yes
PasswordAuthentication no
# Disallow empty password
PermitEmptyPasswords no
# Disable challenge-response
ChallengeResponseAuthentication no
# Use PAM
UsePAM yes
# X11 Forwarding
X11Forwarding yes
# X11DisplayOffset 10
# X11UseLocalhost yes
# Print /etc/motd
PrintMotd no
# Print last login log
PrintLastLog yes
# TCP Keepalive
TCPKeepAlive yes
ClientAliveInterval 120
ClientAliveCountMax 10
# PID
PidFile /var/run/sshd.pid
# Allow client to pass locale environment variables
AcceptEnv LANG LC_*
# override default of no subsystems
Subsystem sftp /usr/lib/openssh/sftp-server
TEXT

# 重启 sshd

service sshd restart
```

[color=red]**🥰注意事项**[/color]

- 重启 sshd 后, 请再次使用公钥尝试登录([color=red]**千万不要关掉当前终端**[/color]), 验证配置.

  倘若登录失败, 回滚备份, 再次重启即可, 然后查询资料寻找原因.

### 0.4. 配置 ZSH (可选)

默认的 `BASH` 比较难看, 个人常用 `ZSH`, 没用过 `FISH`.

#### 0.4.1 安装 Oh My ZSH

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# 配置zsh插件
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

<details>
<summary>也可以使用清华镜像源, 插件使用代理.</summary>

```sh
cd /tmp
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh
# 配置zsh插件
git clone --depth=1 https://gh-proxy.com/https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
git clone https://gh-proxy.com/https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://gh-proxy.com/https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

</details>

#### 0.4.2. 配置 `.zshrc`

```sh
cat <<'TEXT' >  ~/.zshrc
# PROXY SETTING
export proxy=""
# export proxy="http://127.0.0.1:11100" # 自己根据实际情况配置代理
export http_proxy=$proxy
export https_proxy=$proxy
export ftp_proxy=$proxy

# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
# Initialization code that may require console input (password prompts, [y/n]
# confirmations, etc.) must go above this block; everything else may go below.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# Path to your oh-my-zsh installation.
export ZSH=$HOME/.oh-my-zsh

# Set name of the theme to load
ZSH_THEME="powerlevel10k/powerlevel10k"

# Uncomment the following line to use case-sensitive completion.
CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment one of the following lines to change the auto-update behavior
# zstyle ':omz:update' mode disabled  # disable automatic updates
zstyle ':omz:update' mode auto      # update automatically without asking
# zstyle ':omz:update' mode reminder  # just remind me to update when it's time

# Uncomment the following line to change how often to auto-update (in days).
# zstyle ':omz:update' frequency 13

# Uncomment the following line if pasting URLs and other text is messed up.
# DISABLE_MAGIC_FUNCTIONS="true"

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# You can also set it to another string to have that shown instead of the default red dots.
# e.g. COMPLETION_WAITING_DOTS="%F{yellow}waiting...%f"
# Caution: this setting can cause issues with multiline prompts in zsh < 5.7.1 (see #5765)
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(
     git
     extract
     autojump
     zsh-syntax-highlighting
     zsh-autosuggestions
     command-not-found
     colored-man-pages
)

source $ZSH/oh-my-zsh.sh

# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"
source ~/.p10k.terminal.zsh

# Other Settings
alias ls='ls --color=auto'
alias grep='grep --color=auto'
alias ll='ls -lh'
alias la='ls -A'
alias l='ls -CF'
alias cp='cp -i'
alias cat='batcat --paging=never -p'

# autosuggestions
export TERM=xterm-256color
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=8"

# Fix numeric keypad
# 0 . Enter
bindkey -s "^[Op" "0"
bindkey -s "^[On" "."
bindkey -s "^[OM" "^M"
# 1 2 3
bindkey -s "^[Oq" "1"
bindkey -s "^[Or" "2"
bindkey -s "^[Os" "3"
# 4 5 6
bindkey -s "^[Ot" "4"
bindkey -s "^[Ou" "5"
bindkey -s "^[Ov" "6"
# 7 8 9
bindkey -s "^[Ow" "7"
bindkey -s "^[Ox" "8"
bindkey -s "^[Oy" "9"
# + - * /
bindkey -s "^[Ol" "+"
bindkey -s "^[Om" "-"
bindkey -s "^[Oj" "*"
bindkey -s "^[Oo" "/"

TEXT
```

此处使用了 Powerlevel10k 主题, 提供我的主题配置, 直接 `nano ~/.p10k.terminal.zsh`, 把下面文件的内容粘贴进去即可, 或者你自己配置.

[.p10k.terminal.zip|attachment](upload://4VyC70MhdPX5XpbFA5ITsZyX1NE.zip) (19.1 KB)

### 0.5. 防火墙配置 (如果不是阿里云等大厂的机器, 没安全组的情况下必须配置)

一般使用 ufw 管理.

```sh
apt install ufw
ufw default deny incoming
ufw default allow outgoing
# 根据实际情况 allow, 不知道具体语法可以问 AI 哦
ufw allow 22

ufw enable
```

按 y 确认应用即可.

<details>
<summary>可选阻止 PING, 不过略复杂, 一不小心 SSH 断了. 小白还是别动了...</summary>

```sh
cp /etc/ufw/before.rules /etc/ufw/before.rules.bak
cat <<'EOF' > /etc/ufw/before.rules
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# Don't delete these required lines, otherwise there will be errors
*filter
:ufw-before-input - [0:0]
:ufw-before-output - [0:0]
:ufw-before-forward - [0:0]
:ufw-not-local - [0:0]
# End required lines


# allow all on loopback
-A ufw-before-input -i lo -j ACCEPT
-A ufw-before-output -o lo -j ACCEPT

# disallow ping
-A ufw-before-input -p icmp --icmp-type echo-request -s 10.10.0.0/16 -j ACCEPT
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP

# quickly process packets for which we already have a connection
-A ufw-before-input -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw-before-output -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw-before-forward -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# drop INVALID packets (logs these in loglevel medium and higher)
-A ufw-before-input -m conntrack --ctstate INVALID -j ufw-logging-deny
-A ufw-before-input -m conntrack --ctstate INVALID -j DROP

# ok icmp codes for INPUT
-A ufw-before-input -p icmp --icmp-type destination-unreachable -j ACCEPT
-A ufw-before-input -p icmp --icmp-type time-exceeded -j ACCEPT
-A ufw-before-input -p icmp --icmp-type parameter-problem -j ACCEPT
# -A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT

# ok icmp code for FORWARD
-A ufw-before-forward -p icmp --icmp-type destination-unreachable -j ACCEPT
-A ufw-before-forward -p icmp --icmp-type time-exceeded -j ACCEPT
-A ufw-before-forward -p icmp --icmp-type parameter-problem -j ACCEPT
-A ufw-before-forward -p icmp --icmp-type echo-request -j ACCEPT

# allow dhcp client to work
-A ufw-before-input -p udp --sport 67 --dport 68 -j ACCEPT

#
# ufw-not-local
#
-A ufw-before-input -j ufw-not-local

# if LOCAL, RETURN
-A ufw-not-local -m addrtype --dst-type LOCAL -j RETURN

# if MULTICAST, RETURN
-A ufw-not-local -m addrtype --dst-type MULTICAST -j RETURN

# if BROADCAST, RETURN
-A ufw-not-local -m addrtype --dst-type BROADCAST -j RETURN

# all other non-local packets are dropped
-A ufw-not-local -m limit --limit 3/min --limit-burst 10 -j ufw-logging-deny
-A ufw-not-local -j DROP

# allow MULTICAST mDNS for service discovery (be sure the MULTICAST line above
# is uncommented)
-A ufw-before-input -p udp -d 224.0.0.251 --dport 5353 -j ACCEPT

# allow MULTICAST UPnP for service discovery (be sure the MULTICAST line above
# is uncommented)
-A ufw-before-input -p udp -d 239.255.255.250 --dport 1900 -j ACCEPT

# don't delete the 'COMMIT' line or these rules won't be processed
COMMIT

EOF

cp /etc/ufw/before6.rules /etc/ufw/before6.rules.bak
cat <<'EOF' > /etc/ufw/before6.rules
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw6-before-input
#   ufw6-before-output
#   ufw6-before-forward
#

# Don't delete these required lines, otherwise there will be errors
*filter
:ufw6-before-input - [0:0]
:ufw6-before-output - [0:0]
:ufw6-before-forward - [0:0]
# End required lines


# allow all on loopback
-A ufw6-before-input -i lo -j ACCEPT
-A ufw6-before-output -o lo -j ACCEPT

# drop packets with RH0 headers
-A ufw6-before-input -m rt --rt-type 0 -j DROP
-A ufw6-before-forward -m rt --rt-type 0 -j DROP
-A ufw6-before-output -m rt --rt-type 0 -j DROP

# quickly process packets for which we already have a connection
-A ufw6-before-input -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw6-before-output -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw6-before-forward -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# multicast ping replies are part of the ok icmp codes for INPUT (rfc4890,
# 4.4.1 and 4.4.2), but don't have an associated connection and are otherwise
# be marked INVALID, so allow here instead.
-A ufw6-before-input -p icmpv6 --icmpv6-type echo-reply -j ACCEPT

# drop INVALID packets (logs these in loglevel medium and higher)
-A ufw6-before-input -m conntrack --ctstate INVALID -j ufw6-logging-deny
-A ufw6-before-input -m conntrack --ctstate INVALID -j DROP

# ok icmp codes for INPUT (rfc4890, 4.4.1 and 4.4.2)
-A ufw6-before-input -p icmpv6 --icmpv6-type destination-unreachable -j ACCEPT
-A ufw6-before-input -p icmpv6 --icmpv6-type packet-too-big -j ACCEPT
# codes 0 and 1
-A ufw6-before-input -p icmpv6 --icmpv6-type time-exceeded -j ACCEPT
# codes 0-2 (echo-reply needs to be before INVALID, see above)
-A ufw6-before-input -p icmpv6 --icmpv6-type parameter-problem -j ACCEPT
-A ufw6-before-input -p icmpv6 --icmpv6-type echo-request -j ACCEPT
-A ufw6-before-input -p icmpv6 --icmpv6-type router-solicitation -m hl --hl-eq 255 -j ACCEPT
-A ufw6-before-input -p icmpv6 --icmpv6-type router-advertisement -m hl --hl-eq 255 -j ACCEPT
-A ufw6-before-input -p icmpv6 --icmpv6-type neighbor-solicitation -m hl --hl-eq 255 -j ACCEPT
-A ufw6-before-input -p icmpv6 --icmpv6-type neighbor-advertisement -m hl --hl-eq 255 -j ACCEPT
# IND solicitation
-A ufw6-before-input -p icmpv6 --icmpv6-type 141 -m hl --hl-eq 255 -j ACCEPT
# IND advertisement
-A ufw6-before-input -p icmpv6 --icmpv6-type 142 -m hl --hl-eq 255 -j ACCEPT
# MLD query
-A ufw6-before-input -p icmpv6 --icmpv6-type 130 -s fe80::/10 -j ACCEPT
# MLD report
-A ufw6-before-input -p icmpv6 --icmpv6-type 131 -s fe80::/10 -j ACCEPT
# MLD done
-A ufw6-before-input -p icmpv6 --icmpv6-type 132 -s fe80::/10 -j ACCEPT
# MLD report v2
-A ufw6-before-input -p icmpv6 --icmpv6-type 143 -s fe80::/10 -j ACCEPT
# SEND certificate path solicitation
-A ufw6-before-input -p icmpv6 --icmpv6-type 148 -m hl --hl-eq 255 -j ACCEPT
# SEND certificate path advertisement
-A ufw6-before-input -p icmpv6 --icmpv6-type 149 -m hl --hl-eq 255 -j ACCEPT
# MR advertisement
-A ufw6-before-input -p icmpv6 --icmpv6-type 151 -s fe80::/10 -m hl --hl-eq 1 -j ACCEPT
# MR solicitation
-A ufw6-before-input -p icmpv6 --icmpv6-type 152 -s fe80::/10 -m hl --hl-eq 1 -j ACCEPT
# MR termination
-A ufw6-before-input -p icmpv6 --icmpv6-type 153 -s fe80::/10 -m hl --hl-eq 1 -j ACCEPT

# ok icmp codes for OUTPUT (rfc4890, 4.4.1 and 4.4.2)
-A ufw6-before-output -p icmpv6 --icmpv6-type destination-unreachable -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type packet-too-big -j ACCEPT
# codes 0 and 1
-A ufw6-before-output -p icmpv6 --icmpv6-type time-exceeded -j ACCEPT
# codes 0-2
-A ufw6-before-output -p icmpv6 --icmpv6-type parameter-problem -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type echo-request -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type echo-reply -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type router-solicitation -m hl --hl-eq 255 -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type neighbor-advertisement -m hl --hl-eq 255 -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type neighbor-solicitation -m hl --hl-eq 255 -j ACCEPT
-A ufw6-before-output -p icmpv6 --icmpv6-type router-advertisement -m hl --hl-eq 255 -j ACCEPT
# IND solicitation
-A ufw6-before-output -p icmpv6 --icmpv6-type 141 -m hl --hl-eq 255 -j ACCEPT
# IND advertisement
-A ufw6-before-output -p icmpv6 --icmpv6-type 142 -m hl --hl-eq 255 -j ACCEPT
# MLD query
-A ufw6-before-output -p icmpv6 --icmpv6-type 130 -s fe80::/10 -j ACCEPT
# MLD report
-A ufw6-before-output -p icmpv6 --icmpv6-type 131 -s fe80::/10 -j ACCEPT
# MLD done
-A ufw6-before-output -p icmpv6 --icmpv6-type 132 -s fe80::/10 -j ACCEPT
# MLD report v2
-A ufw6-before-output -p icmpv6 --icmpv6-type 143 -s fe80::/10 -j ACCEPT
# SEND certificate path solicitation
-A ufw6-before-output -p icmpv6 --icmpv6-type 148 -m hl --hl-eq 255 -j ACCEPT
# SEND certificate path advertisement
-A ufw6-before-output -p icmpv6 --icmpv6-type 149 -m hl --hl-eq 255 -j ACCEPT
# MR advertisement
-A ufw6-before-output -p icmpv6 --icmpv6-type 151 -s fe80::/10 -m hl --hl-eq 1 -j ACCEPT
# MR solicitation
-A ufw6-before-output -p icmpv6 --icmpv6-type 152 -s fe80::/10 -m hl --hl-eq 1 -j ACCEPT
# MR termination
-A ufw6-before-output -p icmpv6 --icmpv6-type 153 -s fe80::/10 -m hl --hl-eq 1 -j ACCEPT

# ok icmp codes for FORWARD (rfc4890, 4.3.1)
-A ufw6-before-forward -p icmpv6 --icmpv6-type destination-unreachable -j ACCEPT
-A ufw6-before-forward -p icmpv6 --icmpv6-type packet-too-big -j ACCEPT
# codes 0 and 1
-A ufw6-before-forward -p icmpv6 --icmpv6-type time-exceeded -j ACCEPT
# codes 0-2
-A ufw6-before-forward -p icmpv6 --icmpv6-type parameter-problem -j ACCEPT
-A ufw6-before-forward -p icmpv6 --icmpv6-type echo-request -j ACCEPT
-A ufw6-before-forward -p icmpv6 --icmpv6-type echo-reply -j ACCEPT
# ok icmp codes for FORWARD (rfc4890, 4.3.2)
# Home Agent Address Discovery Reques
-A ufw6-before-input -p icmpv6 --icmpv6-type 144 -j ACCEPT
# Home Agent Address Discovery Reply
-A ufw6-before-input -p icmpv6 --icmpv6-type 145 -j ACCEPT
# Mobile Prefix Solicitation
-A ufw6-before-input -p icmpv6 --icmpv6-type 146 -j ACCEPT
# Mobile Prefix Advertisement
-A ufw6-before-input -p icmpv6 --icmpv6-type 147 -j ACCEPT

# allow dhcp client to work
-A ufw6-before-input -p udp -s fe80::/10 --sport 547 -d fe80::/10 --dport 546 -j ACCEPT

# allow MULTICAST mDNS for service discovery
-A ufw6-before-input -p udp -d ff02::fb --dport 5353 -j ACCEPT

# allow MULTICAST UPnP for service discovery
-A ufw6-before-input -p udp -d ff02::f --dport 1900 -j ACCEPT

# don't delete the 'COMMIT' line or these rules won't be processed
COMMIT

EOF
```

</details>

---

到此, 基本配置结束.

## 1. 进阶配置

### 1.1. 使用 [Xanmod](https://xanmod.org/) 内核 (推荐)

Xanmod 内核做了蛮多优化, 而且比那些脚本装的奇奇怪怪的第三方内核来源可靠多了.

值得注意的是, Cloudflare 拉黑了阿里云的 IP, 所以阿里云的机器可能执行脚本失败, 得自己打开脚本网页把内容复制一下, 新建文件粘贴进去然后执行.

#### 1.1.1. 检查 CPU 兼容性

```sh
wget -O check_x86-64_psabi.sh https://dl.xanmod.org/check_x86-64_psabi.sh && chmod +x check_x86-64_psabi.sh && ./check_x86-64_psabi.sh
```

![图片|690x188](upload://1PyQp1qytxKIAXEe82K9MhQq4FZ.png)

如示例, 支持 v2 版本.

#### 1.1.2. 配置 GPG key

```sh
apt install gpg
wget -qO - https://gitlab.com/afrd.gpg | sudo gpg --dearmor -vo /etc/apt/keyrings/xanmod-archive-keyring.gpg
```

<details>
<summary>为什么不用官方说明里面的 GPG key 地址?</summary>

![图片|690x295](upload://oiCI5zAumJXl0O1jgnM2pnbQIyu.jpeg)

</details>

#### 1.1.3. 配置源

```sh
echo 'deb [signed-by=/etc/apt/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-release.list
```

<details>
<summary>或者使用清华镜像源</summary>

```sh
echo 'deb [signed-by=/etc/apt/keyrings/xanmod-archive-keyring.gpg] http://mirrors.tuna.tsinghua.edu.cn/xanmod releases main' | sudo tee /etc/apt/sources.list.d/xanmod-release.list
```

</details>

[color=red]**🥰注意事项**[/color]

- [color=red]**不要使用 HTTPS**[/color], 否则有非常奇怪的 BUG, 原因不知道.

#### 1.1.4. 安装

```sh
apt update && apt install linux-xanmod-x64v2
```

这里前面检测 CPU 的时候会告诉你用 v 几的版本. 一般都可以用 v3, 除非机器太老了.

### 1.2. `sysctl.conf` 优化 (**推荐**)

```sh
cat <<'TEXT' > /etc/sysctl.conf
# ------ 网络调优: 基本 ------
# TTL 配置, Linux 默认 64
# net.ipv4.ip_default_ttl=64

# 参阅 RFC 1323. 应当启用.
net.ipv4.tcp_timestamps=1
# ------ END 网络调优: 基本 ------

# ------ 网络调优: 内核 Backlog 队列和缓存相关 ------
# Ref: https://www.starduster.me/2020/03/02/linux-network-tuning-kernel-parameter/
# Ref: https://blog.cloudflare.com/optimizing-tcp-for-high-throughput-and-low-latency/
# Ref: https://zhuanlan.zhihu.com/p/149372947
# 下面两个缓冲区配置目前没有定论说怎么设置是正确的
# 此处为 Cloudflare 在生产中使用的值 (参见前面的博客)
# 有条件建议依据实测结果调整相关数值
# 由左往右依次为 `最小值` `默认值` `最大值`
# 最大值按照机器带宽 (Bps) * RTT (s) 计算
# 如 1Gbps (125 MBps) 的带宽, 40ms 延迟, 就是 125_000_000 * 0.040 = 5000000
net.ipv4.tcp_rmem=8192 262144 5000000
net.ipv4.tcp_wmem=4096 16384 5000000
# 下面这四个不用配置, 和上面两个效果重复了, 会被覆盖
# net.core.wmem_default=1310720
# net.core.rmem_default=1310720
# net.core.rmem_max=536870912
# net.core.wmem_max=536870912
net.ipv4.tcp_adv_win_scale=-2
net.ipv4.tcp_collapse_max_bytes=6291456
net.ipv4.tcp_notsent_lowat=131072
net.core.netdev_max_backlog=10240
net.ipv4.tcp_max_syn_backlog=10240
net.core.somaxconn=3276800
net.ipv4.tcp_abort_on_overflow=1
# 所有网卡每次软中断最多处理的总帧数量
net.core.netdev_budget = 600
# 流控和拥塞控制相关调优
# Egress traffic control 相关. 可选 fq, cake
# 实测二者区别不大, 保持默认 fq 即可
net.core.default_qdisc=fq
# Xanmod 内核 6.X 版本目前默认使用 bbr3, 无需设置
# 实测比 bbr, bbr2 均有提升
# 不过网络条件不同会影响. 有需求请实测.
# net.ipv4.tcp_congestion_control=bbr3
# 显式拥塞通知
# 已被发现在高度拥塞的网络上是有害的.
# net.ipv4.tcp_ecn=1
# TCP 自动窗口
# 要支持超过 64KB 的 TCP 窗口必须启用
net.ipv4.tcp_window_scaling=1
# 开启后, TCP 拥塞窗口会在一个 RTO 时间
# 空闲之后重置为初始拥塞窗口 (CWND) 大小.
# 大部分情况下, 尤其是大流量长连接, 设置为 0.
# 对于网络情况时刻在相对剧烈变化的场景, 设置为 1.
net.ipv4.tcp_slow_start_after_idle=1
# nf_conntrack 调优
# Add Ref: https://gist.github.com/lixingcong/0e13b4123d29a465e364e230b2e45f60
net.nf_conntrack_max=1000000
net.netfilter.nf_conntrack_max=1000000
net.netfilter.nf_conntrack_tcp_timeout_fin_wait=30
net.netfilter.nf_conntrack_tcp_timeout_time_wait=30
net.netfilter.nf_conntrack_tcp_timeout_close_wait=15
net.netfilter.nf_conntrack_tcp_timeout_established=300
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established=7200
# TIME-WAIT 状态调优
# Ref: http://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux.html
# Ref: https://www.cnblogs.com/lulu/p/4149312.html
# 4.12 内核中此参数已经永久废弃, 不用纠结是否需要开启
# net.ipv4.tcp_tw_recycle=0
## 只对客户端生效, 服务器连接上游时也认为是客户端
net.ipv4.tcp_tw_reuse=1
# 系统同时保持TIME_WAIT套接字的最大数量
# 如果超过这个数字 TIME_WAIT 套接字将立刻被清除
net.ipv4.tcp_max_tw_buckets=55000
# ------ END 网络调优: 内核 Backlog 队列和缓存相关 ------

# ------ 网络调优: 其他 ------
# Ref: https://zhuanlan.zhihu.com/p/149372947
# Ref: https://www.starduster.me/2020/03/02/linux-network-tuning-kernel-parameter/\#netipv4tcp_max_syn_backlog_netipv4tcp_syncookies
# 启用选择应答
# 对于广域网通信应当启用
net.ipv4.tcp_sack=1
# 启用转发应答
# 对于广域网通信应当启用
net.ipv4.tcp_fack=1
# TCP SYN 连接超时重传次数
net.ipv4.tcp_syn_retries=3
net.ipv4.tcp_synack_retries=3
# TCP SYN 连接超时时间, 设置为 5 约为 30s
net.ipv4.tcp_retries2=5
# 开启 SYN 洪水攻击保护
# 注意: tcp_syncookies 启用时, 此时实际上没有逻辑上的队列长度,
# Backlog 设置将被忽略. syncookie 是一个出于对现实的妥协,
# 严重违反 TCP 协议的设计, 会造成 TCP option 不可用, 且实现上
# 通过计算 hash 避免维护半开连接也是一种 tradeoff 而非万金油,
# 勿听信所谓“安全优化教程”而无脑开启
net.ipv4.tcp_syncookies=0

# Ref: https://linuxgeeks.github.io/2017/03/20/212135-Linux%E5%86%85%E6%A0%B8%E5%8F%82%E6%95%B0rp_filter/
# 开启反向路径过滤
# Aliyun 负载均衡实例后端的 ECS 需要设置为 0
net.ipv4.conf.default.rp_filter=2
net.ipv4.conf.all.rp_filter=2

# 减少处于 FIN-WAIT-2 连接状态的时间使系统可以处理更多的连接
# Ref: https://www.cnblogs.com/kaishirenshi/p/11544874.html
net.ipv4.tcp_fin_timeout=10

# Ref: https://xwl-note.readthedocs.io/en/latest/linux/tuning.html
# 默认情况下一个 TCP 连接关闭后, 把这个连接曾经有的参数保存到dst_entry中
# 只要 dst_entry 没有失效, 下次新建立相同连接的时候就可以使用保存的参数来初始化这个连接.
# 通常情况下是关闭的, 高并发配置为 1.
net.ipv4.tcp_no_metrics_save=1
# unix socket 最大队列
net.unix.max_dgram_qlen=1024
# 路由缓存刷新频率
net.ipv4.route.gc_timeout=100

# Ref: https://gist.github.com/lixingcong/0e13b4123d29a465e364e230b2e45f60
# 启用 MTU 探测，在链路上存在 ICMP 黑洞时候有用（大多数情况是这样）
net.ipv4.tcp_mtu_probing = 1

# No Ref
# 开启并记录欺骗, 源路由和重定向包
net.ipv4.conf.all.log_martians=1
net.ipv4.conf.default.log_martians=1
# 处理无源路由的包
net.ipv4.conf.all.accept_source_route=0
net.ipv4.conf.default.accept_source_route=0
# TCP KeepAlive 调优
# 最大闲置时间
net.ipv4.tcp_keepalive_time=600
# 最大失败次数, 超过此值后将通知应用层连接失效
net.ipv4.tcp_keepalive_probes=3
# 发送探测包的时间间隔
net.ipv4.tcp_keepalive_intvl=15
# 放弃回应一个 TCP 连接请求前, 需要进行多少次重试
net.ipv4.tcp_retries1 = 5
# 在丢弃激活(已建立通讯状况)的 TCP 连接之前, 需要进行多少次重试
net.ipv4.tcp_retries2 = 5
# 孤立 Socket
net.ipv4.tcp_orphan_retries = 3
# 系统所能处理不属于任何进程的TCP sockets最大数量
net.ipv4.tcp_max_orphans=3276800
# arp_table的缓存限制优化
net.ipv4.neigh.default.gc_thresh1=128
net.ipv4.neigh.default.gc_thresh2=512
net.ipv4.neigh.default.gc_thresh3=4096
net.ipv4.neigh.default.gc_stale_time=120
net.ipv4.conf.default.arp_announce=2
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_announce=2
# ------ END 网络调优: 其他 ------

# ------ 内核调优 ------

# Ref: Aliyun, etc
# 内核 Panic 后 1 秒自动重启
kernel.panic=1
# 允许更多的PIDs, 减少滚动翻转问题
kernel.pid_max=32768
# 内核所允许的最大共享内存段的大小（bytes）
kernel.shmmax=4294967296
# 在任何给定时刻, 系统上可以使用的共享内存的总量（pages）
kernel.shmall=1073741824
# 设定程序core时生成的文件名格式
kernel.core_pattern=core_%e
# 当发生oom时, 自动转换为panic
vm.panic_on_oom=1
# 表示强制Linux VM最低保留多少空闲内存（Kbytes）
# vm.min_free_kbytes=1048576
# 该值高于100, 则将导致内核倾向于回收directory和inode cache
vm.vfs_cache_pressure=250
# 表示系统进行交换行为的程度, 数值（0-100）越高, 越可能发生磁盘交换
vm.swappiness=10
# 仅用10%做为系统cache
vm.dirty_ratio=10
vm.overcommit_memory=1
# 增加系统文件描述符限制
# Fix error: too many open files
fs.file-max=6553560
fs.inotify.max_user_instances=8192
fs.inotify.max_user_instances=8192
# 内核响应魔术键
kernel.sysrq=1
# 弃用
# net.ipv4.tcp_low_latency=1

# Ref: https://gist.github.com/lixingcong/0e13b4123d29a465e364e230b2e45f60
# 当某个节点可用内存不足时, 系统会倾向于从其他节点分配内存. 对 Mongo/Redis 类 cache 服务器友好
vm.zone_reclaim_mode=0

# Ref: Unknwon
# 开启F-RTO(针对TCP重传超时的增强的恢复算法).
# 在无线环境下特别有益处, 因为在这种环境下分组丢失典型地是因为随机无线电干扰而不是中间路由器阻塞
net.ipv4.tcp_frto = 2
# TCP FastOpen
net.ipv4.tcp_fastopen = 3
# TCP 流中重排序的数据报最大数量
net.ipv4.tcp_reordering = 300
# 开启后, 在重传时会试图发送满大小的包. 这是对一些有 BUG 的打印机的绕过方式
net.ipv4.tcp_retrans_collapse = 0
# 自动阻塞判断
net.ipv4.tcp_autocorking = 1
# TCP内存自动调整
net.ipv4.tcp_moderate_rcvbuf = 1
# 单个TSO段可消耗拥塞窗口的比例, 默认值为 3
net.ipv4.tcp_tso_win_divisor = 3
# 对于在 RFC1337 中描述的 TIME-WAIT Assassination Hazards in TCP 问题的修复
net.ipv4.tcp_rfc1337 = 1
# 包转发. 出于安全考虑, Linux 系统默认禁止数据包转发
net.ipv4.ip_forward = 0
# 取消对广播 ICMP 包的回应
net.ipv4.icmp_echo_ignore_broadcasts = 1
# 开启恶意 ICMP 错误消息保护
net.ipv4.icmp_ignore_bogus_error_responses = 1

TEXT
```

推荐重启彻底应用, 不过不用着急, `systemctl -p` 暂时应用也行.

> 这些调优是我广泛查阅资料总结而来, 出处及理由都注释了, 我个人认为比那些优化脚本靠谱多了. 欢迎各位佬友评论, 补充, 修正~

### 1.3. ulimit 优化 (推荐)

```sh
cat <<'TEXT' > /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
root soft nofile 65535
root hard nofile 65535
root soft nproc 65535
root hard nproc 65535
TEXT

cat <<'TEXT' > /etc/security/limits.d/90-nproc.conf
* soft nproc 65535
root soft nproc 65535
TEXT
```

需要重启才能应用, 不过不要着急, 完成所有部署工作再重启也不迟.

### 1.4. `systemd-journald` 等日志系统配置

前车之鉴: <https://linux.do/t/topic/274270>

```sh
cat <<'EOF' > /etc/systemd/journald.conf
[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitIntervalSec=30s
#RateLimitBurst=10000
# Limit total persist storage
SystemMaxUse=512M
# Limit single file persist storage
SystemMaxFileSize=128M
#SystemMaxFiles=100
# Limit RAM usage
RuntimeMaxUse=64M
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#RuntimeMaxFiles=100
#MaxRetentionSec=
#MaxFileSec=1month
# Stop forward to Syslog
ForwardToSyslog=no
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
#LineMax=48K
#ReadKMsg=yes
#Audit=no
EOF

systemctl restart systemd-journald

# Install rsyslog if not exist
apt install rsyslog
systemctl enable rsyslog --now
```

### 1.5. 时间同步配置 (使用 `systemd-timesyncd`)

[详细参考 (in English)](https://acfun.win/posts/25-02-11-time-synchronization-based-on-systemd-timesyncd/)

```sh
apt update

# 忘了时区代码可以列一下
# timedatectl list-timezones
# 设置时区, 此处以中国香港为例, 中国大陆就是 `Asia/Shanghai`
timedatectl set-timezone Asia/Hong_Kong

apt purge ntp
apt install systemd-timesyncd

cat <<'EOF' > /etc/systemd/timesyncd.conf
[Time]
# 主 NTP 服务器
# 中国大陆, 使用国家授时中心官方 NTP, 海外的用 Cloudflare 的很不错, 改成 `time.cloudflare.com`
NTP=ntp.ntsc.ac.cn

# 备份 NTP 服务器
FallbackNTP=pool.ntp.org time1.google.com time1.apple.com time.cloudflare.com time.windows.com time.nist.gov

# 同步配置
RootDistanceMaxSec=5
PollIntervalMinSec=5
PollIntervalMaxSec=300
ConnectionRetrySec=30
SaveIntervalSec=60
EOF

systemctl enable --now systemd-timesyncd
```

### 1.6. 使用 netplan 管理网络 (危险)

一般用不到, 一不小心就是断网, 一般都是家里的 Linux 主机使用, 此处仅简单介绍.

```sh
apt update && apt install netplan.io

# Backup original one
mv /etc/network/interfaces /etc/network/interfaces.bak

cat <<'TEXT' > /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
TEXT
```

参考资料: <https://www.debian.org/doc/manuals/debian-reference/ch05.zh-cn.html>

IP 固定(大部分都是) 的示例:

```sh
cat <<'TEXT' > /etc/netplan/50-static.yaml
network:
    version: 2
    ethernets:
        {eth_name}:
            addresses:
            - {v4}
            - {v6}
            routes:
            - to: default
              via: {v4_gateway}
              on-link: true
            - to: default
              via: {v6_gateway}
              on-link: true
            match:
                macaddress: {macaddress}
            nameservers:
                addresses:
                - 1.1.1.1
                - 1.0.0.1
                - 2606:4700:4700::1001
                - 2606:4700:4700::1111
            set-name: {eth_name}
TEXT
```

DHCP 的一则例子, 使用  "en*" 匹配网卡, 如 `enp2s0` 一类的名字.

```sh
cat <<'TEXT' > /etc/netplan/99-dhcp.yaml
network:
    version: 2
    ethernets:
        all-en:
            match:
                name: "en*"
            dhcp4: true
            dhcp6: true
            ipv6-privacy: true
            accept-ra: true
TEXT
```

```sh
# try configurating network, ENTER to comfirm
netplan try

# Check if netplan managed
networkctl
```

---

## 2. 一些叮嘱

- 尽量不要日用 `root`, 养成使用普通账户然后必要时 `sudo` 的习惯 (TODO)
- **不要使用 `PHP` 等**, 以及相关项目, 如 `Wordpress`, 实在太多安全问题了, 非要使用请使用 Docker 隔离. 个人对于非自己编写或审查的东西一律 Docker 伺候, 最大限度减少攻击面.
- **谨慎使用第三方脚本**, 脚本里埋雷不是没见过, 执行前务必详细检查, 让 AI 帮忙也是可以的.
- 尽量不要使用服务器管理面板, 尤其是宝塔等闭源产品, 扩大攻击面, 自身权限过高.
- 尽量不使用密码登录 SSH, 请使用密钥登录, 且私钥务必设置密码以免被盗用.
- DNS 及 SSL 篇

  TL,DR: 不要设置 DNS 直接指向源站, 使用泛域名证书

  帮助好几个 UP 排查源站泄露的原因, DNS 泄露是个相当常见的原因. 所以, **不要设置 DNS 直接指向源站**! 就算后面改为 CNAME 到 CDN 的域名, DNS 记录是可以查历史的!

  其次, 子域名爆破是查源站常用方法了, 有个非常好用的查子域名的方法是 crt.sh, 原理是查 SSL 证书颁发记录, 所以, 推荐**使用泛域名证书**.

  还有 RDNS, 不过一般没人会将自己服务器 IP 的 RDNS 配置为自己的域名, 许多商家也没提供这个功能, 此处按下不表.

---

## 结语

到这里, 新机器就开荒完毕了. 有关各类常用应用, 如 Docker 等的部署也是常见的, 但本文篇幅已经很长了, 让我们下期再见!

若本指南帮到了你, 还请点个赞~

---

*本文版权遵照 CC BY-NC-SA 协议开放, 转载请标注出处.*

*囿于自身水平, 本文所述可能并非最佳实践, 但均已经过我个人亲测. 也欢迎各位佬友补充修正, 共建这份指南!*

*2025年3月3日晚, 于北京.*

---
