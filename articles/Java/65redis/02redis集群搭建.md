单机启动

```
# 修改配置
daemonize yes
# 后台启动
redis-server redis.conf
# 访问
redis-cli
# 退出
quit
# redis停止
redis-cli shutdown
```



集群启动

```
# 1. 解压
# 2. 编译和安装
cd redis-5.0.6/
make
make install
# 安装完以后/usr/local/bin/下面有redis的脚本
# 3. 到/usr/local下创建redis-cluster, 并创建7个
cd /usr/local/
mkdir redis-cluster
cd redis-cluster/
mkdir 7000 7001 7002
# 4. 拷贝并修改配置到各个目录下

# 配置========================
bind 10.45.151.213 #IP  
port 7000    #端口
daemonize yes   #是否后台启动
pidfile /var/run/redis_7000.pid #进程存放文件的地址（daemonize需要）
logfile "/usr/local/redis-cluster/7000/7000.log" #日志文件
save 900 1  #rdb存储
appendonly yes #aof存储
appendfilename "appendonly.aof"
cluster-enabled yes  #是否是集群方式启动
cluster-config-file nodes-7000.conf

# 5. 启动各个redis
redis-server 7000/redis.conf
redis-server 7001/redis.conf
redis-server 7002/redis.conf

# 如果以前有过集群，请把各个目录下的配置文件、备份数据删除
# 6. 以前使用ruby搭建集群，现在直接用redis-cli：
redis-cli --cluster create 10.45.151.213:7000 10.45.151.213:7001 10.45.151.213:7002 --cluster-replicas 1

# 8. 连接，测试
redis-cli -c -p 7000 -h 10.45.151.213
```