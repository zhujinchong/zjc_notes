按照教程来

```
1. 安装VMWare
2. 安装Centos
3. 修改网络配置（NAT）
```



# 虚拟机环境

## 1. IP

修改网络 IP 地址为静态 IP 地址，避免 IP 地址经常变化。

```
[root@hadoop100 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33

BOOTPROTO="static" 	# 静态IP
IPADDR=192.168.10.100 #IP 地址
GATEWAY=192.168.10.2 #网关
DNS1=192.168.10.2	#域名解析器
```



## 2. hostname

修改主机

```
[root@hadoop100 ~]# vim /etc/hostname
hadoop100
```

修改主机映射文件

```
[root@hadoop100 ~]# vim /etc/hosts
192.168.10.100 hadoop100
192.168.10.101 hadoop101
192.168.10.102 hadoop102
192.168.10.103 hadoop103
192.168.10.104 hadoop104
192.168.10.105 hadoop105
192.168.10.106 hadoop106
192.168.10.107 hadoop107
192.168.10.108 hadoop108
```

重启

```
reboot
```



## 3. xsync同步脚本

编辑同步主机名

```
[root@hadoop100 ~]# vim ~/bin/xsync
```



## 4. ssh无密登录

xsync同步脚本还需要无密登录

```
# 进入ssh目录
[root@hadoop100 ~]# cd ~/.ssh
# 生成key
[root@hadoop100 .ssh]# ssh-keygen -t rsa
# 拷贝公钥到其他机器 （完成后在其他机器重复此操作）
[root@hadoop100 .ssh]# ssh-copy-id hadoop102
[root@hadoop100 .ssh]# ssh-copy-id hadoop103
```





