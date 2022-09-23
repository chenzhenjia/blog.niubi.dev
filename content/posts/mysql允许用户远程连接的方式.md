---
title: mysql允许用户远程连接的方式
aliases: ["/mysqlyun-xu-yong-hu-yuan-cheng-lian-jie-de-fang-shi/"]
date: 2018-08-10T22:26:17+08:00
---

# mysql允许用户远程连接的方式。
* 第一步（修改配置文件）
    `vim /etc/mysql/my.cnf` 找到 `bind-address = 127.0.0.1`
    
    注释掉这行，如：`#bind-address = 127.0.0.1`
    
    或者改为： `bind-address = 0.0.0.0`
    
    允许任意IP访问；
    
    或者自己指定一个IP地址。
    
    重启 MySQL：`sudo /etc/init.d/mysql restart`

* 第二步（授权用户能进行远程连接）
        
    `grant all privileges on *.* to root@"%" identified by "password" with grant option;`
   `flush privileges;`
    > 第一行命令解释如下，*.* ：第一个*代表数据库名；第二个*代表表名。这里的意思是所有数据库里的所有表都授权给用户。root：授予root账号。 “%”：表示授权的用户IP可以指定，这里代表任意的IP地址都能访问MySQL数据库。“password”：分配账号对应的密码，这里密码自己替换成 你的mysql root帐号密码。
    
    > 第二行命令是刷新权限信息，也即是让我们所作的设置马上生效。
    

