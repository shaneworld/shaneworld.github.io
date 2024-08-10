---
title: MariaDB 数据库
author: shane
date: 2024-08-09
description: Arch Linux 官方默认的 MySQL 分支 MariaDB
categories: [SQL]
tags: [MySQL, MariaDB, archlinux]
comments: true
toc: true
---

## MariaDB 

MariaDB 由 MySQL 的原始开发者 Michael "Monty" Widenius 在 2009 年创立，以应对 Oracle 收购 MySQL 后可能引发的担忧。MariaDB 是 MySQL 的一个开源分支，旨在保持与 MySQL 的兼容性，但由独立的社区维护和开发。

MariaDB 和 MySQL 在 SQL 语法、API、协议等方面高度兼容，大多数情况下，应用程序可以在不做或仅做少量修改的情况下在两者之间切换。MariaDB 和 MySQL 使用相同的数据文件格式，数据库文件可以在两者之间互相迁移。

随着时间的推移，MariaDB 引入了许多 MySQL 中没有的特性，例如一些新存储引擎、查询优化器改进以及更多的安全功能。自 2013 年起，MariaDB 就被 Arch Linux 当作官方默认的 MySQL 实现。

[MariaDB wiki](https://wiki.archlinuxcn.org/wiki/MariaDB)

## 安装和启动

使用包管理器安装

```shell
sudo pacman -S mariadb
```

启动服务

```shell
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
systemctl start mariadb.service
```

## 配置

```shell
sudo mariadb -u root -p
```

### 添加新用户

创建一个密码为 `some_pass` 的 `monty` 用户的示例，并赋予 monty 用户对 mydb 数据库的完全操作权限

```shell
MariaDB> CREATE USER 'monty'@'localhost' IDENTIFIED BY 'some_pass';
MariaDB> GRANT ALL PRIVILEGES ON mydb.* TO 'monty'@'localhost';
MariaDB> FLUSH PRIVILEGES;
MariaDB> quit
```

设置好后可以以设置的用户名称进入 mariadb：

```shell
mariadb -u monty -p
Enter password:
```

### 配置默认监听的 IP 地址

Linux 中默认配置文件在 `/etc/my.cnf.d/`，在 `server.cnf` 中：

```
[mysqld]
bind-address = 127.0.0.1
```

### 启用自动补全

编辑 `/etc/my.cnf.d/mysql-clients.cnf`，在 mysql 下添加 `auto-rehash`。

## 常用命令

全局操作一般需要以 `root` 用户身份登录 MariaDB。

- 检查用户权限：

    ```sql
    SELECT user, host FROM mysql.user WHERE user = 'shane';
    ```

- 设置或更改用户密码：

    ```sql
    SET PASSWORD FOR 'shane'@'localhost' = PASSWORD('new_password');
    ```

- 创建新数据库

    ```sql
    CREATE DATABASE mydatabase;
    -- 或
    mariadb -u username -p -e "CREATE DATABASE mydatabase;"
    ```

- 导入 SQL 文件
    
    ```shell
    mariadb -u username -p mydatabase < /path/to/yourfile.sql
    ```

- 查看数据库中的表格

    ```sql
    USE mydatabase;
    SHOW TABLES;
    ```

- 查看特定用户的权限：

    ```sql
    SHOW GRANTS FOR 'shane'@'localhost';
    ```

- 查看当前使用的端口号：

    ```sql
    SHOW VARIABLES LIKE 'port';
    ```

- 查看当前监听的 IP 地址：

    ```sql
    SHOW VARIABLES LIKE 'bind_address';
    ```

## 未完待续...
