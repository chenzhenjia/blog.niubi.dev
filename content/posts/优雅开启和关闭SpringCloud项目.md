---
title: 优雅开启和关闭Spring Cloud项目
aliases: ["/you-ya-kai-qi-he-guan-bi-spring-cloudxiang-mu/"]
date: 2018-08-18T12:59:08+08:00
---

## 第一种方式,使用脚本(不推荐)
使用命令行开启和关闭，直接 kill 掉Spring boot 进程，这样会导致Spring Cloud 项目不会去 注册中心把自己销毁掉，明明项目已经关闭，但是注册中心还是显示在线，而且这样还会有一个弊端，如果需
要监听Spring Boot 项目的生命周期钩子，比如项目启动和关闭，也会监听不到。
`/home/dev/app.sh`

```bash
#!/bin/bash
APP_NAME=$2
#使用说明，用来提示输入参数
usage() {
    echo "Usage: sh 执行脚本.sh [start|stop|restart|status]"
    exit 1
}

#检查程序是否在运行
is_exist(){
  count=`ps -ef |grep java|grep $APP_NAME|grep -v grep|wc -l`
  #如果不存在返回1，存在返回0     
  if [ $count == 0 ]; then
   return 1
  else
    return 0
  fi
}

#启动方法
start(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is already running. pid=${pid} ."
  else
    SHELL_FOLDER=$(cd "$(dirname "$0")";pwd)
    JAR_PATH="$SHELL_FOLDER/../applications/$APP_NAME.jar"
    CONF_PATH="$SHELL_FOLDER/../conf/logback-spring.xml"
    nohup java -jar $JAR_PATH --logging.config=$CONF_PATH --spring.profiles.active=test > /dev/null 2>&1 &
   # java -jar $JAR_PATH --logging.config=$CONF_PATH
  fi
}

#停止方法
stop(){
  is_exist
  if [ $? -eq "0" ]; then
    pid=`ps -ef |grep java|grep $APP_NAME|grep -v grep|awk '{print $2}'`
    echo "Stop $APP_NAME, pid $pid"    
    kill -9 $pid
  else
    echo "${APP_NAME} is not running"
  fi  
}

#输出运行状态
status(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is running. Pid is ${pid}"
  else
    echo "${APP_NAME} is NOT running."
  fi
}

#重启
restart(){
  stop
  start
}

#根据输入参数，选择执行对应方法，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  *)
    usage
    ;;
esac
```
### 使用
```
* 开启
```bash
./app start myapp.jar
```
* 关闭
```bash
./app stop myapp.jar
```
* 重启
```bash
./app.sh restart myapp.jar 
```
## 第二种方式,打包可执行jar(推荐)
打包可执行的jar可以支持 systemctl 启动，也可以支持脚本启动，并且执行关闭后会冲注册中心销毁掉服务，并且会在启动和关闭会调用Spring Boot 的生命周期事件。
1. 编译工具配置
    * Maven 
    ```xml
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <executable>true</executable>
        </configuration>
    </plugin>
    ```
    * Gradle 
    ```groovy
    bootJar {
        launchScript()
    }
    ```
修改之后然后重新打包     

2. systemd 配置
    * 添加 systemd 的配置文件
    `/etc/systemd/system/my-app.service`
    _注意：多个服务请创建多个文件_
    ```text
    [Unit]
    Description=my-app.service
    After=syslog.target
    [Service]
    User=dev
    ExecStart=my-app.jar --logging.config=logback-spring.xml --spring.profiles.active=prod
    SuccessExitStatus=143
    TimeoutStopSec=10
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    ```
    * 启用 服务
    ```bash
    systemctl enable my-app.service
    ## 开启服务
    systemctl start my-app
    ## 关闭服务
    systemctl stop my-app
    ## 查看服务状态
    systemctl status my-app
    ## 重启服务状态
    systemctl restart my-app
    ```
3. 指定应用重新的内存等等
   在 my-app.jar 的同一个文件夹创建一个 my-app.conf 文件。
   _注意:配置文件必须和jar包的文件名一直才生效。_
   其它配置请看 [更多配置](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/deployment-install.html#deployment-script-customization-when-it-runs)
   ```text
   # 指定内存为256
   JAVA_OPTS="-Xmx256m -Xms256m"
   # 指定操作模式，通常是 auto，但是我们这里使用 service
   MODE=service
   # 指定 pid文件路径
   PID_FOLDER=./pid
   # 指定日志文件夹，如果使用了logback可以指定自己的日志文件夹路径，我用了 logback，所有这个日志文件夹没是没用，但是在启动logback 之前还是会输出一点日志，所有也需要指定一下，我是放在 /tmp 目录下
   LOG_FOLDER=/tmp
   # 指定spring boot 的参数
   RUN_ARGS="--logging.config=/home/conf/logback-spring.xml --spring.profiles.active=test"
   ```
 4. 不使用 systemd 而使用脚本启动和关闭.
    该方式同样支持冲注册中心销毁并且调用生命周期事件。
    ```shell
    #!/bin/bash
    APP_NAME=$2
    #使用说明，用来提示输入参数
    usage() {
        echo "Usage: sh 执行脚本.sh [start|stop|restart|status]"
        exit 1
    }
    #拼接可执行 jar 的路径
    jar_path(){
        SHELL_FOLDER=$(cd "$(dirname "$0")";pwd)
        JAR_PATH="$SHELL_FOLDER/../applications/$APP_NAME.jar"
        return $JAR_PATH
    }

    #启动方法
    start(){
      /bin/bash `jar_path` start
    }

    #停止方法
    stop(){
      /bin/bash `jar_path` stop
    }

    #输出运行状态
    status(){
      /bin/bash `jar_path` status
    }

    #重启
    restart(){
      /bin/bash `jar_path` restart 
    }

    #根据输入参数，选择执行对应方法，不输入则执行使用说明
    case "$1" in
      "start")
        start
        ;;
      "stop")
        stop
        ;;
      "status")
        status
        ;;
      "restart")
        restart
        ;;
      *)
        usage
        ;;
    esac
    ```
