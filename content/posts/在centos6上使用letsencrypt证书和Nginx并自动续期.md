---
title: 在 centos6上使用 letsencrypt 证书和 Nginx 并自动续期
aliases: [zai-centosshang-shi-yong-letsencrypt-zheng-shu-he-nginx-bing-zi-dong-xu-qi]
date: 2017-10-13T23:19:00+08:00
---

## 安装 Nginx 
1.执行命令

```bash
vim /etc/yum.repos.d/nginx.repo
```
2. 在英文状态下按 i 进入编辑模式，把一下代码贴进去，然后保存
```text
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

3.最后执行命令安装 Nginx

```bash
yum install nginx -y
```

4.查看 Nginx 是否安装

```bash
nginx -V
```
> 如果提示 command not found 则表示安装失败

5.配置你要申请证书域名到 Nginx
> 编辑 /etc/nginx/conf.d/blog.ai5suoai.com.conf
> **注意把路径后面的域名换为自己的域名**
> 注意 **server_name** 需要些自己的域名
```text
server {
  listen 80;
  server_name blog.ai5suoai.com;
  location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
6.启动 Nginx
```bash
/etc/init.d/nginx start
```
> 访问你的地址看是否显示 Nginx 的欢迎页面，如果显示则表示启动成功。到了这就可以开始安装letsencrypt工具了
7.附加选项，开机启动 Nginx
```bash
chkconfig nginx on
```


## 安装 letsencrypt 的工具
> 官网教程 [打开链接](https://certbot.eff.org/#centos6-nginx)
1. 下载 letsencrypt 执行文件

```bash
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
```
> 如果你想以后直接使用 certbot-auto 的话需要把 certbot-auto 移动到 `/usr/local/bin/` 下这样就可以自己执行 `certbot-auto` 命令了
### 申请证书
1. 开始申请证书
> 注意域名要写你自己的域名
```bash
sudo ./certbot-auto certonly --webroot -w /usr/share/nginx/html -d blog.ai5suoai.com
```
> 成功的话会在 `/etc/letsencrypt/live/` 路径下面后一个你的域名的文件夹。
2. 配置 Nginx 的 https
> 修改 /etc/nginx/conf.d/blog.ai5suoai.com.conf
> 把之前的配置删掉不要,注意路径为你自己写的配置文件路径
> 注意修改域名，还有证书路径
```text
# 强制 https
server {
  listen 80;
  server_name blog.ai5suoai.com;
  server {
  listen 80;
    server_name blog.ai5suoai.com;
    return 301 https://blog.ai5suoai.com;
  }
}

server {
    server_name blog.ai5suoai.com;
    # 成功的日志文件，注意路径
    access_log   /var/log/nginx/blog.ai5suoai.com.log  main;
    # 失败的日志文件，注意路径
    error_log   /var/log/nginx/blog.ai5suoai.com.error.log;
    listen 443 ssl http2;
      # 证书，注意路径
      ssl_certificate /etc/letsencrypt/live/blog.ai5suoai.com/fullchain.pem;
      # 私钥，注意路径
      ssl_certificate_key /etc/letsencrypt/live/blog.ai5suoai.com/privkey.pem;

      ssl_session_cache shared:le_nginx_SSL:1m;
      ssl_session_timeout 1440m;
      ssl_session_tickets off;

      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      ssl_prefer_server_ciphers on;

      ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS";

      ssl_stapling on;
      ssl_stapling_verify on;

      add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload";
      add_header X-Frame-Options DENY;
      add_header X-Content-Type-Options nosniff;
    # 这里根据实际情况  
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
3. 刷新 Nginx 配置

```bash
/etc/init.d/nginx reload
```
> 访问你的域名看是否出现 https。

### 自动续期证书
> 由于证书只有90天有效，所以需要续期

