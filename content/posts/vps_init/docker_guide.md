---
title: "[转载]Docker建议教程"
description: ""
date: 2022-03-29T11:59:55+08:00
draft: false
showComments: true
featureimage: https://picsum.photos/seed/ea5af9/1600/900.webp
tags:
  - 技术教程
  - 经验转载
series: ["Linux部署教程"]
series_order: 3
---

{{< alert >}}
以下内容转载来自于 [text](https://linux.do/t/topic/310934)
{{< /alert>}}

## 一、说明

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。

### 1.应用场景

* Web 应用的自动化打包和发布
* 自动化测试和持续集成、发布（例如：部署包运行的测试、编译系统测试等）
* 在服务型环境中部署和调整数据库或其他的后台应用

### 2.docker包含三个基本概念

* 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统；
* 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等；
* 仓库（Repository）：仓库可看成一个代码控制中心，用来保存镜像；

举个栗子：Docker相当于 VM Ware 或者 VM Virtual，镜像相当于Windows/Ubuntu安装镜像，容器相当于已经安装好的Windows/Ubuntu虚拟机；仓库相当于Windows/Ubuntu，Tag（标签）为仓库标签（大部分为仓库版本号）；

### 3.其他概念

* 客户端(Client)：简单说就是Docker安装好的客户端，通过客户端命令可与容器进行通讯；
* （宿）主机(Host)：安装Docker的机器；

## 二、安装（CentOS为例）

### 1.使用官方脚本/一键安装命令

```bash
# 官方命令
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 国内一键安装命令
curl -sSL https://get.daocloud.io/docker | sh
```

### 2.手动安装

```bash
# 卸载旧版本
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-engine

## 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

## 设置官方仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

## 阿里仓库
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

## 清华仓库
yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo

## 选择安装最新版或特定版本Docker Engine-Community
# 1.安装最新版Docker Engine-Community
yum install docker-ce docker-ce-cli containerd.io
# 提示接受 GPG 密钥，请选是

## 2.安装特定版本Docker Engine-CommunityDocker
# 列出仓库中可用版本
yum list docker-ce --showduplicates | sort -r
# 输出如下
# docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
# docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
#
## 安装
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
# 例如 yum install docker-ce-18.06.1 docker-ce-cli-18.06.1 containerd.io

## 启动
systemctl start docker
# 卸载
yum remove docker-ce
# 删除镜像、容器、配置文件等内容
rm -rf /var/lib/docker
```

### 3.离线安装

先从<https://download.docker.com/linux/static/stable/x86_64/> 下载docker-xx.xx.x-ce.tgz，并上传至服务器中

```bash
# 解压安装包
tar -zxvf docker-xx.xx.x-ce.tgz

## 复制文件到/usr/bin
cp docker/* /usr/bin

## 手动添加docker.service配置文件
vi /etc/systemd/system/docker.service
# Docker默认的镜像和容器存储位置在/var/lib/docker中
# 内容如下（其中ExecStart行 registry-mirror需设置为可访问的镜像服务器）：
```

```vim
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
 
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
## Only systemd 226 and above support this version.
##TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target
```

```bash
# 配置docker参数
mkdir -p /etc/docker
vi /etc/docker/daemon.json
```

```bash
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"],
  "log-opts": {
    "max-size": "5m",
    "max-file":"3"
  },
  "exec-opts": ["native.cgroupdriver=systemd"],
  "graph": "/app/docker"
}
## registry-mirrors 为镜像加速地址  无需修改
# log-opts 为日志参数 无需修改
# exec-opts 为docker启动参数  指定cg驱动为systemd   无需修改
# graph docker运行路径 可任意配置
```

```bash
# 授权
chmod +x /etc/systemd/system/docker.service
# 重载守护进程配置
systemctl daemon-reload
# 启动docker
systemctl start docker
# 设置开机自启
systemctl enable docker.service

## 验证docker状态
systemctl status docker
# 查看docker版本
docker -v
```

## 三、Docker安装Ubuntu

在docker官网<https://hub.docker.com/search?q=查找ubuntu，>

```bash
# 拉取镜像
docker pull ubuntu
# 或
docker pull ubuntu:latest
# 查看已经下载的镜像
docker images
# 创建并运行容器
docker run -itd --name ubuntu-test ubuntu:15.10
# 连接容器
docker exec -it ubuntu-test /bin/bash
# 完成

## 查看运行中容器
docker ps
# 查看所有容器
docker ps -a
```

## 四、常用命令

### 1.pull（拉取镜像）命令

```bash
# 拉取最新镜像
docker pull mysql
# 或 docker pull mysql:lasted
# 或 docker pull mysql:8.0.29 指定版本
```

### 2.images（查看镜像）命令

```bash
docker images
# REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
# ubuntu              14.04               90d5884b1ee0        9 weeks ago         188 MB
# ubuntu              15.10               4e3b13c8a266        3 months ago        136.3 MB
# REPOSITORY 仓库名
# TAG 标签（可以理解为版本）
# IMAGE ID 镜像ID
```

### 3.run（创建并运行容器）命令

```bash
mkdir mysql-data
docker run -itd -v /root/mysql-data:/etc/mysql/data -p 13306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql-test mysql:lasted
# -i 以交互模式运行容器，通常与 -t 同时使用；
# -t 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
# -d 后台运行容器，并返回容器ID，如果不加此参数，启动容器时自动连入容器，且退出容器时，容器自动停止；
# -v 等同 --volume ，挂载一个卷（文件或文件夹）到容器中，示例中将root下mysql-data文件夹挂载至容器中/etc/mysql/data，用于存放mysql数据文件
# -p 端口映射，示例中13306为宿主机端口，3306为容器端口，映射后在宿主机中访问13306会自动转发至容器的3306端口
# -e 设置环境变量，示例中将MySQL的Root密码（MYSQL_ROOT_PASSWORD）设置为123456
# --name 为容器指定一个名称

## 注
# run命令中，可使用参数 --privileged=true
# 在现有版本中，容器创建时默认root权限只是容器的部分权限（例如centos不能配置自启动相关，防火墙相关，ftpd相关内容等）
# 加入以上命令使得容器可以获得完整root权限，并能对宿主机进行部分操作
# 使用场景如：创建一个centos容器，使得宿主机以外的其他机器可直接使用ftp工具连入centos容器中
```

### 4.ps命令

```bash
# 查看运行中容器
docker ps
# 查看所有容器
docker ps -a
# CONTAINER ID   IMAGE          COMMAND                ...  PORTS                    NAMES
# 09b93464c2f7   mysql:latest   "####################" ...  80/tcp, 443/tcp          mysql-test
# 96f7f14e99ab   ubuntu:latest  "####################" ...  0.0.0.0:3306->3306/tcp   ubuntu-test
# CONTAINER ID 容器ID
# IMAGE 容器所用镜像
# COMMAND 启动容器时运行的命令
# PORTS 映射的端口
# NAMES 容器名
```

### 5.start（启动）/stop（停止）/restart（重启）命令

```bash
docker start mysql-test
docker stop mysql-test
docker restart mysql-test
# 或 使用ps命令查询容器ID并使用容器ID
docker start 09b93464c2f7
docker stop 09b93464c2f7
docker restart 09b93464c2f7
```

### 6.exec（执行）命令

```bash
docker exec -i -t  mysql-test /bin/bash
# -i 交互模式
# -t 分配伪终端
# mysql-test 运行中的容器名称或ID
# /bin/bash 在容器中运行/bin/bash，一般用此命令来连接终端
docker exec -it mysql-test /bin/sh mysql -u root -p
# 在容器中执行/bin/sh mysql -u root -p
# 意思为在容器终端里执行 mysql -u root -p 命令
# 等同使用 docker exec -it mysql-test /bin/sh 连接至容器中并执行 mysql -u root -p
```

### 7.commit（创建镜像）命令

```bash
# 查看运行中容器
docker ps
# CONTAINER ID   IMAGE          COMMAND                ...  PORTS                    NAMES
# 09b93464c2f7   mysql:latest   "####################" ...  80/tcp, 443/tcp          mysql-test

## 使用运行中容器创建镜像
docker commit -a "runoob.com" -m "my apache" -p 09b93464c2f7  mymysql:v1 
## -a 镜像作者
# -m 镜像文字说明
# -p 创建镜像时将容器暂停
# 09b93464c2f7 容器ID
# mymysql:v1 冒号前为仓库名，冒号后为标签名
```

### 8.save（保存镜像）命令

```bash
# 将制作好的镜像保存至文件，以供其他docker使用
docker save -o my_mysql_v1.tar mymysql:v1
# -o 输出至文件
# my_mysql_v1.tar 文件名
# mymysql:v1 冒号前为仓库名，冒号后为标签名
```

### 9.load（载入镜像）命令

```bash
# 使用
docker load < my_mysql_v1.tar
# 或
docker load -i -q my_mysql_v1.tar
# -i 或 --input 指定载入的文件 用以替换 < （STDIN,标准输入方式）方式载入
# -q 或 --quiet 精简输出信息
```

## 五、dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

### 1.使用dockerfile构建tomcat运行环境

#### 1.1.创建dockerfile文件

```bash
vi dockerfile
```

文件内容如下：

```bash
FROM docker.io/centos
ARG WAR_FILE
ADD ./apache-tomcat-7.0.109.tar.gz /usr/local/
ADD ./jdk-7u80-linux-x64.tar.gz  /usr/local/
COPY ${WAR_FILE} /usr/local/apache-tomcat-7.0.109/webapps/
ENV LOCAL_PATH /usr/local
WORKDIR $LOCAL_PATH
ENV JAVA_HOME=/usr/local/jdk1.7.0_80
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV CATALINA_HOME=/usr/local/apache-tomcat-7.0.109
ENV CATALINA_BASH=/usr/local/apache-tomcat-7.0.109
ENV PATH=/sbin:$JAVA_HOME/bin:$PATH:$CATALINA_HOME/lib:$CATALINA_HOME/bin
EXPOSE 8080
CMD /usr/local/apache-tomcat-7.0.109/bin/startup.sh && tail -F /usr/local/apache-tomcat-7.0.109/logs/catalina.out
```

文件内容说明：

* `FROM <image name>`：声明定制镜像基于哪个基础镜像
* `ARG <参数名>\[=<默认值>\]`：声明参数，在构建镜像时使用--build-arg <参数名>=<值> 来覆盖
* `ADD <host file> <container path>`：将宿主机中文件添加至容器对应路径中，对于gzip, bzip2 以及 xz压缩文件自动解压（例如上面dockerfile中apache-tomcat-7.0.109.tar.gz）
* `COPY <host file> <container path>`：与ADD命令类似，复制文件或路径到容器对应路径中
* `ENV <key> <value>`：设置环境变量
* WORKDIR <工作目录路径>：指定工作目录，使用终端连接容器时，会到此路径
* EXPOSE <端口1> \[<端口2>...\] 声明容器端口（注意：仅作声明，帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。）
* CMD <shell 命令> ：在docker容器运行时执行的命令
* RUN <命令行命令>：在docker构建时执行的命令（注意：多个命令用反斜杠\\分割）

#### 1.2.把容器内需要用到的文件全部复制到容器中

#### 1.3.构建镜像

```bash
docker build --build-arg WAR_FILE=demo-web.war -t centos7:jdk7-tomcat7 .
```

说明：

* build：docker构建命令
* \--build-arg <key=value> 设置参数
* \-t：声明构建的镜像的标签
* 注意：最后面有个小数点，指上下文路径

### 2.使用dockerfile构建python运行环境

#### 2.1.创建dockerfile文件

```bash
vi dockerfile
```

文件内容如下：

```bash
FROM python:*.*.*
ADD ./pip.conf /root/.pip/pip.conf
ADD ./sources.list /etc/apt/sources.list
ADD ./requirements.txt /var/requirements.txt
ADD ./startup.sh /var/startup.sh
ADD ./install.sh /var/install.sh
WORKDIR /var/app
RUN /var/install.sh
CMD /var/startup.sh
```

文件内容说明：

* pip.conf：pip源定义
* sources.list：配置容器环境包下载的源
* requirements.txt：python所需插件列表
* startup.sh：启动命令

```bash

##!/bin/bash
set -e
service cron start
pip install -r requirements.txt
python manage.py runserver 0.0.0.0:8000
```

* install.sh：构建容器时需要执行的shell命令

```bash
#!/bin/bash
set -e
apt update
apt install -y cron libsasl2-dev python-dev libldap2-dev libssl-dev nodejs
pip install --upgrade pip
pip install -r /var/requirements.txt
chmod -R 777 /var/startup.sh
```

#### 2.2.把容器内需要用到的文件全部复制到容器中

#### 2.3.构建镜像

见本文1.3章。

## 六、安装docker-compose

下载路径 [https://github.com/docker/compose/tags](https://github.com/docker/compose/tags) （找版本号下 docker-compose-linux-x86\_64 文件下载）

```bash
# 文件上传
# 复制文件到/usr/bin
cp docker-compose-linux-x86_64 /usr/bin/docker-compose

## 授予执行权限
chmod u+x /usr/bin/docker-compose

## 检查是否正常
docker-compose -v
```

## 七、安装Harbor

下载路径 [https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases) （文件如：harbor-offline-installer-v2.5.3.tgz）

```bash
# 创建文件夹
mkdir -p /app/harbor
# 文件上传

## 文件解压
cd /app
tar -zxvf harbor-offline-installer-v2.5.3.tgz
cd harbor
# 创建配置文件
cp harbor.yml.tmpl harbor.yml

## 修改配置文件
vi harbor.yml
```

```bash
# host 配置 可为本机ip 或 域名
hostname: 192.168.56.102

## http配置
http:
  port: 11080
# https配置
#https:
##  port: 11443
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

## 控制台密码
harbor_admin_password: Harbor12345

## 数据库配置
database:
  password: root123
  max_idle_conns: 100
  max_open_conns: 900

## 安装位置 || 镜像存储路径
data_volume: /app/registry

## 镜像扫描
trivy:
  ignore_unfixed: false
  skip_update: false
  offline_scan: false
  insecure: false

## 定时任务
jobservice:
  max_job_workers: 10

## 提醒
notification:
  webhook_job_max_retry: 10

## 图表
chart:
  absolute_url: disabled

## 日志
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor

## 版本
_version: 2.5.0

## 代理
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy

## 上传配置
upload_purging:
  enabled: true
  age: 168h
  interval: 24h
  dryrun: false
```

```bash
# 安装
sh install.sh

## 输出以下内容表示成功
✔ ----Harbor has been installed and started successfully.----
## 浏览器访问http://192.168.56.102:11080/
## 用户名 amdin
# 密码   Harbor12345
```

```bash
# docker 配置私有仓库 增加以下配置
vi /etc/docker/daemon.json
```

```bash
{
  "insecure-registries": [
    "http://192.168.56.102:11080"
  ]
}
```

```bash
# 重载docker
systemctl reload docker
```
