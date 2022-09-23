---
title: 使用kubespray部署kubernetes集群
aliases: [use-kubespray-to-deploy-the-kubernetes-cluster]
date: 2017-11-28T10:43:40+08:00
---

## 准备工作
1. 因为是部署集群，所以需要两台机器，(我使用的是<a href="https://m.do.co/c/ddf9176cf4a7" target="_blank">digitalocean</a>这个云提供商)
1. 内存至少1.5GB 以上，不然会安装失败。
1. 需要每一台机器都能 ping 通

## 我的一些配置
1. 系统为 centos7
1. 节点的信息
    *这里使用digitalocean提供的内网地址*
<table>
    <thead>
        <tr>
        <td>ip 地址</td>
        <td>角色</td>
        <td>内存</td>
        <td>CPU</td>
        </tr>
    </thead>
    <tr>
    <td>10.132.37.154</td>
    <td>master</td>
    <td>3G</td>
    <td>1</td>
    </tr>
    <tr>
    <td>10.132.45.167</td>
    <td>node1</td>
    <td>3G</td>
    <td>1</td>
    </tr>
</table>

## 准备安装
1. 修改主节点的 hosts 文件，把节点ip 和名字写进去
    `vi /etc/hosts`
    
    ```
    10.132.37.154 master
    10.132.45.167 node1
    ```
    
1. 所有节点无密码登录
    1. 生成无密码的密钥对，用来进行主节点到副节点之间的无密码登录。
    直接全都回车，生成的密钥对在 ~/.ssh 目录下面，默认名称为 `id_rsa`和`id_rsa.pub`
        ```bash
        ssh-keygen
        ```
    1. 把公钥分别写到主节点和副节点的`~/.ssh/authorized_keys` 文件中。
    *注意：每个节点都有添加这个公钥，不管是本机还是其他机器* 
    ```bash
    # 先查看 id_rsa.pub 内容，复制出来，贴贴到 authorized_keys文件的最末尾
    cat ~/.ssh/id_rsa.pub
    # 贴到末尾
    vi ~/.ssh/authorized_keys
    ```

1. 在主节点安装pip
    1. centos 安装
        `easy_install pip`
    1. ubuntu 安装
        ```bash
        apt-get update -y
        apt-get install python python-pip
        ```
1. centos 需要安装 `git`
    ```bash
    yum -y install git
    ```
1. 使用 pip 安装 ansible 和 kubespray
    ```bash
    pip install ansible kubespray
    ```
1.  修改所有节点的 `/etc/resolv.conf`
    ```bash
    echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
    ```
1. 添加 kubespray 的配置文件
    ```bash
    cat <<EOF > ~/.kubespray.yml
    kubespray_git_repo: "https://github.com/kubernetes-incubator/kubespray.git"
    # Logging options
    loglevel: "info"
    EOF
    ```
1. 用 kubespray 生成节点的配置文件
    会在 `~/` 目录下生成一个 `.kubespray` 文件夹
    ```bash
    kubespray prepare --masters master --etcds master --nodes node1
    ```

1. 编辑 inventory.cfg 文件
    ```bash
    cd .kubespray
    vi inventory/inventory.cfg
    ```
    ```text
    master ansible_ssh_host=10.132.37.154 ip=10.132.37.154
    node1 ansible_ssh_host=10.132.45.167 ip=10.132.45.167

    [kube-master]
    master

    [all]
    node1
    master

    [k8s-cluster:children]
    kube-node
    kube-master

    [kube-node]
    node1

    [etcd]
    master
    ```
1. 在 ~/.kubespray 目录下执行部署 k8s 命令。
    *注意:* 在 <a href="https://m.do.co/c/ddf9176cf4a7" target="_blank">digitalocean</a> 的 centos7 上会遇到一个 `Check_certs | Set 'sync_certs' to true` 的错误。<a href="https://blog.ai5suoai.com/kubespray-k8s-error-sync_certs/" target="_blank">解决方法</a>
    显示了 `Kubernetes deployed successfuly` 则部署成功
    ```bash
    cd ~/.kubespray
    ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v --private-key=~/.ssh/id_rsa
    ```
1. 验证 k8s 集群
    ```bash
    kubectl get nodes
    ```
    ```text
    NAME      STATUS    ROLES     AGE       VERSION
    master    Ready     master    2m        v1.8.3+coreos.0
    node1     Ready     node      2m        v1.8.3+coreos.0
    ```

## 增加节点
1. 把新增的节点 ip 配置到 hosts 文件中
    `vi /etc/hosts`
    ```
    10.132.37.154 master
    10.132.45.167 node1
    10.132.45.87  node2
    ```
1. 在 `~/.kubespray` 目录执行命令
   `cd ~/.kubespray`
   `ansible-playbook -i inventory/inventory.cfg scale.yml -b -v --private-key=~/.ssh/id_rsa -l node2`