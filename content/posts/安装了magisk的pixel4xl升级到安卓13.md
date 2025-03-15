---
title: 安装了magisk的pixel4xl升级到安卓13
keywords:
- magisk 
- pixel4xl
- 升级安卓13出错
- oat 升级安卓13
- oat 升级安卓13出错
- upgrade android 13
- oat upgrade android 13
date: 2022-10-12T14:25:00+08:00
---
## 原因
我的pxiel4xl在线升级安卓13的时候报升级出错，后来找了一下原因，是因为系统更新（OTA）更新的时候，会检测验证system分区是否完整，如果被修改，则会导致更新失败，所以在下载的时候直接提示安装失败。刷Magisk时修改了boot.img这个system分区的文件，所以刷Magisk后不能直接安装OTA更新。

## 解决方法
### 准备工作
1. 先关闭开发者设置中的系统自动更新
    ![](/images/pixel-4xl-devlper-settings-xxxxxxx.png)
### 先卸载magisk
1. 打开Magisk
2. 点击卸载Magisk
3. 点击弹出窗口的还原原厂镜像
    还原远程镜像会可能会提示 `stock backup does not exist` `原厂镜像备份不存在`
### 解决原厂镜像不存在的问题
> 如果还原成功不需要进行该操作    
1. 在设置里面打开关于手机->版本号

    ![关于手机](/images/pixel-4xl-about-device.jpg)
    > 我在这里的版本号就是 221005.002 ，你需要注意你的安装版本号
2. 下载对应的镜像

   [下载网页](https://developers.google.com/android/images#coral)
   > 找到你手机对应的版本号下载
3. 解压boot.img
    
    在下载好的压缩包找到 image- 开头的压缩包
    
    再从这个image-开头的压缩包里面提取boot.img
4. 制作准备还原的boot.img
    1. 在电脑上push boot.img到手机上
    ```shell
    adb push boot.img /sdcard/boot.img
    ```
    2. 进入adb shell
    ```shell
    adb shell
    su
    ```
    3. 分别执行一下命令
    ```shell
    SHA1=$(cat $(magisk --path)/.magisk/config | grep SHA1 | cut -d '=' -f 2)
    gzip -9f /sdcard/boot.img
    mkdir /data/magisk_backup_${SHA1}
    mv /sdcard/boot.img.gz /data/magisk_backup_${SHA1}/boot.img.gz
    chmod -R 755 /data/magisk_backup_${SHA1}
    chown -R root.root /data/magisk_backup_${SHA1}
    ```
    上面的操作都成功了就可以了
### 升级系统
再次重试升级系统，如果这个时候还是报升级失败的话，只能重启系统后再升级
但是如果已经可以升级了就直接升级，升级完成后**千万别重启**，
### 升级系统后不重启安装magisk
1. 打开magisk -> 安装 -> 安装到未使用的槽位，然后重启
![](/images/magisk-install-xxxx.jpg)

### 卸载magisk还是不能升级系统操作
> 一下步骤其实就是重新刷入magisk
1. 重启magisk
2. 进入系统后确认一下magisk是否安装，如果未安装可以进行一下步骤
3. 直接点击升级系统（如果还是不行就不确定是什么问题了，有没有可能是boot.img 不正确之类的）
    等待升级完成，重启系统
4. 下载新系统对应的镜像
   1. 进入关于手机查看系统版本
   2. 还是在这个网页下载系统对应的镜像
   [下载网页](https://developers.google.com/android/images#coral)
5. 解压出boot.img
6. 上传boot.img到手机
   ```shell
   adb push boot.img /sdcard/Download/boot.img
   ```
7. 打开Magisk Manager安装程序
8. 制作 Magisk镜像
    安装 -> 选择 选择并修复一个文件-> 刚刚push到手机上的boot.img
    等待执行完成会告诉你文件名
9. pull 镜像到电脑上
    ```shell
    adb pull /sdcard/Download/magisk_pathced_xxx.img ./
    ```
10. 在电脑上刷入magisk
    ```shell
    fastboot flash boot /xx/xx/magisk_pathced_xxx.img
    ```
    ```shell
    fastboot reboot
    ```
### 执行 fastboot 一直在wait device解决
需要下载安装的usb 驱动并安装才可以
1. 下载驱动
[latest_usb_driver_windows 下载链接](https://dl-ssl.google.com/android/repository/latest_usb_driver_windows.zip)
2. 解压到文件
3. 打开设备管理
![](/images/2022-10-13_11-00-27.png)
4. 选择刚才解压的usb驱动的文件夹确认安装
> 再次执行 fastboot flash boot 就可以了