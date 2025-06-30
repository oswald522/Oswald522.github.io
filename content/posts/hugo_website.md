---
title: "基于Hugo+Blowfish主题搭建的博客网站"
description: "详细记录从零开始搭建个人博客的过程，涵盖Hugo框架、Blowfish主题、Cloudflare Pages部署及域名配置"
date: 2020-10-18T22:43:38+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/83e83b/1600/900.webp"
tags: ["经验记录","技术分享"]
series: "建站技术"
---

## 前言

作为一个技术爱好者，我一直想拥有一个既简洁又高效的个人博客。这次重构博客，我将重点放在了速度、编辑体验和工作流的顺畅性上。经过一番调研，我选择了Hugo作为静态网站生成器，搭配Blowfish主题，并使用Cloudflare Pages进行部署和域名管理。下面，我将详细记录整个搭建过程。

|             项目名称             | 简要介绍    | 类型    | 仓库地址                                                   |                                                                    论文                                                                     |
| :--------------------------: | ------- | ----- | ------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------: |
|           AO-MARL            | 多智能AO   | SHWFS | <https://github.com/Tomeu7/AO-MARL>                    |                                Adaptive Optics control with Multi-Agent Model-Free Reinforcement Learning                                 |
| Integrating-SL-and-RL-for-AO | UNET+RL | PYWFS | <https://github.com/Tomeu7/Integrating-SL-and-RL-for-AO> | Integrating supervised and reinforcement learning for predictive control with an unmodulated pyramid wavefront sensor for adaptive optics |

## 教程基础及前言

在开始之前，确保你已经具备以下基础：

1. **Git**：用于版本控制和代码管理。
2. **Hugo**：一个快速、灵活的静态网站生成器。
3. **GitHub账号**：用于托管代码和版本管理。
4. **Cloudflare账号**：用于部署和域名管理。

## 1. Hugo 新建项目

首先，安装Hugo。如果你还没有安装，建议直接访问[Hugo Release](https://github.com/gohugoio/hugo/releases)选择相应的版本进行下载安装。然后，创建一个新的Hugo项目：

```bash
hugo new site my-blog  # my-blog 是项目的名称
cd my-blog
```

## 2. 安装主题（Git Submodule方法）

Blowfish是一个简洁、现代的Hugo主题。为了便于更新和管理，我们使用Git Submodule方法安装主题。

```bash
git init
git submodule add https://github.com/nunocoracao/blowfish.git themes/blowfish
```

接下来，将主题的示例配置复制到项目根目录，删除根目录当中的`hugo.toml`文件：

```bash
cp themes/blowfish/exampleSite/config/*  .
```

编辑`config.toml`文件，根据你的需求进行个性化配置。

## 3. 主题相关配置

Blowfish主题提供了丰富的配置选项，可以通过编辑`config.toml`文件来定制博客的外观和功能。以下是一些常见的配置项：

```toml
baseURL = "https://yourdomain.com"
title = "我的博客"
theme = "blowfish"

[params]
  description = "这是我的个人博客，分享技术经验和生活感悟。"
  author = "你的名字"
  favicon = "favicon.ico"
```

## 4. 配置域名

### 4.1 上传至GitHub进行历史版本管理

首先，将项目推送到GitHub仓库：

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/my-blog.git
git push -u origin main
```

### 4.2 使用Cloudflare Pages实现在线发布及域名访问

1. 登录Cloudflare，进入Pages页面，选择“Create a project”。
2. 连接到你的GitHub仓库，选择刚刚推送的`my-blog`项目。
3. 在构建设置中，选择Hugo作为框架，并设置构建命令为`hugo`，输出目录为`public`。
4. 点击“Save and Deploy”开始部署。

部署完成后，Cloudflare会提供一个默认的`.pages.dev`域名。如果你有自己的域名，可以在Cloudflare的DNS设置中添加CNAME记录，将你的域名指向Cloudflare Pages提供的域名。

## 附录

以下是一些参考教程和资源，可以进一步学习和探索：

1. [Hugo官方文档](https://gohugo.io/documentation/)
2. [Blowfish主题官方文档](https://blowfish.page/)
3. [Cloudflare Pages官方文档](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/)

希望这篇教程能帮助你顺利搭建自己的博客网站。如果有任何问题或建议，欢迎在评论区留言讨论。

---
