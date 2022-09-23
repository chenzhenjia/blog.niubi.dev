---
title: Ubuntu16.04.5 安装 Docker
aliases: [ubuntu16-04-5-an-zhuang-docker]
date: 2018-08-17T09:42:29+08:00
---

## 使用 apt-get 安装
### 卸载掉旧版本
命令:
```bash
sudo apt remove docker docker-engine docker.io
```

### 安装
1. 更新软件包
```bash
sudo apt update
```
1. 添加使用 HTTPS 传输的软件包以及 CA 证书
```bash
sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
2. 添加Docker的官方GPG密钥
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
验证秘钥
```bash
sudo apt-key fingerprint 0EBFCD88
```
查看是否能找到 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88。
1. 添加 docker 源
```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable" && sudo apt update
```
1. 安装
```bash
sudo apt install docker-ce
```

## 启动 docker
```bash
sudo systemctl enable docker
sudo systemctl start docker
```