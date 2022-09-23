---
title: 在Linux 上安装oracle 的 sqlpuls 教程
aliases: [zai-fu-wu-qi-shang-an-zhuang-oracle-de-sqlpuls-jiao-cheng]
date: 2017-10-19T11:02:36+08:00
---

由于公司业务需要用到 sqlplus 和 exp 还有 imp，用来执行 sql 和导入导出 dmp 文件，然而只有安装了 oracle 服务端才有 sqlplus，但是 oracle 提供了另外一种方式可以自己安装 sqlplus
## 安装 sqlplus 
1. 需要三个文件
        
    [basic-11.1.0.70-linux-x86_64.zip](https://github.com/chenzhenjia/blog-comment/raw/master/sqlplus/basic-11.1.0.70-linux-x86_64.zip)

    [sdk-11.1.0.7.0-linux-x86_64](https://github.com/chenzhenjia/blog-comment/raw/master/sqlplus/sdk-11.1.0.7.0-linux-x86_64.zip)

    [sqlplus-11.1.0.7.0-linux-x86_64.zip](https://github.com/chenzhenjia/blog-comment/raw/master/sqlplus/sqlplus-11.1.0.7.0-linux-x86_64.zip)

 2. 解压下载好的文件到 `/opt/oracle/` 目录下
 
    ```bash
    unzip basic-11.1.0.70-linux-x86_64.zip -d /opt/oracle/
    unzip sdk-11.1.0.7.0-linux-x86_64.zip -d /opt/oracle/
    unzip sqlplus-11.1.0.7.0-linux-x86_64.zip  -d /opt/oracle/
    ```
    
3. 配置环境变量
    在用户目录编辑 .bashrc
    ```bash
    vim ~/.bashrc
    ```

    ```bash
    # 配置 oracle 的 sqlplus
    export ORACLE_HOME=/opt/oracle/instantclient_11_1/
    export TNS_ADMIN=$ORACLE_HOME
    export PATH=$PATH:$ORACLE_HOME
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
    export NLS_LANG='AMERICAN_AMERICA.ZHS16GBK'
    ```

4. 刷新 bashrc

    ```bash
    source ~/.bashrc
    ```
 
5. 链接数据库

    **示例:**`sqlplus user/password@host/sid`
    ```bash
    sqlplus device/device@144.131.252.79/qmfgjb
    ```
6. sqlplus 安装完成

## 安装 exp 和 imp
需要从你的 oracle 服务器上复制 exp 和 imp 还有一个必须项 `rdbms/` 文件夹

1. 登录你的所在 oracle 的 Linux，然后进入到`/oracle/product/11.2.2/` 目录，**注意你自己的 oracle 的路径**
2. 把 `rdbms/` 整个目录复制到你本地。
3. 进入到 `/oracle/product/11.2.2/bin` 目录，把 `expdp`,`exp`  和 `impdp`,`imp` 复制到你本地。
4. 把刚才下载到本地的几个文件夹全部上传到`/opt/oracle/instantclient_11_1/` 目录。
![最终结果](https://github.com/chenzhenjia/blog-comment/blob/a10cbb83b05f4f934732cc9651f2c6e1679a3b86/sqlplus/jietu.png?raw=true)
5. 给 exp 命令和 imp 命令执行权限
```bash
chmod +x instantclient_11_1/exp
chmod +x instantclient_11_1/imp
```

6.安装 exp 和 imp 完成