---
title: "Zinit 配置留存备份"
date: 2023-03-15T16:41:49+08:00
draft: false
description: "Zinit 是一个强大的 Zsh 插件管理器，用于快速加载和管理 Zsh 插件。本文记录了 Zinit 的配置和使用方法。"
showComments: true
featureimage: "https://picsum.photos/seed/8701f424/1600/900.webp"
tags: ["Linux", "Shell", "教程配置"]
---
## Zinit 简介

Zinit 是一个高效的 Zsh 插件管理器，旨在加速 Zsh 的启动速度并简化插件的管理。它支持以下功能：

- **快速加载插件**：通过延迟加载和异步加载技术，减少 Zsh 启动时间。
- **插件管理**：轻松安装、更新和删除插件。
- **语法高亮**：支持语法高亮和自动补全插件。
- **主题支持**：兼容 Oh-My-Zsh 主题和插件。

## 安装及配置教程

### 安装 Zinit

在终端中运行以下命令安装 Zinit：`bash -c "$(curl --fail --show-error --silent --location https://raw.githubusercontent.com/zdharma-continuum/zinit/HEAD/scripts/install.sh)"`，安装完成后，Zinit 会自动将初始化代码添加到 `~/.zshrc` 文件中。

### 常用命令

```bash
zinit light <插件名称>  #安装插件
zinit update [--all]  #更新插件
zinit self-update  #zinit本体更新
zinit delete <插件名称> #删除插件
zinit list
```

---

## 配置文件示例

以下是一个完整的 Zinit 配置文件示例，包含常用插件和配置：

```zsh
# 初始化 Zinit
source ~/.zinit/bin/zinit.zsh

# 加载 Oh-My-Zsh 插件
zinit snippet OMZ::lib/git.zsh
zinit snippet OMZ::plugins/git/git.plugin.zsh

# 加载语法高亮插件
zinit light zdharma-continuum/fast-syntax-highlighting

# 加载自动补全插件
zinit light zsh-users/zsh-autosuggestions

# 加载主题（例如 Powerlevel10k）
zinit ice depth=1; zinit light romkatv/powerlevel10k

# 延迟加载插件（例如 Docker 补全）
zinit ice wait'1' lucid; zinit snippet OMZ::plugins/docker/_docker

# 加载自定义脚本
zinit light username/my-custom-plugin

# 配置 Powerlevel10k 主题
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```

## 配置文件说明

1. **初始化 Zinit**：
   - `source ~/.zinit/bin/zinit.zsh`：加载 Zinit 的核心功能。

2. **加载 Oh-My-Zsh 插件**：
   - `zinit snippet OMZ::lib/git.zsh`：加载 Oh-My-Zsh 的 Git 库。
   - `zinit snippet OMZ::plugins/git/git.plugin.zsh`：加载 Git 插件。

3. **语法高亮和自动补全**：
   - `zinit light zdharma-continuum/fast-syntax-highlighting`：加载语法高亮插件。
   - `zinit light zsh-users/zsh-autosuggestions`：加载自动补全插件。

4. **加载主题**：
   - `zinit ice depth=1; zinit light romkatv/powerlevel10k`：加载 Powerlevel10k 主题。

5. **延迟加载插件**：
   - `zinit ice wait'1' lucid; zinit snippet OMZ::plugins/docker/_docker`：延迟加载 Docker 补全插件。

6. **自定义脚本**：
   - `zinit light username/my-custom-plugin`：加载自定义插件或脚本。

7. **Powerlevel10k 配置**：
   - 如果使用 Powerlevel10k 主题，需要加载其配置文件 `~/.p10k.zsh`。

## 自己的配置文件

