# 软件介绍

一、软件介绍

1、TVBox是一个空壳软件，相当于一个本地播放器，软件并不能直接播放网络视频。

2、配置文件内有资源网站地址信息，tvbox+配置文件=网络视频播放器。

3、目前配置文件分为两种：网络和本地（建议直接用网络配置，在配置地址栏粘贴即可）

二、源码

源码地址：https://github.com/CatVodTVOfficial/TVBoxOSC

功能：支持直播及搜索；提供三种播放器；UI界面；安卓、电视盒子均可安装等。



# 自制接口

配置文件只有三个（先下载别人）

```
xxx.jar		爬虫+解析
xxx.json	配置软件：jar的路径、电视频道配置的路径、视频源的地址、搜索引擎、壁纸链接等等 
xxx.txt		电视频道的配置文件，里面就是各个频道的直播地址
```

1、创建仓库，并上传

2、复制xxx.txt的网络地址`https://xxx.../xxx.txt`，编译成base64编码

3、复制xxx.jar的网络地址`https://xxx.../xxx.jar`

4、修改xxx.json中的上面两个的地址

![img](https://i0.hdslb.com/bfs/article/0719c73383a23990a4624090fb0a3e9d33512914.png@942w_143h_progressive.webp)



