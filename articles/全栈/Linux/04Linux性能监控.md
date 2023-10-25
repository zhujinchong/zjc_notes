# 内存

```
[zjc@localhost ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:         643466       25024      520640          40       97801      616556
Swap:          4095           0        4095
[zjc@localhost ~]$ free -h
              total        used        free      shared  buff/cache   available
Mem:           628G         24G        508G         40M         95G        602G
Swap:          4.0G          0B        4.0G
```



其中， `-m` 选项是以MB为单位来展示内存使用信息; `-h` 选项则是以人类(human)可读的单位来展示。



# GPU

`nvitop`是一款交互式NVIDIA-GPU设备性能&资源&进程的实时监测工具。

conda安装

```
pip install --upgrade nvitop
```

使用时有三种模式

```text
# Automatically configure the display mode according to the terminal size
$ nvitop -m auto     # shortcut: `a` key

# Arbitrarily display as `full` mode
$ nvitop -m full     # shortcut: `f` key

# Arbitrarily display as `compact` mode
$ nvitop -m compact  # shortcut: `c` key
```

