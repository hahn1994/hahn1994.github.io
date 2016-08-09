---
layout:   post
title:    CentOS安装配置VSFTP服务器
date:     2016 August 09 10:40:35
category: Linux
tags:     linux vsftp yum ftp
---

# CentOS 安装配置VSFTP服务器

## 安装vsftpd

    [root@localhost ~]# yum -y install vsftpd

## 配置vsftpd.conf文件

    [root@localhost ~]# vi /etc/vsftpd/vsftpd.conf
    # Example config file /etc/vsftpd/vsftpd.conf
    # 
    # The default compiled in settings are fairly paranoid. This sample file 
    # loosens things up a bit, to make the ftp daemon more usable. 
    # Please see vsftpd.conf.5 for all compiled in defaults. 
    # 
    # READ THIS: This example file is NOT an exhaustive list of vsftpd options. 
    # Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's 
    # capabilities. 
    # 
    # Allow anonymous FTP? (Beware - allowed by default if you comment this out). 
    #anonymous_enable=YES
    # 
    # Uncomment this to allow local users to log in. 
    local_enable=YES
    # 
    # Uncomment this to enable any form of FTP write command. 
    write_enable=YES
    # 
    # Default umask for local users is 077. You may wish to change this to 022, 
    # if your users expect that (022 is used by most other ftpd's) 
    local_umask=022
    # 
    # Uncomment this to allow the anonymous FTP user to upload files. This only 
    # has an effect if the above global write enable is activated. Also, you will 
    # obviously need to create a directory writable by the FTP user. 
    #anon_upload_enable=YES
    # 
    # Uncomment this if you want the anonymous FTP user to be able to create 
    # new directories. 
    #anon_mkdir_write_enable=YES
    # 
    # Activate directory messages - messages given to remote users when they 
    # go into a certain directory. 
    dirmessage_enable=YES
    # 
    # The target log file can be vsftpd_log_file or xferlog_file. 
    # This depends on setting xferlog_std_format parameter 
    xferlog_enable=YES
    # 
    # Make sure PORT transfer connections originate from port 20 (ftp-data). 
    connect_from_port_20=YES
    # 
    # If you want, you can arrange for uploaded anonymous files to be owned by 
    # a different user. Note! Using "root" for uploaded files is not 
    # recommended! 
    #chown_uploads=YES
    #chown_username=whoever
    # 
    # The name of log file when xferlog_enable=YES and xferlog_std_format=YES 
    # WARNING - changing this filename affects /etc/logrotate.d/vsftpd.log 
    #xferlog_file=/var/log/xferlog
    # 
    # Switches between logging into vsftpd_log_file and xferlog_file files. 
    # NO writes to vsftpd_log_file, YES to xferlog_file 
    xferlog_std_format=YES
    # 
    # You may change the default value for timing out an idle session. 
    idle_session_timeout=600
    # 
    # You may change the default value for timing out a data connection. 
    data_connection_timeout=120
    # 
    # It is recommended that you define on your system a unique user which the 
    # ftp server can use as a totally isolated and unprivileged user. 
    #nopriv_user=ftpsecure
    # 
    # Enable this and the server will recognise asynchronous ABOR requests. Not 
    # recommended for security (the code is non-trivial). Not enabling it, 
    # however, may confuse older FTP clients. 
    #async_abor_enable=YES
    # 
    # By default the server will pretend to allow ASCII mode but in fact ignore 
    # the request. Turn on the below options to have the server actually do ASCII 
    # mangling on files when in ASCII mode. 
    # Beware that on some FTP servers, ASCII support allows a denial of service 
    # attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd 
    # predicted this attack and has always been safe, reporting the size of the 
    # raw file. 
    # ASCII mangling is a horrible feature of the protocol. 
    ascii_upload_enable=YES
    ascii_download_enable=YES
    # 
    # You may fully customise the login banner string: 
    #ftpd_banner=Welcome to lightnear FTP service. 
    # 
    # You may specify a file of disallowed anonymous e-mail addresses. Apparently 
    # useful for combatting certain DoS attacks. 
    #deny_email_enable=YES
    # (default follows) 
    #banned_email_file=/etc/vsftpd/banned_emails
    # 
    # You may specify an explicit list of local users to chroot() to their home 
    # directory. If chroot_local_user is YES, then this list becomes a list of 
    # users to NOT chroot(). 
    chroot_local_user=YES
    #chroot_list_enable=YES
    # (default follows) 
    #chroot_list_file=/etc/vsftpd/chroot_list
    # 
    # You may activate the "-R" option to the builtin ls. This is disabled by 
    # default to avoid remote users being able to cause excessive I/O on large 
    # sites. However, some broken FTP clients such as "ncftp" and "mirror" assume 
    # the presence of the "-R" option, so there is a strong case for enabling it. 
    ls_recurse_enable=YES
    # 
    # When "listen" directive is enabled, vsftpd runs in standalone mode and 
    # listens on IPv4 sockets. This directive cannot be used in conjunction 
    # with the listen_ipv6 directive. 
    listen=YES
    listen_port=2521
    # 
    # This directive enables listening on IPv6 sockets. To listen on IPv4 and IPv6 
    # sockets, you must run two copies of vsftpd with two configuration files. 
    # Make sure, that one of the listen options is commented !! 
    #listen_ipv6=YES
    
    pam_service_name=vsftpd
    userlist_enable=YES
    userlist_deny=NO
    local_root=/var/public_root
    tcp_wrappers=YES    
    use_localtime=YES

## 增加FTP帐户

    [root@localhost ~]# useradd cent -s /sbin/nologin
    [root@localhost ~]# passwd cent

## 编辑user_list文件，允许cent用户访问FTP

    [root@localhost ~]# vi /etc/vsftpd/user_list
    # vsftpd userlist
    # If userlist_deny=NO, only allow users in this file
    # If userlist_deny=YES (default), never allow users in this file, and
    # do not even prompt for a password.
    # Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
    # for users that are denied.
    root
    bin
    daemon
    adm
    lp
    sync
    shutdown
    halt
    mail
    news
    uucp
    operator
    games
    nobody
    cent

## 建立我们的根目录，并设置访问权限

    [root@localhost ~]# mkdir /var/public_root
    [root@localhost ~]# chown -R cent /var/public_root
    [root@localhost ~]# chmod -R 755 /var/public_root

## 开启vsftpd服务

    [root@localhost ~]# service vsftpd start
    Starting vsftpd for vsftpd:              [  OK  ]

## 默认开启vsftp服务

    [root@localhost var]# chkconfig vsftpd on