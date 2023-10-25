# IDEA

## 常用插件

    mybatis log free   打印sql语句
    free mybatis   链接到mapper.xml
    alibaba java coding guidelines  提示编码规范
    vue     vue插件
    sonarlint   检查代码（没用过）
    lombok    构建javaBean
## 常用设置

编码格式设置：

```
setting -> file encoding -> 都选utf-8
```

# 常用快捷键

**搜索相关**

```
两次shift      搜索 all / class
ctrl + f   当前文件搜索
ctrl + r   替换
ctrl + shift + f  全局搜索
```

**类相关**

```
ctrl + h  显示类的继承关系
ctrl + f12  显示类的结构（方法、变量）
```

**代码编辑相关**

```
ctrl + shift + enter 跳至末尾加分号
shift + enter   换行
alt + enter   变量名补全 （修复当前代码 只能补全）

ctrl + p   方法参数提示
ctrl + shift + v 变量名补全
alt + insert  插入代码
ctrl + alt + t  代码块加if / try catch 等
ctrl + shift + u 大小写转换
ctrl + shift + l  代码格式化
ctrl + shift + o  整理导入包
```

**debug调试相关**

```
shift + f9  调试
shift + f10  运行
debug
 F7 进入方法（步入）
 F8  下一步（步出）
 F9 下一个断点（恢复程序）
```

# 常见问题

## MybatisLog不显示SQL

有时插件无法打印SQL，并不是插件坏了，有如下原因

**1. 项目的日志等级过高，修改日志等级为 DEBUG 或 INFO**

```
# log4j.properties文件
log4j.rootLogger = DEBUG,stdout,D

或者

# appliation-dev.yml 文件
logging.level.root: DEBUG
```

**2. mybatis配置中没有设置将sql日志输出到控制台**

```
# mybatis-config.xml
<configuration >
<settings>
<setting name="logImpl" value="org.apache.ibatis.logging.stdout.StdOutImpl" />
</settings>
...
</configuration>

或者

# application-dev.yml
mybatis-plus.onfiguration.log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
# 这里我成功了，但是不清楚为什么原项目是这样配置的（好像是将sql输出到log4j）
# mybatis-plus.onfiguration.log-impl: org.apache.ibatis.logging.log4j2.Log4j2Impl
```





