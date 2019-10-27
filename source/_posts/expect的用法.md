---
title: expect 的用法
permalink: expect-de-yong-fa
id: 5d5e865ce9326100bc7569c4
updated: '2018-08-17T03:52:44.000Z'
date: 2018-08-17 11:52:44
---

### !/usr/bin/expect
&nbsp;&nbsp;&nbsp;&nbsp;告诉操作系统脚本里的代码使用那一个`shell`来执行。

> 注意：这段代码必须在第一行。

### set timeout
设置超时时间，计时单位是：秒 ，`timeout -1` 为永不超时。
### spawn
`spawn`是进入`expect`环境后才可以执行的`expect`内部命令，如果没有装`expect`或者直接在默认的SHELL下执行是找不到`spawn`命令的。所以不要用 `which spawn` 之类的命令去找`spawn`命令。
> 它主要的功能是给ssh运行进程加个壳，用来传递交互指令。 

### expect "str"
这个命令的意思是判断上次输出结果里是否包含“str”的字符串，如果有则立即返回，否则就等待一段时间后返回，这里等待时长就是 `set timeout `
### send ""
这里就是执行交互动作，
> 命令字符串结尾别忘记加上“\r”，如果出现异常等待的状态可以核查一下。

### interact
执行完成后保持交互状态，把控制权交给控制台，这个时候就可以手工操作了。如果没有这一句登录完成后会退出，而不是留在远程终端上。如果你只是登录过去执行
### $argv 参数数组
expect脚本可以接受从bash传递过来的参数.可以使用[lindex $argv n]获得，n从0开始，分别表示第一个,第二个,第三个....参数
## 示例
### expect 调用 ssh 密码登陆

```bash
#!/usr/bin/expect

set timeout 5
set password "1234"
spawn ssh root@127.1.1
expect {
	"yes/no" { send "yes\r"; exp_continue }
	"Password:" { send "$password\r" }
	}
interact
```