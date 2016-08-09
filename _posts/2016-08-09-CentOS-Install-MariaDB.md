---
layout:   post
title:    CentOS安装配置MariaDB
date:     2016 August 09 10:25:54
category: Linux
tags:     linux sql mariadb mysql
---
# CentOS安装配置MariaDB

## 添加 yum 数据源

使用 yum 的权限要求是 root 用户,如果你不是,那么可以需要 在 shell命令之前加上 sudo, 或者 su root  切换到 super 管理员进行操作. 并可能需要输入密码.

建议命名为 MariaDB.repo 类似的名字：

    cd /etc/yum.repos.d/
    vim /etc/yum.repos.d/MariaDB.repo

然后,写入文件内容:

    # MariaDB 10.0 CentOS repository list - created 2016-06-18 11:21 UTC
    # http://downloads.mariadb.org/mariadb/repositories/
    [mariadb]
    name = MariaDB
    baseurl = http://yum.mariadb.org/10.0/centos6-amd64
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
    gpgcheck=1

该文件的内容是参考官网,并从官网上生成的，[具体地址点这里](https://downloads.mariadb.org/mariadb/repositories/)

选择好操作系统版本之后既可以查看，其他操作系统的安装源也可以在此处查看并设置。

如果服务器不支持https协议，或者gpgkey 保错，确保没问题的话，可以将 gpgcheck=1 修改为 gpgcheck=0,则不进行校验.

## 安装数据库

    # yum remove MariaDB-server MariaDB-client  
    yum -y install MariaDB-server MariaDB-client 

如果要删除旧的数据库可以使用remove, 参数 -y 是确认，不用提示。此处，安装的是服务器和客户端，一般来说安装这两个就可以了。

## 启动数据库

如果不用进行其他的操作，则现在就可以直接启动数据库，并进行测试了。

    # 查看mysql状态;关闭数据库
    # service mysql status
    # service mysql stop
    # 启动数据库
    service mysql start

## 修改root密码

    #  修改root密码  
    mysqladmin -u root password 'root'

因为安装好以后的root密码是空,所以需要设置; 如果是测试服务器,那么你可以直接使用root,不重要的密码很多时候可以设置为和用户名一致，以免忘记了又想不起来。

如果是重要的服务器，请使用复杂密码，例如邮箱，各种自由组合的规则的字符等。

## 登录数据库

    mysql -u root -p

如果是本机,那可以直接使用上面的命令登录，当然，需要输入密码. 如果是其他机器，那么可能需要如下的形式: 

    mysql -h 127.0.0.1 -P 3306 -u root -p

##  简单SQL测试

    >  
    -- 查看MySQL的状态  
    status;  
    -- 显示支持的引擎  
    show engines;  
    -- 显示所有数据库  
    show databases;  
    -- 切换数据库上下文,即设置当前会话的默认数据库  
    use test;  
    -- 显示本数据库所有的表  
    show tables;  
    -- 创建一个表  
    CREATE TABLE t_test (  
      id int(11) UNSIGNED NOT NULL AUTO_INCREMENT,  
      userId char(36),  
      lastLoginTime timestamp,  
      PRIMARY KEY (id)  
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;  
    -- 简单查询  
    select * from t_test;  
    select id,userId from t_test  where userId='admin' ; 

## 修改数据存放目录

mysql, MariaDB 的默认数据存放在 /var/lib/mysql/ 目录下,如果不想放到此处,或者是想要程序和数据分离，或者是磁盘原因,需要切换到其他路径,则可以通过修改 datadir系统变量来达成目的.

    # 停止数据库  
    service mysql stop
      
    # 创建目录,假设没有的话  
    mkdir /usr/local/ieternal/mysql_data
      
    # 拷贝默认数据库到新的位置  
    # -a 命令是将文件属性一起拷贝,否则各种问题  
    cp -a /var/lib/mysql /usr/local/ieternal/mysql_data
      
    # 备份原来的数据  
    cp -a /etc/my.cnf /etc/my.cnf_original
      
    # 其实查看 /etc/my.cnf 文件可以发现  
    # MariaDB 的此文件之中只有一个包含语句  
    # 所以需要修改的配置文件为 /etc/my.cnf.d/server.cnf  
    cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf_original
    vim /etc/my.cnf.d/server.cnf

然后 按 i 进入编辑模式,可以插入相关内容.使用键盘的上下左右键可以移动光标, 编辑完成以后，按 ESC 退出编辑模式(进入命令模式), 然后输入命令:wq 保存并退出

    # 在文件的 mysqld 节下添加内容
    [mysqld]  
    datadir=/usr/local/ieternal/mysql_data/mysql
    socket=/var/lib/mysql/mysql.sock
    #default-character-set=utf8 
    character_set_server=utf8
    slow_query_log=on
    slow_query_log_file=/usr/local/ieternal/mysql_data/slow_query_log.log
    long_query_time=2

其中,也只有 datadir 和 socket 比较重要; 而 default-character-set 是 mysql 自己认识的,而 mariadb5.5 就不认识,相当于变成了 character_set_server

## 创建慢查询日志文件

既然上面指定了慢查询日志文件，我后来看了下MariaDB的err日志，发现MariaDB不会自己创建该文件，所以我们需要自己创建,并修改相应的文件权限(比如 MySQL 采用 mysql用户，可能我们使用 root用户创建的文件,此时要求慢查询日志文件对mysql用户可读可写就行。)

    touch /usr/local/ieternal/mysql_data/slow_query_log.log
    chmod 666 /usr/local/ieternal/mysql_data/slow_query_log.log

然后重新启动MySQL.

    service mysql start