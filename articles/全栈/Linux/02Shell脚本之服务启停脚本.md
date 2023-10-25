启动脚本startup.sh

```
#!/bin/sh

PORT=7888
GPU=1
nohup python run_app.py --port=$PORT 2>&1 &
```

停止脚本shutdown.sh

```
#!/bin/sh

#设置关闭的端口
port=7888
#获取此端口运行的进程
pid=`netstat -ntlp | grep $port | awk '{print $NF}' | awk -F '/' '{print $1}'`
#判断如果进程号不为空则，关闭进程
if test -z "$pid";then
  echo "$port的进程未启动！"
else
  kill -9 $pid
  echo "$port的进程$pid 关闭成功！"
fi
```

其中`$NF`是awk命令自带的变量，表示文本列数，这里表示要取最后一列。

