---
title: 在 Oh-my-zsh 里面配置命令的别名
aliases: ["/zai-oh-my-zsh-li-mian-pei-zhi-ming-ling-de-bie-ming/"]
date: 2017-10-11T17:24:11+08:00
---

## 配置

Zsh的配置文件位于～/.zshrc
从文件末尾开始添加配置。

### 别名：

```text
alias ll='ls -al'
alias la='ls -a'
alias gita='git add'
alias gitc='git commit'
```

### 扩展名别名：

```text
alias -s gz='tar -xzvf'
alias -s bz2='tar -xjvf'
```