---
title: Overleaf社区版搭建基础教程
description: 具体描述Overleaf的搭建过程及教程
date: 2024-03-15T13:57:59+08:00
Lastmod: 2025-04-20T08:06:45+08:00
draft: false
showComments: true
featureimage: https://picsum.photos/seed/bb448b/1600/900.webp
tags:
  - Overleaf
  - 常用软件
  - 技术教程
  - 教程配置
---

## 概述

Overleaf是一个在线的LaTeX编辑器，支持多人协作编辑，提供了丰富的模板和宏包，可以在线编译LaTeX文档。Overleaf有免费版和付费版，免费版有一些限制，如编译时间限制、项目数限制等。Overleaf的付费版提供了更多的功能，如无限编译时间、无限项目数、Git同步等。

开源版本的Overleaf称为"Overleaf Community Edition"，简称"Overleaf CE"。它是Overleaf团队推出的开源LaTeX协作编辑器，可让用户在自己的服务器上创建类似Overleaf的在线LaTeX编辑环境。这种自托管方式赋予用户在内部网络或私人服务器上创建、编辑和共享LaTeX文档的完全控制权。

- 数据隐私和安全性： 在某些情况下，可能对数据隐私和安全性有严格要求，不希望文档和敏感信息存储在第三方云服务上。独立部署Overleaf允许您完全掌握数据存储和访问，确保合规和安全标准。
- 机构内部部署： 一些大学、研究机构或企业可能需要在内部网络提供LaTeX编辑和协作功能，以支持学术、研究和文档创作。独立部署Overleaf能够满足这一需求，使机构内部用户轻松创建和编辑LaTeX文档。
定制需求： 如果您有特定的定制需求，希望对编辑器界面、功能或工作流程进行个性化定制，独立部署Overleaf提供更大的灵活性，可满足各种需求，实现编辑环境的定制化。
- 带宽和访问限制： 在某些地区或网络环境中，外部云服务的访问可能受到带宽或访问限制。独立部署Overleaf可为用户提供更流畅的内部网络访问体验。
- 成本效益： 对于一些组织而言，长期使用第三方云服务可能会导致高昂费用。独立部署Overleaf有助于节省成本，尤其是在大规模使用情况下。
- 技术支持和控制： 独立部署Overleaf让您在技术层面上更具掌控力，可以根据需要进行维护、升级和修复，与内部技术团队直接合作解决问题。

## 前提

