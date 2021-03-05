---
title: 让我们用脚本启动jar吧
date: 2020-06-09 11:59:05
tags: 
 - 操作系统
 - Linux
categories: 
 - Linux
keywords: "Linux"
description: 让我们用脚本启动jar吧
---

### 说明
因为spring boot项目每次打包成jar都要手动写一串命令启动，不像正式服有自动发布系统。
```
nohup java -jar xxx.jar & 
```

### 编写脚本

所以让我们写个脚本来代替吧：
```shell
#!/bin/bash
#description: 启动重启server服务
#端口号，根据此端口号确定PID
PORT=7788
#启动命令所在目录
HOME='/home/ec2-user/server/online-consumer-backups'
jarName='consumer-online-backups.jar'

#查询出监听了PORT端口TCP协议的程序
#pid=`netstat -anp|grep $PORT|awk '{printf $7}'|cut -d/ -f1`
pid=`ps -ef | grep $HOME/$jarName | grep -v grep | awk '{print $2}'`

start(){
   if [ -n "$pid" ]; then
      echo "server already start,pid:$pid"
      return 0
   fi
   #进入命令所在目录
   cd $HOME
   nohup java -jar $HOME/$jarName > /dev/null 2>&1 &
   echo "start at port:$PORT"
}

stop(){
   if [ -z "$pid" ]; then
      echo "not find program on port:$PORT"
      return 0
   fi
   #结束程序，使用讯号2，如果不行可以尝试讯号9强制结束
   kill -9 $pid
   rm -rf $pid

#   fuser -k -n tcp $PORT
   echo "kill program succeed,PORT:$PORT,pid:$pid"
}

status(){
   if [ -z "$pid" ]; then
      echo "not find program on port:$PORT"
   else
      echo "program is running,port:$PORT,pid:$pid"
   fi
}

case $1 in
   start)
      start
   ;;
   stop)
      stop
   ;;
   restart)
      $0 stop
      sleep 2
      $0 start
    ;;
   status)
      status
   ;;
   *)
      echo "Usage: {start|stop|status}"
   ;;
esac

exit 0
```

说明：
脚本中的**HOME**表示jar包存放的路径，**jarName**表示jar包的名称。

### 使用
例如我们把上述的shell保存为 test.sh,我们需要先授权(具体权限可以根据每个人需求定制)：
```
chmod 777 test.sh
```

使用脚步：
```
启动项目：./test.sh start
停止项目：./test.sh stop
重启项目：./test.sh restart
查看项目运行状态：./test.sh status
```

好了，完美，终于不用手动折腾了！！！