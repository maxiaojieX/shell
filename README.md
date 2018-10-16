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
#nohup java -jar ${string} >${string_pwd}"/init.out" & >${string_pwd}"/FILEPID.pid"
nohup java -jar ${string} >${string_pwd}"/${jar_name}_init.out" & echo $! > ${string_pwd}"/${jar_name}_pid.pid"
echo "运行jar文件>>>"${jar_name}"···············································OK"
```
配合alias命令使用
```
alias startjar="/www/ctl/jar.sh"
```