1. 使用Linux或者Window服务器均可，性能稍强。（搭建的环境使用甲骨文的4H24GArm服务器,性能足够使用，操作系统使用Debian12.）
2. 安装Docker和Docker Compose。该部分比较简单，直接省略。如果是linux系统，直接运行`curl -fsSL https://get.docker.com | sh --mirror=Aliyun`即可。
3. 具有自己的二级域名（[自建latex编译服务](https://latex.baos.eu.org)），使用**caddy**进行反向代理和自动https。

## 安装方法

使用Docker来方便快捷地搭建Overleaf社区版服务。这里有两种安装方式：一种是直接使用Overleaf社区版仓库里提供的docker-compose.yml文件，另一种是使用Overleaf官方提供的工具箱。Overleaf官方推荐使用第二种方式，但这里两种方式都介绍一下。

### 方法一：使用docker-compose.yml文件安装

```yaml
待续
```

可以在Overleaf社区版仓库中找到docker-compose.yml文件，下载到本地，然后使用命令`docker-compose up -d`启动服务。这里可能需要根据自己的实际情况修改docker-compose.yml文件，如修改端口、数据卷路径等。

### 方法二：使用Overleaf工具箱安装

Overleaf官方提供了一个工具箱，包装了一些常用的docker命令，可以初始化、启动、停止、诊断、升级Overleaf服务。虽然我觉得相比直接使用docker-compose.yml，这个工具箱使部署docker的流程更复杂了，但这个工具箱的确提供了更多更灵活的定制选项。

使用这个工具箱部署Overleaf社区版服务，可以参考Overleaf社区版工具箱文档。简单来讲，有以下几步：

1. 下载工具箱`git clone https://github.com/overleaf/toolkit.git`
2. 进入工具箱目录`cd toolkit`
3. 初始化安装配置`bin/init`，运行此命令，将在当前目录下生成一个config文件夹，里面包含了三个配置文件：
   - overleaf.rc：Overleaf配置文件。用户可以在这个文件中配置Overleaf的一些参数，如端口、数据卷路径等。
   - variables.env：环境变量配置文件
   - version：选择Overleaf版本。注意在5.0.0版本之后，Overleaf将原来的ShareLaTeX商标替换为Overleaf商标，所以如果环境变量配置文件中如果使用OVERLEAF前缀的变量，需要选择5.0.0之后的版本。
4. 在修改完配置文件后，运行命令`bin/up`启动服务，在运行后可以看到服务的输出日志。如果想要停止服务，可以按Ctrl+C。 **检查无误之后，使用`bin/up -d`以后台模式启动服务。
5. 如果想要停止服务，可以使用命令`bin/stop`。
6. 配置Nginx反向代理（可选）。Overleaf社区版的docker-compose.yml文件中有一个nginx容器，如果使用Overleaf工具箱，在overleaf.rc配置文件中也可以配置Nginx。如果需要TSL/SSL加密，可以使用在初始化时使用bin/init --tls命令。这样会在config目录中生成一个nginx文件夹，里面包含了Nginx的配置文件和作为示例的SSL证书。

> 由于之前在服务器上已经部署了单独的Caddy服务来管理所有的网站，所以不使用Nginx服务来反向代理Overleaf服务。

## 其他常见问题

### 1. 升级TexLive

Overleaf的docker镜像中自带了一个基础版本的TeX Live，但是这个版本可能不包含所有的宏包。如果需要编译一些特殊的LaTeX文档，可能需要安装完整版的TeX Live。可参考Overleaf工具箱文档中升级Tex Live的文档。安装完整版TeX Live主要步骤如下：

```shell
docker exec -it overleaf bash #进入Overleaf容器
tlmgr --version #查看当前TeX Live版本
tlmgr install scheme-full  #更新TeX Live
tlmgr path add
```

如果不执行路径添加的命令，在Overleaf中编译LaTeX文档时，可能会出现无法编译EPS图片等问题。做完上述更改后，升级的TeX Live只会保存在当前的容器中，如果容器被删除，这些更改也会丢失。如果想要将这些更改保存到镜像中并在之后创建容器时使用，可以执行下面的操作：
`docker commit sharelatex sharelatex/sharelatex:with-texlive-full`
然后直接在`overleaf-toolkit/config/overleaf.rc`文件当中修改`OVERLEAF_IMAGE_NAME=sharelatex/sharelatex:with-texlive-full`，按照上面的方法运行`/bin/up`或者`/bin/up -d`实现服务重新部署。

### 2. 更改Overleaf实例为简体中文

直接增加`overleaf-toolkit/config/variables.env`文件内部的环境变量，设置`OVERLEAF_SITE_LANGUAGE=zh-CN`即可

### 3. ARM架构镜像选择

实际操作当中发现，ARM版本的Overleaf版本总存在各种各样的问题，因此需要选择合适的版本镜像。具体来说，需要修改文件`overleaf-toolkit/config/overleaf.rc`当中修改变量`OVERLEAF_IMAGE_NAME=kylindemons/sharelatex`（镜像当前版本为5.1.0，官方工具箱最新版本为5.4.0，但是并未更新太多的新特性，不必特意求更新，稳定是第一位的）。

### 出现Email Protect提示

出现这种情况是因为使用CF小黄云的代理，为避免邮箱受到垃圾邮件的骚扰启用的保护措施。将邮箱地址使用`<!--email_off-->contact@example.com<!--/email_off-->`进行包裹即可

## 参考资料

1. [Overleaf](https://github.com/overleaf/overleaf)
2. [Quick-Start-Guide](https://github.com/sharelatex/sharelatex/wiki/Quick-Start-Guide)