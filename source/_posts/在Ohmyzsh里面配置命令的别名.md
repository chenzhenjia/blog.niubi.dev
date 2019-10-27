---
title: 在 Oh-my-zsh 里面配置命令的别名
permalink: zai-oh-my-zsh-li-mian-pei-zhi-ming-ling-de-bie-ming
id: 5d5e865ce9326100bc7569b0
updated: '2017-10-11T09:24:11.000Z'
date: 2017-10-11 17:24:11
---

### 配置

Zsh的配置文件位于～/.zshrc
从文件末尾开始添加配置。

#### 别名：

```
alias ll='ls -al'
alias la='ls -a'
alias gita='git add'
alias gitc='git commit'
```

#### 扩展名别名：

```
alias -s gz='tar -xzvf'
alias -s bz2='tar -xjvf'
```