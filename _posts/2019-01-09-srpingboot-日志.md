---
title: Springboot 日志
date: 2019-01-09 23:29:53
categories:
- Springboot
tags:
- 日志
- Springboot
---

### 启动方式1：

java -jar app.jar 前台启动

### 启动方式2：

nohup java -jar app.jar & 后台启动
 区别:前台启动ctrl+c就会关闭程序,后台启动ctrl+c不会关闭程序

制定控制台的标准输出
 java -jar app.jar > catalina.out  2>&1 &

- catalina.out将标准输出指向制定文件catalina.out
- 2>&1 输出所有的日志文件
- & 后台启动

对于上面的命令的解释：
 bash 中 0、1、2 三个数字分别代表 STDIN_FILENO 、 STDOUT_FILENO 、STDERR_FILENO ，即标准输入（一般是键盘），标准输出（一般是显示屏，准确的说是用户终端控制台），标准错误（出错信息输出）。

数字  含义
 0   标准输入（一般是键盘）
 1   标准输出（一般是显示屏，准确的说是用户终端控制台）
 2   标准错误（出错信息输出）

### 启动方式3：

编写shell脚本启动

在app.jar 同目录下编辑app.sh脚本文件
 内容如下：

```sh
#!/bin/sh
#功能简介：启动app.jar 文件
#注意：在sh文件中=赋值，左右两侧不能有空格


APP=app
APP_NAME=${APP}".jar"

log_dir=/home/jar_logs/
log_file=/home/jar_logs/app.log

command=$1

# 启动
function start(){
    # 日志文件夹不存在，则创建
    if [ ! -d "${log_dir}" ];then
        mkdir "${log_dir}"
    fi

    rm -f tpid
    nohup java -jar ${APP_NAME} 1>/dev/null 2>"${log_file}" &
    echo $! > tpid
    check
}

# 停止
function stop(){
    tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
    if [ ${tpid} ]; then
        echo 'stop process...'
        kill -15 $tpid
    fi

    sleep 5

    tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
    if [ ${tpid} ]; then
        echo 'Kill Process!'
        kill -9 $tpid
    else
        echo 'Stop Success!'
    fi
}

# 检查
function check(){
    tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
    if [ ${tpid} ]; then
        echo 'App is running.'
    else
        echo 'App is NOT running.'
    fi

}

# 强制kill进程
function forcekill(){
    tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

    if [ ${tpid} ]; then
        echo 'Kill Process!'
        kill -9 $tpid

    fi

}

# 输出进程号
function showtpid(){
    tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
    if [ ${tpid} ]; then
        echo 'process '$APP_NAME' tpid is '$tpid
    else
        echo 'process '$APP_NAME' is not running.'
    fi
}

if [ "${command}" ==  "start" ]; then
    start

elif [ "${command}" ==  "stop" ]; then
     stop

elif [ "${command}" ==  "check" ]; then
     check

elif [ "${command}" ==  "kill" ]; then
     forcekill

elif [ "${command}" == "tpid" ];then
     showtpid

else
    echo "Unknow argument...."
fi
```

编写完成后需要将脚本文件设置超级管理员权限
 chmod +x app.sh

之后使用./app.sh start/stop等命令启动即可
