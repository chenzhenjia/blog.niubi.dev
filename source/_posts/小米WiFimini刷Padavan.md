---
title: 小米 WiFi mini 刷Padavan
permalink: xiao-mi-wifi-mini-shua-padavan
id: 5d5e865ce9326100bc7569b6
updated: '2017-11-12T10:08:14.000Z'
date: 2017-11-09 10:16:22
typora-copy-images-to: ../images
typora-root-url: ../
---

## 准备工作
1. 准备一跟网线备用
1. 获取路由器的 ssh 权限
    1. 刷入**开发版** <a href="http://bbs.xiaomi.cn/t-11720354" target="_blank">刷机教程</a>
    1. 先使用你的小米账号绑定你的路由器
    1. 进入下面的地址登录小米账号获取 ssh 工具 <a href="http://d.miwifi.com/rom/ssh" target="_blank">获取地址</a>
    1. 把 ssh 刷入路由器
    1. 进入小米路由器
       `ssh root@192.168.31.1`

## PandoraBox 刷 Padavan

1. 先下载小米的开发版 ROM 
2. 把 ROM 刷入小米路由 <a href="http://bbs.xiaomi.cn/t-11720354" target="_blank">刷机教程</a>
3. 然后刷 Padavan 和小米 ROM 刷 Padavan 的步骤一样

## 小米ROM刷 Padavan
1. 下载 Breed 
    1. ssh 进入路由器
      `ssh root@192.168.31.1`
    
    1. 进入 tmp 文件夹
      `cd /tmp`
    
    1. 下载 Breed
      `wget https://breed.hackpascal.net/breed-mt7620-xiaomi-mini.bin`
    
    1. 刷入 Breed  
    
```bash
      chmod +x breed-mt7620-xiaomi-mini.bin
      mtd write breed-mt7620-xiaomi-mini.bin Bootloader
      ```
      
      
      **执行命令之后会出现 _Writing from breed-mt7620-xiaomi-mini.bin to Bootloader_ 然后重启路由器，重启路由器后小米路由器的灯会变蓝灯闪烁，这时候说明已经刷成功了但是却找不到 WiFi，这时候需要使用网线和电脑连接,连接后再进行下一个步骤**    
    
1. 刷入 Padavan
    1. 进入 Breed 的管理界面 <a href="http://192.168.1.1" target="_blank">在浏览器中打开</a>
    1. 下载 Padavan 固件 <a href="https://eyun.baidu.com/s/3kV0JV19/" target="_blank">百度云盘下载地址</a>
      
        **选择和你路由器相对应的固件，我下载了 *Padavan/2017-10-22/RT-AC54U-GPIO-30-xiaomimini-128M3.4.3.9-099.trx* 这个路径下面的固件**
        
    1. 在breed恢复界面，选择 **固件启动设置-->固件类型-->小米路由mini**
    1. 然后选择 **固件更新-->固件-->选择刚才下载的RT-AC54U-GPIO-30-xiaomimini-128M_3.4.3.9-099.trx**
      
        刷入固件重启之后,***依旧使用网线连接路由器***,然后 <a href="http://192.168.123.1" target="_blank">在浏览器中打开</a>,进入 Padavan 管理界面。这时候修改路由器的 ssid 名称和密码，然后使用 WiFi 连接上路由器。

       
    

