如果我们Spring Boot服务直接部署在Windows或Linux环境下，如果我们启动或关闭服务，需要频繁手敲命令，很不方便。

此时，我们可以编写启动脚本来对服务进行控制

# Windows

## 启动脚本

**startup.bat**

~~~bat
@echo off

title Spring Boot Demo
java -jar spring-boot-demo.jar --server.config=application.yml

@pause
~~~

## 关闭脚本

**shutdown.bat**

~~~bat
@echo off

set port=8090
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
    echo kill the process %%m who use the port %port%
    taskkill /pid %%m -t -f
)
~~~

## 重启脚本

**restart.bat**

~~~bat
@echo off

call ./shutdown.bat
call ./startup.bat

@pause
~~~
或
~~~bat
@echo off

set port=8090
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
    echo kill the process %%m who use the port 
    taskkill /pid %%m -t -f
    goto start
)

cd %~dp0
start java -jar spring-boot-demo.jar --server.config=application.yml
exit
:start
start java -jar spring-boot-demo.jar --server.config=application.yml
exit

@pause
~~~

# Linux

## 启动/重启脚本

**startup.sh**

~~~sh
startTime=`date +'%Y-%m-%d %H:%M:%S'`

#jar包文件路径
APP_PATH=/home/demo

#jar包文件名称
APP_NAME=$APP_PATH/spring-boot-demo.jar

#日志文件名称
LOG_FILE=$APP_PATH/spring-boot-demo_out.log

rm -rf $LOG_FILE

echo "开始停止服务"

#查询进程，并杀掉当前jar/java程序
pid=`ps -ef|grep $APP_NAME | grep -v grep | awk '{print $2}'`
if [ $pid ];then
  echo "pid: $pid"
  kill -9 $pid
  echo "服务停止成功"
fi

sleep 2

#判断jar包文件是否存在，如果存在启动jar包，并实时查看启动日志
if test -e $APP_NAME;then
  echo '文件存在，开始启动服务'
  
  #启动jar包，指向日志文件，2>&1 & 表示打开或指向同一个日志文件
  nohup java -jar -Duser.timezone=GMT+08 $APP_NAME --server.config=application.yml > spring-boot-demo_out.log 2>&1 &
  echo "服务启动中"
  sleep 10s
  
  #通过检测日志来判断
  while [ -f $LOG_FILE ]
  do
      success=`grep "Started SpringBootDemoApplication in " $LOG_FILE`
      if [[ "$success" != "" ]]
      then
          break
      else
          sleep 1s
      fi
	  
      #开始检测启动失败标记
      fail=`grep "Fail" $LOG_FILE`
      if [[ "$fail" != "" ]]
      then
          echo "服务启动失败"
          tail -f $LOG_FILE
          break
      else
          sleep 1s
      fi
  done
  echo "服务启动成功"
  
  endTime=`date +'%Y-%m-%d %H:%M:%S'`
  startSecond=$(date --date="$startTime" +%s);
  endSecond=$(date --date="$endTime" +%s);
  
  total=$((endSecond-startSecond))
  echo "运行时间："$total"s"
  echo "当前时间："$endTime
else
  echo $APP_NAME ' 文件不存在'
fi
~~~

## 关闭脚本

**shutdown.sh**

~~~sh
#jar包文件名称
APP_NAME=/data/demo/spring-boot-demo.jar

echo "开始停止服务"

#查询进程，并杀掉当前jar/java程序
pid=`ps -ef|grep $APP_NAME | grep -v grep | awk '{print $2}'`

echo "pid: $pid "

if [ $pid ];then
  echo "pid: $pid"
  kill -9 $pid
  echo "服务停止成功"
else
  echo "未找到对应服务"
fi
~~~



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/139223258](https://blog.csdn.net/dkbnull/article/details/139223258)

微信：[https://mp.weixin.qq.com/s/I3BxfEiwaAyi1pD9SCGC4Q](https://mp.weixin.qq.com/s/I3BxfEiwaAyi1pD9SCGC4Q)

知乎：[https://zhuanlan.zhihu.com/p/699970189](https://zhuanlan.zhihu.com/p/699970189)

---

