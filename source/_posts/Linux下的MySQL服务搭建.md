---
title: Linux下的MySQL服务搭建
date: 2023-07-06 17:51:27
tags:
  - MySQL
categories:
  - xieajiu
description: "使用压缩包的方式在Ubuntu上安装MySQL，并注册服务"
---

### 1、获取Ubuntu对应版本的MySQL安装包

熟悉的步骤，登录[MySQL官网](https://www.mysql.com/)，进行安装包的下载。不过这次不使用RPM或者APT的方式进行MySQL的安装。需要先获取`glibc`版本

```shell
ldd --version
```
{% asset_img ldd_version.png glic的版本 %}

我这边的版本是`2.35`。我使用的Ubuntu版本是22.04，我选择安装的MySQL版本是8.0.33，使用的安装包是

{% asset_img MySQL_Version.png MySQL下载版本 %}

### 2、安装并配置MySQL

解压MySQL的安装包`tar -zxvf mysql-xxx-xx-xx.tar.gz(会将压缩包解压到当前目录)`，会得到如下目录结构：

```shell
├── LICENSE          -
├── README           -
├── bin              - 执行文件所在目录，mysql,mysqld等等
├── data             - 我自己创建的，用来存储MySQL的数据，在my.cnf中配置
├── docs             - 
├── include          - 
├── lib              - 
├── logs             - 自己创建的，用于存放error日志，在my.cnf中配置
├── man              - 
├── my.cnf           - 需要自己创建，这个是MySQL的配置文件
├── share            - 
└── support-files    - 
```

> 接下来用`~`表示MySQL解压目录的绝对路径，小伙伴要自己替换啊

创建MySQL的配置文件

```shell
touch my.cnf
```

编写配置文件

```shell
[mysqld]
# 设置端口，默认3306
port = 13306
# 服务端字符集
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
# MySQL的Base目录
basedir = ~
# 数据存储目录
datadir = ~/data
# 日志，默认在/var/log/mysql.log
log-error = ~/error.log
socket = ~/mysql.sock
pid-file = ~/mysqld.pid
default-storage-engine = InnoDB
max_connections = 200
# 设置时区
default-time-zone = '+08:00'

[client]
default-character-set = utf8mb4
```

接下来开始初始化MySQL

```shell
~/bin/mysqld --initialize --defaults-file=~/my.cnf
```

> 我这里并没有创建mysql用户和用户组，其实只要不使用root用户就可以了。要是自己玩的话，使用root也没有关系。我这边用的Linux是Ubuntu，所以我是使用的用户是我自己的账户。
>
> 关于MySQL初始化还有一个命令，`--initialize-insecure`这个会生成一个空密码的root用户。
>
> 也可以加上 `--console`在控制台打印。

如果控制台没有打印日志，可以去`/var/log/mysql.log`或者配置文件中的`log-error`，可以找到：`[Note] A temporary password is generated for root@localhost:  XxxXxxXxx`其中**XxxXxxXxx**就是密码。

然后登录启动MySQL，登录并重置root用户密码。

```shell
# 启动MySQL服务，这是在后台运行的。
~/bin/mysqld_safe --defaults-file=~/my.cnf

# 登录MySQL
~/bin/mysql -uroot -p密码 -P 端口 -S ~/mysql.sock
```

若没有配置`mysql.sock`文件则不需要。然后重置root密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_new_password';
FLUSH PRIVILEGES;
```

关闭临时启动的MySQL服务，防止在接下来的配置中出现端口冲突

```shell
~/bin/mysqladmin -uroot -p密码 shutdown
```

### 3、配置MySQL服务

**我使用的是`systemctl`来管理服务，其他的没有尝试过**

创建`mysqld.service`文件

```shell
touch mysqld.service
```

编写文件内容

```shell
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
# User=mysql
# Group=mysql

Type=forking

PIDFile=/home/xieajiu/DevelopmentTools/Database/MySQL/8033/mysqld.pid

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Start main service
ExecStart=~/mysqld --defaults-file=~/my.cnf  --daemonize

# Use this to switch malloc implementation
# EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 5000

Restart=on-failure

RestartPreventExitStatus=1

PrivateTmp=false
WorkingDirectory=/home/xieajiu/DevelopmentTools/Database/MySQL/8033
```

> 其中的`~`是mysql的解压目录

然后配置MySQL服务

```shell
cp ~/mysqld.service /etc/systemd/system/

# 加载配置
systemctl daemon-reload
#启动MySQL服务
systemctl start mysqld
#设置服务开机自启
systemctl enable mysqld
```

ok，到此结束！
