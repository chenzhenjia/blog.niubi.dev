---
title: MacOS 下使用命令 base64
permalink: macos-base64
id: 5d5e865ce9326100bc7569b5
updated: '2017-10-31T09:36:52.000Z'
date: 2017-10-31 17:36:52
---

### 编码
```shell
echo 'string' | base64
```

### 解码
```
echo 'c3RyaW5nCg==' | base64 -D 
```