---
title: "MySQL\\MariaDB数据库安装教程"
description: "记录MySQL和MariaDB的安装过程"
date: 2025-06-13T20:32:19+08:00
Lastmod: 2025-06-13T20:32:19+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/3be906/1600/900.webp"
tags: ["数据库软件"]
series: ["数据库教程"]
# series_order: 1
---


## MySQL/MariaDB数据库安装教程

## 前言

MySQL和MariaDB是当前最流行的开源关系型数据库管理系统，广泛应用于Web应用程序、数据仓库和嵌入式系统中。MySQL由Oracle公司开发维护，而MariaDB则是MySQL的一个分支，由MySQL的原始开发者创建，完全兼容MySQL并添加了许多新特性。

本教程将详细介绍在Windows系统下通过ZIP压缩包方式安装和配置MySQL和MariaDB数据库的完整流程。

## MySQL安装教程（ZIP安装方式）

### 1. 下载MySQL社区版

1. 访问MySQL官方网站：<https://dev.mysql.com/downloads/mysql/>
2. 在"MySQL Community (GPL) Downloads"部分选择"MySQL Community Server"
3. 选择适合您系统的版本（通常选择Windows (x86, 64-bit), ZIP Archive）
4. 点击"Download"按钮下载

### 2. 解压安装包

1. 将下载的ZIP文件解压到您选择的目录（例如：`C:\mysql`）
2. 建议路径不要包含空格或特殊字符

### 3. 创建配置文件

1. 在MySQL根目录下创建`my.ini`文件
2. 添加以下基本配置内容：

```ini
[mysqld]
# 设置MySQL安装目录
basedir=C:/mysql
# 设置MySQL数据存储目录
datadir=C:/mysql/data
# 设置端口号
port=3306
# 允许最大连接数
max_connections=200
# 服务端默认字符集
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 设置SQL模式
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

### 4. 初始化MySQL

1. 以管理员身份打开命令提示符
2. 切换到MySQL的bin目录：

   ```shell
   cd C:\mysql\bin
   ```

3. 执行初始化命令：

   ```shell
   mysqld --initialize --console
   ```

4. 记下生成的临时密码（在最后一行显示）

### 5. 安装MySQL服务

1. 在命令提示符中执行：

   ```shell
   mysqld --install
   ```

2. 启动MySQL服务：

   ```shell
   net start mysql
   ```

### 6. 修改root密码

1. 登录MySQL：

   ```shell
   mysql -u root -p
   ```

2. 输入之前记录的临时密码
3. 修改密码：

   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
   ```

### 7. 环境变量配置（可选）

1. 将`C:\mysql\bin`添加到系统PATH环境变量中
2. 这样可以在任何目录下直接使用mysql命令

## MariaDB安装配置教程（ZIP安装方式）

### 1. 下载MariaDB

1. 访问MariaDB官方网站：<https://mariadb.org/download/>
2. 选择适合您系统的版本（Windows x86_64 ZIP package）
3. 点击下载

### 2. 解压安装包

1. 将下载的ZIP文件解压到您选择的目录（例如：`C:\mariadb`）
2. 建议路径不要包含空格或特殊字符

### 3. 创建配置文件

1. 在MariaDB根目录下创建`my.ini`文件
2. 添加以下基本配置内容：

```ini
[mysqld]
# 设置MariaDB安装目录
basedir=C:/mariadb
# 设置MariaDB数据存储目录
datadir=C:/mariadb/data
# 设置端口号（默认3306）
port=3306
# 字符集设置
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
# 存储引擎
default-storage-engine=InnoDB
# 日志设置
log-error=C:/mariadb/data/mariadb.err
```

### 4. 初始化MariaDB

1. 以管理员身份打开命令提示符
2. 切换到MariaDB的bin目录：

   ```shell
   cd C:\mariadb\bin
   ```

3. 执行初始化命令：

   ```shell
   mysql_install_db.exe --datadir=C:/mariadb/data --service=MySQL --password=your_password
   ```

### 5. 安装MariaDB服务

1. 在命令提示符中执行：

   ```shell
   mysqld --install MariaDB
   ```

2. 启动MariaDB服务：

   ```shell
   net start MariaDB
   ```

### 6. 安全配置

1. 执行安全配置向导：

   ```shell
   mysql_secure_installation
   ```

2. 按照提示设置root密码、移除匿名用户、禁止root远程登录等

### 7. 验证安装

1. 登录MariaDB：

   ```
   mysql -u root -p
   ```

2. 输入密码后应能看到MariaDB命令行提示符

## 常见问题解决

### 1. 服务启动失败

- 确保端口3306未被占用
- 检查错误日志（通常在data目录下的.err文件）
- 确保data目录为空或不存在（初始化前）

### 2. 忘记root密码

1. 停止MySQL/MariaDB服务
2. 创建临时文件`reset.txt`，内容为：

   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
   ```

3. 以跳过权限检查方式启动：

   ```
   mysqld --init-file=C:\path\to\reset.txt --console --skip-grant-tables
   ```

4. 启动服务后删除临时文件

### 3. 字符集问题

确保在配置文件中正确设置了字符集：

```ini
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
```

## 性能优化建议

1. 根据服务器内存调整缓冲池大小：

   ```ini
   innodb_buffer_pool_size=1G  # 对于4GB内存的服务器
   ```

2. 启用查询缓存：

   ```ini
   query_cache_size=64M
   query_cache_type=1
   ```

3. 调整连接数：

   ```ini
   max_connections=100
   ```

## 卸载步骤

### MySQL卸载

1. 停止服务：

   ```shell
   net stop mysql
   ```

2. 删除服务：

   ```shell
   sc delete mysql
   ```

3. 删除安装目录和数据目录

### MariaDB卸载

1. 停止服务：

   ```shell
   net stop MariaDB
   ```

2. 删除服务：

   ```shell
   sc delete MariaDB
   ```

3. 删除安装目录和数据目录

## 总结

通过本教程，您已经学会了如何在Windows系统下使用ZIP压缩包方式安装和配置MySQL和MariaDB数据库。这种安装方式相比安装程序更加灵活，适合需要自定义配置的开发环境。安装完成后，您可以使用各种MySQL客户端工具连接数据库，开始您的数据库开发之旅。
