---
title: 在 centos 上搭建k3s 集群
aliases: [centos-k3s]
date: 2019-08-24T15:38:25+08:00
---

## 介绍 k3s 
k3s 是Rancher Labs（以下简称Rancher）宣布推出轻量级Kubernetes发行版K3s（已开源），这款产品专为在资源有限的环境中运行Kubernetes的研发和运维人员设计。Rancher此次发布的K3s项目，将满足在边缘计算环境中运行在x86、ARM64和ARMv7处理器上的小型、易于管理的Kubernetes集群日益增长的需求。
## 开始安装
1. 安装 docker
    [ 官方文档 ](https://docs.docker.com/install/linux/docker-ce/centos/)
    这里推荐使用官方的脚本一键安装

    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

    > 如果你想使用 k3s 默认的 contanerd 容器可以跳过，但是由于我对 contanerd 不熟悉所以我在这里使用的是 docker
1. 安装 k3s server

    安装 k3s server 有几种方式，由于 k3s 的执行文件是是放在 github上的，但是在中国由于网速不好可能会导致下载不下来，所以可以先使用其它工具下载可执行文件然后上传到服务器上的  `/usr/local/bin` 目录下，然后执行 `chmod 0755 /usr/local/bin/k3s ` 命令给与执行权限

    直接下载安装: 
    ```bash
    curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --no-deploy traefik --docker" sh -s -
    `````

    自己下载安装:
    ```bash
    curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --no-deploy traefik --docker" INSTALL_K3S_BIN_DIR="/usr/local/bin" INSTALL_K3S_SKIP_DOWNLOAD=true sh -s -
    ```

    参数说明:
    **INSTALL_K3S_EXEC** k3s 启动的参数  _--no-deploy traefik_ 是说不需要部署 traefik ingress，_--docker_是说使用 docker 容器
    **INSTALL_K3S_BIN _DIR**k3s 可执行文件的目录
    **INSTALL_K3S_SKIP _DOWNLOAD** true 跳过下载k3s 可执行文件

    查看 k3s server 是否安装成功
    ```bash
    systemctl status k3s
    ```
    如果k3s 的状态为运行的话说明成功

    获取节点列表验证
    ```bash
    kubectl get nodes
    ```

2.  安装 k3s-agent
3.  
    首先在 k3s 的 server 服务器上查看节点的 token
    ```bash
    cat /var/lib/rancher/k3s/server/node-token
    ```
    输出为 _xxxxxxx::node:xxxxxx_

    进入到 需要安装 agent 节点服务器执行
    ```bash
    curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent --docker"  K3S_URL=https://192.168.0.1:6443 K3S_TOKEN=xxxxxxx::node:xxxxxx sh -s -
    ```
    如果是自己下载并上传的话执行
    ```bash
    curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent --docker" INSTALL_K3S_BIN_DIR="/usr/local/bin" K3S_URL=https://192.168.0.1:6443 K3S_TOKEN=xxxxxxx::node:xxxxxx INSTALL_K3S_SKIP_DOWNLOAD=true sh -s -
    ```

    参数基本都是一样的,现在多加了一个 _K3S\_TOKEN_ 和 _K3S\_URL_ 参数

    **K3S\_TOKEN** 是主节点的 token
    
    **K3S\_URL** 是主节点的 ip 地址和端口一般 https 和 6443 是默认的

    验证安装:
    ```bash
    systemctl status k3s-agent
    ```
    如果 _Active: active (running)_ 说明启动成功
    
    进入k3s server 的服务器查看子节点是否添加成功
    ```bash
    kubectl get nodes
    ```

    > 如果列出了 agent 的节点说明安装成功,如果没有请检查 url 和 token 是否正确，并检查 agent 和 server 的网络是否能相互连接,还有就是 server 的防火墙是否打开了 6443端口

## 遇到的问题
当 agent 已经加入到了集群，但是发现部署在 server 里面的 pod 不能访问 agent 里面的 pod，发现 dns 是正常的，说明就是网络问题,进过搜索发现只要在 server 服务器和 agent 服务器执行 `sudo iptable -F` 就行