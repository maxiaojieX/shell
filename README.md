# 自定义常用Shell备份

## Nginx启动、停止、重启
```
#!/bin/bash
if [ $1 == "stop" ]
then
        PIDFILE="/www/applog/nginx.pid"
        PID=$(cat $PIDFILE)
        if [ -f $PIDFILE ]; then
                sudo kill -QUIT $PID
                echo "NGINX STOP SUCCESS"
        else
        echo "PID FILE NOT FOUND"
        fi
elif [ $1 == "start" ]
then
    sudo /usr/local/nginx/sbin/nginx
        echo "NGINX START SUCCESS"
elif [ $1 == "restart" ]
then
        PIDFILE="/www/applog/nginx.pid"
        PID=$(cat $PIDFILE)
        if [ -f $PIDFILE ]; then
                sudo kill -QUIT $PID
                sudo /usr/local/nginx/sbin/nginx
                echo "NGINX RESTART SUCCESS"
        else
        echo "PID FILE NOT FOUND"
        fi
else
        echo "I don't know what you want me to do"
fi
```
配合alias命令使用
```
alias nginxctl="/www/ctl/nginxctl.sh"
```

## jar文件启动
```
#!/bin/bash
string=$1;
string_pwd=${string%/*}
jar_name=${string##*/}
if [ ! -f ${string_pwd}"/${jar_name}_init.out" ];then
touch ${string_pwd}"/${jar_name}_init.out"
echo "创建初始化输出文件·······································OK"
fi
if [ -f ${string_pwd}"/${jar_name}_pid.pid" ];then
PID=$(cat ${string_pwd}"/${jar_name}_pid.pid")

sudo kill -9 $PID
echo "kill当前jar运行进程(${PID})········································OK"
fi
nohup java -jar ${string} >${string_pwd}"/${jar_name}_init.out" & echo $! > ${string_pwd}"/${jar_name}_pid.pid"
count=0
while [ ${count} -lt 1 ]; do
    echo -e ".\c"
    sleep 1
    count=`ps -ef | grep java | grep ${jar_name} | awk '{print $2}' | wc -l`
    if [ ${count} -gt 0 ]; then
        break
    fi
done
echo "运行jar文件>>>"${jar_name}"···············································OK"
```
配合alias命令使用
```
alias startjar="/www/ctl/jar.sh"
```

## jar文件启动优化shell
```
#!/bin/bash
string=$1;
string_pwd=${string%/*}
jar_name=${string##*/}
OLD_PIDS=0
OLD_PIDS=`ps -ef | grep java | grep "${jar_name}" | awk '{print $2}'`
echo -e "\033[33m 执行关闭当前项目旧进程 \033[0m"
if [ ${OLD_PIDS} -gt 0 ]; then
sudo kill -9 ${OLD_PIDS}
fi
num_kill=0
while true
do
    let num_kill++
    PI=`ps -ef | grep java | grep "${jar_name}" | awk '{print $2}'`
    if [ !${PI} ];then
        break
    fi
    echo -e ".\c"
    sleep 1
    if [ ${num_kill} -gt 30 ]; then
        echo -e "\033[31m 关闭当前项目旧进程失败,已退出shell,请检查并重新操作 \033[0m"
        exit 1
    fi
done
nohup java -jar ${string} >/dev/null 2>&1&
num=0
count=0
while [ ${count} -lt 1 ]; do
    let num++
    echo -e ".\c"
    sleep 1
    count=`ps -ef | grep java | grep ${jar_name} | awk '{print $2}' | wc -l`
    if [ ${count} -gt 0 ]; then
        break
    fi
    if [ ${num} -gt 30 ]; then
        break
    fi
done
if [ ${count} -gt 0 ];then
NOW_PIDS=`ps -ef | grep java | grep "${jar_name}" | awk '{print $2}'`
echo -e "\033[36m 运行:"${jar_name}"成功,端口:"${NOW_PIDS}" \033[0m"
else echo -e "\033[31m 启动失败 \033[0m"
fi

```

## jenkins自动部署脚本
```
#!/bin/bash -iel
if [ ! -d "/www/autoDeploy/tracking" ];then
mkdir /www/autoDeploy/tracking
else
rm -rf /www/autoDeploy/tracking
mkdir /www/autoDeploy/tracking
fi
svn --force --username  maxiaojie  --password xx000000  export svn://39.xxx.xx.38/tracking/trunk/tracking /www/autoDeploy/tracking
cd /www/autoDeploy/tracking
mvn clean install -Dmaven.test.skip=true
cd target
cp tracking-0.0.1-SNAPSHOT.jar /www/app/tracking/tracking-0.0.1-SNAPSHOT.jar
rm -rf /www/autoDeploy/tracking
cd /www/app/tracking
startjar /www/app/tracking/tracking-0.0.1-SNAPSHOT.jar
echo "Get it all done!"
```