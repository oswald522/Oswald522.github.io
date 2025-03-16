---
title: "自建服务一览及搭建配置文件"
date: 2025-03-15T17:41:40+08:00
draft: true
description: ""
showComments: true
featureimage: "https://picsum.photos/seed/907fd2/1600/900.webp"
tags: ["瞎折腾","服务器","Linux VPS"]
series: ["服务器自建"]
series_order: 1
---

## 推荐的一些自建服务及配置文件

---

### 1. **Alist**

**简介**：官网为[https://alist.nn.ci/](https://alist.nn.ci/)，Alist 是一个支持多种存储的目录文件列表程序，支持本地存储、对象存储、网盘等，可以用于离线下载和文件管理。

```yaml
services:
  alist:
    image: xhofe/alist:latest
    container_name: alist
    volumes:
      - $PWD/data:/opt/alist/data
    ports:
      - "5244:5244"
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
```

---

### 2. **Aria2**

简介: [Aria2](https://aria2.github.io/)是一个轻量级的多协议命令行下载工具，支持 HTTP、FTP、BitTorrent 等协议，常用于离线下载。

```yaml

services:
  aria2:
    image: p3terx/aria2-pro:latest
    container_name: aria2
    volumes:
      - $PWD/config:/config
      - $PWD/downloads:/downloads
    ports:
      - "6800:6800"
    environment:
      - RPC_SECRET=your_secret_key
    restart: unless-stopped
```

---

### 3. **Caddy**

简介:[Caddy](https://caddyserver.com/) 是一个现代化的 Web 服务器，支持自动 HTTPS、反向代理等功能，配置简单且性能优秀。

```yaml

services:
  caddy:
    image: caddy:latest
    container_name: caddy
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - $PWD/data:/data
      - $PWD/config:/config
    ports:
      - "80:80"
      - "443:443"
    restart: unless-stopped
```

---

### 4. **Vaultwarden**

简介:[Vaultwarden](https://github.com/dani-garcia/vaultwarden) 是 Bitwarden 密码管理器的轻量级实现，兼容 Bitwarden 客户端，适合自建密码管理服务。

```yaml

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    volumes:
      - $PWD/data:/data
    ports:
      - "8080:80"
    environment:
      - ADMIN_TOKEN=your_admin_token
    restart: unless-stopped
```

---

### 5. **青龙脚本**

简介:[青龙](https://github.com/whyour/qinglong)脚本是一个支持定时任务的脚本管理平台，常用于京东签到、自动化脚本等。

```yaml

services:
  qinglong:
    image: whyour/qinglong:latest
    container_name: qinglong
    volumes:
      - $PWD/config:/ql/config
      - $PWD/scripts:/ql/scripts
      - $PWD/log:/ql/log
    ports:
      - "5700:5700"
    environment:
      - ENABLE_HANGUP=true
      - ENABLE_WEB_PANEL=true
    restart: unless-stopped
```

---

### 6. **Nextcloud**

简介:[Nextcloud](https://nextcloud.com/)是一个自建云存储和协作平台，支持文件存储、日历、联系人、笔记等功能。

```yaml

services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    volumes:
      - $PWD/data:/var/www/html
    ports:
      - "8080:80"
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your_password
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mariadb:latest
    container_name: nextcloud_db
    volumes:
      - $PWD/db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=your_root_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your_password
    restart: unless-stopped
```

### 7.Adminer

简介： Adminer 是一个类似于PHPMyAdmin，主要用于数据库的管理，支持各种类型的数据库，如MySQL、PostSQL、Sqlite 等等。

```yaml
services:
  # mariadb:
  #   image: mariadb:lts
  #   container_name: mariadb-lts
  #   restart: always
  #   ports:
  #     - "3306:3306" # 将容器的 3306 端口映射到宿主机
  #   environment:
  #     MYSQL_ROOT_PASSWORD: password # 设置 root 用户密码
  #     MYSQL_DATABASE: fastapi      # （可选）自动创建的数据库
  #     MYSQL_USER: fast                   # （可选）创建的普通用户
  #     MYSQL_PASSWORD: password           # （可选）普通用户的密码
  #   volumes:
  #     - ./data:/var/lib/mysql                 # 持久化数据到宿主机的 ./data 目录
  #     # - ./my.cnf:/etc/mysql/my.cnf:ro         # （可选）挂载自定义配置文件
  adminer:
    # image: shyim/adminerevo
    image: adminer
    container_name: adminer
    restart: always
    ports:
      - "8080:8080" # 将容器的 8080 端口映射到宿主机
    # 以下内容仅仅适用SQLite等不设密码的数据库
    volumes:
      - $PWD/one-api.db:/db/one-api.db
      - $PWD/login-password-less.php:/var/www/html/plugins-enabled/login-password-less.php
```

当前 Adminer 不支持 sqlite 等不加密的数据库连接，因此需要修改相应的php-plugins。官网给出了三种方法，在这里提供个人认为最简单的方法，即 增加`login-password-less.php`，实现默认`admin`密码登录连接。

```php
<?php
require_once('plugins/login-password-less.php');

/** Set allowed password
 * @param string result of password_hash
 */
return new AdminerLoginPasswordLess(
 $password_hash = password_hash("admin", PASSWORD_DEFAULT)
);
```
