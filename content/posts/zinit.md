---
title: "Zinit 配置留存备份"
date: 2025-03-15T16:41:49+08:00
draft: false
description: ""
showComments: true
featureimage: "https://picsum.photos/seed/8701f424/1600/900.webp"
tags: ["Linux", "Shell", "教程配置"]
---

## Zinit 简介

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
    command git clone --depth 1 https://bgithub.xyz/zdharma-continuum/zinit "$HOME/.local/share/zinit/zinit.git" && \
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
## Sorts
alias llm='eza -lbGF --git --sort=modified'
alias lld='eza -lbhHFGmuSa --group-directories-first' 
alias du='dust'
alias cat='batcat'
alias unzip='unar'
alias rm="trash-put"



########################################################
##### 自定义函数区
########################################################
setopt Autocd  #不需要输入cd
setopt HIST_IGNORE_ALL_DUPS #  移除重复的命令历史
```
