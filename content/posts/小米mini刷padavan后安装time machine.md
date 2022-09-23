---
title: 小米 mini 刷 padavan 后安装 time machine
aliases: ["/xiao-mi-mini-shua-padavan-hou-an-zhuang-time-machine/"]
date: 2017-11-11T16:13:45+08:00
---

## 准备工作

1. 刷了 padavan 后的小米mini 路由器 
<a href="/xiao-mi-wifi-mini-shua-padavan/" target="_blank">小米 mini 刷 padavan 教程</a>
1. 一个移动硬盘或者 u 盘
1. 把 u 盘格式化为 ext4 文件格式
1. 挂载 u 盘到路由器
1. 安装 opkg 命令和其它设置
    1. <a href="192.168.123.1" target="_blank">进入 padavan 后台</a> 
      *默认用户名和密码是：admin*
    
    1. <a href="http://192.168.123.1/Advanced_System_Content.asp" target="_blank">点击系统管理</a> 修改管理员账号和密码为 `root`
    
      ![修改管理员账号的结果](/images/9A1BCE5B-5BE1-4E6A-866B-7DA2D7B247D8.png)
    
    1. <a href="http://192.168.123.1/Advanced_System_Content.asp" target="_blank">点击配置扩展环境</a> 启用 opt 自动更新 ---> 应用
    
      ![结果](/images/4CEB7E69-4C3C-4CC0-A765-B69D5E78D9DA.png)

## 开始安装
1. 使用 ssh 进入路由器，密码为你修改后的密码
    `ssh root@192.168.123.1`
1. 先执行 `opkg update`
1. 安装 mc, 执行 `opkg install mc`
1. 安装 dbus，执行 `opkg install dbus`
1. 安装和配置 avahi，执行`opkg install avahi-daemon avahi-utils` 
    `/opt/etc/avahi/avahi-daemon.conf` 这个文件是 avahi-daemon 的配置文件，不需要管。
    1. 创建一个 */opt/etc/avahi/services/afpd.service* 文件
    `touch /opt/etc/avahi/services/afpd.service`
    2. 编辑 afpd.service 文件,把一下内容贴进去
        ```xml
        <?xml version="1.0" standalone='no'?>
        <!DOCTYPE service-group SYSTEM "avahi-service.dtd">
        <service-group>
            <name replace-wildcards="yes">AFP on %h</name>
            <service>
                <type>_afpovertcp._tcp</type>
                <port>548</port>
            </service>
            <service>
                <type>_device-info._tcp</type>
                <port>0</port>
                <txt-record>model=TimeMachine</txt-record>
            </service>
        </service-group>
        ```
    1. 启动 avahi
    `/opt/etc/init.d/S20dbus restart`
    `/opt/etc/init.d/S42avahi-daemon restart`
    1. 查看 avahi 是否允许
    `ps | grep avahi`
        ```
         5463 nobody    3144 S    avahi-daemon: running [TimeMachine.local]
         5541 root      1592 S    grep avahi
        ```
1. 安装和配置 netatalk，`opkg install netatalk`
    1. 编辑配置文件
    `vi /opt/etc/afp.conf`
    1. 把下面的内容贴进去，并按照你的环境修改配置
        ```text
        [Global]
        afp listen = 192.168.123.1
        ; 你路由器的 hostname ，执行 hostname 命令查看
        hostname = hostname 
        ;log file = /opt/var/log/afpd.log
        ;log level = default:info afpdaemon:debug uamsdaemon:info
        uam list = uams_guest.so uams_dhx.so uams_dhx2.so
        uam path = /opt/lib/uams
        mimic model = TimeCapsule6,106
        hosts allow = 192.168.123.0/16
        guest account = bobody

        [TimeMachine]
        ; timemachine 的路径
        path = /media/TIMEMACHINE/
        time machine = yes
        cnid scheme = dbd
        ea = auto
        file perm = 0664 directory perm = 0775
        ```
    1. 重启 afpd
    `/opt/etc/init.d/S27afpd restart`

## 在 Mac 电脑上连接
1. 在 Finder 里面快捷键 `command + k`
1. 输入 `afp://192.168.123.1` 连接
1. 使用 访客连接

## 其它
1. 参考 
    1. [外国人写的安装 afp 教程](https://pztrn.name/blog/afp-zeroconf-timemachine-%D0%BD%D0%B0-%D0%BF%D1%80%D0%BE%D1%88%D0%B8%D0%B2%D0%BA%D0%B5-%D0%BE%D1%82-padavana/)
    我修改了他安装 avahi 的错误
    1. [外国人写的安装 afp 教程](https://w3bsit3-dns.com/forum/index.php?showtopic=605963&st=1700)
    他好多东西都不正确，但是安装 avahi和用户配置都是参考他的
1. 需要用户名和密码登录 afp
   1. 创建用户 `useradd timemachine`
   1. 修改用户名的密码 `passwd timemachine` 
    输入两次一样的密码
   1. 把用户添加到 afp 配置文件中 
        ```text
        [Global]
        afp listen = 192.168.123.1
        ; 你路由器的 hostname ，执行 hostname 命令查看
        hostname = hostname 
        ;log file = /opt/var/log/afpd.log
        ;log level = default:info afpdaemon:debug uamsdaemon:info
        uam list = uams_guest.so uams_dhx.so uams_dhx2.so
        uam path = /opt/lib/uams
        mimic model = TimeCapsule6,106
        hosts allow = 192.168.123.0/16
        ; 禁用客人登录
        ; guest account = bobody

        [TimeMachine]
        ; timemachine 的路径
        path = /media/TIMEMACHINE/
        ; 修改的地方
        valid users = timemachine
        time machine = yes
        cnid scheme = dbd
        ea = auto
        file perm = 0664 directory perm = 0775
       ```
1. 遇到的一些坑
    1. dbus 没启动成功
    执行 `dbus-daemon --system` 查看打印日志，一般报用户或者组没找到的问题。
        1. 修改配置，查看哪个报的是哪个用户或者组，把这个 xml 节点注释掉
            `/opt/etc/dbus-1/system.d/avahi-dbus.conf`
        1. 如果报了 root 用户没找到，请去看准备工作里面的修改管理员账号的步骤，并把管理员修改为 root。    
    1. avahi-daemon 没启动成功
    执行 `avahi-daemon --debug` 查看打印日志
        1. 报一下错误说明 dbus 没启动
            ```text
            WARNING: No NSS support for mDNS detected, consider installing nss-mdns!
            dbus_bus_get_private(): Failed to connect to socket /opt/var/run/dbus/system_bus_socket: No such file or directory
            WARNING: Failed to contact D-Bus daemon.
            avahi-daemon 0.6.32 exiting.
            ```
      
