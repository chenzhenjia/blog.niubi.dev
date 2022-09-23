---
title: "Golang在Windows上同时编译64位和32位可执行程序"
date: 2018-11-02T18:38:39+08:00
aliases: [golang-zai-windows-shang-tong-shi-bian-yi-64wei-he-32wei-ke-zhi-xing-cheng-xu]
---

##  Golang 在 Windows 上同时编译64位和32位可执行程序
_需要在windows上同时编译需要64位的系统，32位只能编译32位的可执行程序。_
* 编译32位
    ```bash
    GOOS=windows GOARCH=386 go build main.go
    ```
* 编译64位
    ```bash
    GOOS=windows GOARCH=amd64 go build main.go
    ```
### 遇到的问题
由于在项目中用到了**sqlite**所以在编译的时候报 **exec: "gcc": executable file not found in %PATH%**，这是因为在windows上缺少 _gcc_,所以在windows 上需要安装 _MinGW_。
* 下载 MinGW
    [进入下载页面](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/%20)
    * 下载 MinGW64，注意请选择 \_x86\_64-posix-seh
        [MinGW64 8.1.0 下载地址](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-posix/seh/x86_64-8.1.0-release-posix-seh-rt_v6-rev0.7z)

    * 下载 MinGW32
        [MinGW32 8.1.0 下载地址](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-win32/seh/x86_64-8.1.0-release-win32-seh-rt_v6-rev0.7z)

* 解压 MinGW32和MinGW64
    _注意需要解压到不同地方，可以平级但是千万不要在同一个目录_
* 编译
    在编译之前先设置 MinGW 的环境变量，因为是一次性的所以直接用命令行来设置
    * 64位编译
        请注意_C:MinGW64bin_ 应该改为你的64位MinGW路径
        ```bash
        set PATH=C:\MinGW64\bin;%PATH%
        GOOS=windows GOARCH=386 go build main.go
        * 32位编译
        请注意_C:MinGW32bin_ 应该改为你的32位MinGW路径
        ```bash
        set PATH=C:\MinGW32\bin;%PATH%
        GOOS=windows GOARCH=amd64 go build main.go
		
# 大公告成