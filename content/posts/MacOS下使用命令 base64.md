---
title: "MacOS下使用命令 Base64"
date: 2017-10-31T17:36:52+08:00
aliases: ["/macos-base64/"]
---

### 编码
```bash
echo 'string' | base64
```

### 解码
```bash
echo 'c3RyaW5nCg==' | base64 -D 
