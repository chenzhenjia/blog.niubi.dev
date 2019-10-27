---
title: 在 coreos 上部署 Convoy docker 插件
permalink: zai-coreos-shang-bu-shu-convoy-docker-cha-jian
id: 5d5e865ce9326100bc7569bc
updated: '2018-03-16T09:43:06.000Z'
date: 2018-03-16 17:43:06
---

## 前言
* docker swarm 需要共享数据
* coreos 相对其他 Linux 系统部署 nfs 的资料少

## 首先在 coreos 启用 nfs <a href="http://desert3.iteye.com/blog/1675522" target="_blank">参考</a>
1. 创建文件夹用来开启 nfs
    `mkdir /mnt/nfs`
1. 编辑 nfs 的 配置文件

    `vim /etc/exports`
    
    ```text
    /mnt/nfs  127.0.0.1(rw,sync,no_subtree_check,no_root_squash)
    ```
1. 在 coreos 启用 nfs 服务
    `systemctl enable nfs`

> nfs 服务只需要在一台服务器上启动就可以，其它服务器只需要挂载

## 安装 convoy <a href="https://www.jianshu.com/p/47ae8290ddd4" target="_blank">参考</a>
1. 下载安装
    
    ```bash
    wget https://github.com/rancher/convoy/releases/download/v0.5.0/convoy.tar.gz
    tar xvf convoy.tar.gz
    sudo cp convoy/convoy convoy/convoy-pdata_tools /opt/bin/
    ```
    
    
    
1. 挂载 nfs 目录
    `mount -t nfs 127.0.0.1:/mnt/nfs /nfs`
    
1. 安装 convoy 到 docker 的 plugin
    
    ```bash
    mkdir -vp /etc/docker/plugins/
    echo "unix:///var/run/convoy/convoy.sock" > /etc/docker/plugins/convoy.spec
    ```
    
    
    
3. 配置 convoy 的 systemd
    `vim /etc/systemd/system/convoy.service`
    
    ```text
    [Unit]
    Description=Convoy Daemon
    Requires=docker.service
    [Service]
    ExecStart=/opt/bin/convoy daemon --drivers vfs --driver-opts vfs.path=/nfs
    [Install]
    WantedBy=multi-user.target
    ```
    
1. 启动 convoy systemd
    
    ```bash
    systemctl daemon-reload
    systemctl enable convoy
    systemctl start convoy
    ```
    
    

> 如果是多台服务器每台就需要安装 convoy
# 遇到的问题
1.当修改 convoy 的路径的时候重启过后路径不生效，可以直接删除 `/var/lib/rancher/` 路径
    `rm -rf /var/lib/rancher/`

