启动

```
注意事项：用启动项目的用户登录arthus

1. 先查看项目的进程号
netstat -ntlp | grep 8083
2. 启动
java -jar arthas-boot.jar
3. 输入进程编号并回车
```

反编译、热加载

```
反编译
jad --source-only com.sinovatio.owls.business.ifs.target.service.IfsTargetInfoService > /tmp/IfsTargetInfoService.java
查看hash值
sc -d com.sinovatio.owls.business.ifs.target.service.IfsTargetInfoService | grep classLoaderHash
重新编译
mc -c 574caa3f /tmp/IfsTargetInfoService.java -d /tmp
重新加载
redefine /tmp/com/sinovatio/owls/business/ifs/target/service/IfsTargetInfoService.class
```

方法异常定位

```
// 耗时 + 执行异常
trace *.TargetManageService setToRedis
// 方法出参 入参
watch *.LinkAnalysisService queryBeforeToday '{params[0].toString(),returnObj.toString()}'
```

线程定位

```
arthus线程定位：thread all
jdk命令行工具查看jvm内存：
jstat -gc 2764 250 20   // 每250毫秒查看2764进程的垃圾收集情况，查看20次。
```