```ini
# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
# Initialization code that may require console input (password prompts, [y/n]
# confirmations, etc.) must go above this block; everything else may go below.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

### Added by Zinit's installer
if [[ ! -f $HOME/.local/share/zinit/zinit.git/zinit.zsh ]]; then
    print -P "%F{33} %F{220}Installing %F{33}ZDHARMA-CONTINUUM%F{220} Initiative Plugin Manager (%F{33}zdharma-continuum/zinit%F{220})…%f"
    command mkdir -p "$HOME/.local/share/zinit" && command chmod g-rwX "$HOME/.local/share/zinit"
    command git clone --depth 1 https://github.com/zdharma-continuum/zinit "$HOME/.local/share/zinit/zinit.git" && \
        print -P "%F{33} %F{34}Installation successful.%f%b" || \
        print -P "%F{160} The clone has failed.%f%b"
fi

source "$HOME/.local/share/zinit/zinit.git/zinit.zsh"
autoload -Uz _zinit
(( ${+_comps} )) && _comps[zinit]=_zinit

# Load a few important annexes, without Turbo
# (this is currently required for annexes)
zinit light-mode for \
    zdharma-continuum/zinit-annex-as-monitor \
    zdharma-continuum/zinit-annex-bin-gem-node \
    zdharma-continuum/zinit-annex-patch-dl \
    zdharma-continuum/zinit-annex-rust

### End of Zinit's installer chunk


# 加载 powerlevel10k 主题
zinit ice  lucid depth=1
zinit light romkatv/powerlevel10k
POWERLEVEL9K_INSTANT_PROMPT=quiet
# 补全
zinit wait lucid atload="zicompinit; zicdreplay" blockf for zsh-users/zsh-completions
zinit light zsh-users/zsh-completions
# 模糊查找
zinit light Aloxaf/fzf-tab
# disable sort when completing `git checkout`
zstyle ':completion:*:git-checkout:*' sort false
# set descriptions format to enable group support
zstyle ':completion:*:descriptions' format '[%d]'
# set list-colors to enable filename colorizing
zstyle ':completion:*' list-colors ${(s.:.)LS_COLORS}
# preview directory's content with eza when completing cd
zstyle ':fzf-tab:complete:cd:*' fzf-preview 'eza -1 --color=always $realpath'
# switch group using `,` and `.`
zstyle ':fzf-tab:*' switch-group ',' '.'
# 自动建议
zinit ice lucid wait="0" atload='_zsh_autosuggest_start'
zinit light zsh-users/zsh-autosuggestions

# VI-MODE 插件
# zinit ice depth=1
# zinit light jeffreytse/zsh-vi-mode
# OTHER plugins
zinit light djui/alias-tips


# 语法高亮/显示高亮
zinit ice lucid wait='0' atinit='zpcompinit'
zinit light z-shell/F-Sy-H

# 加载 OMZ 框架及部分插件
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/lib/git.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/plugins/git/git.plugin.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/lib/history.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/lib/key-bindings.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/lib/completion.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/lib/clipboard.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/plugins/aliases/aliases.plugin.zsh
zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/plugins/sudo/sudo.plugin.zsh
# zinit snippet https://gitee.com/mirrors/oh-my-zsh/raw/master/plugins/autojump/autojump.plugin.zsh
# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh

########################################################
##### ALIASES
########################################################
alias ls='eza'
alias ll='eza -lbF --git'
alias la='eza -lbhHigmuSa --time-style=long-iso --git --color-scale'
alias lx='eza -lbhHigmuSa@ --time-style=long-iso --git --color-scale'
alias llt='eza -l --git --tree'
alias lt='eza --tree --level=2'
alias llm='eza -lbGF --git --sort=modified'
alias lld='eza -lbhHFGmuSa --group-directories-first' 
alias du='dust'
alias cat='batcat'
alias unzip='unar'
alias rm="trash-put"
#######################################################
##### 自定义函数区
########################################################
setopt Autocd  #不需要输入cd
setopt HIST_IGNORE_ALL_DUPS #  移除重复的命令历史
```

## 参考链接

- [Zinit 官方文档](https://zdharma-continuum.github.io/zinit/)
- [Oh-My-Zsh 插件列表](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)
- [Powerlevel10k 主题](https://github.com/romkatv/powerlevel10k)
