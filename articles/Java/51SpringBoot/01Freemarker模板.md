# FreeMarker

## 简介

FreeMarker是一个用Java语言编写的模板引擎，它基于模板来生成文本输出。FreeMarker与Web容器无关，即在Web运行时，它并不知道Servlet或HTTP。它不仅可以用作表现层的实现技术，而且还可以用于生成XML，JSP或Java 等。

## 快速入门

FreeMarker是一个模板引擎，要想使用，先导入Maven依赖：

```
<dependency>
<groupId>freemarker</groupId>
<artifactId>freemarker</artifactId>
<version>2.3.9</version>
</dependency>
```

如果是SpringBoot，导入如下：

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

配置文件如下：

```
# 都是默认的不配也行
# 模板存放路径
freemarker.template-loader-path=classpath:/templates
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.check-template-location=true
spring.freemarker.content-type=text/html
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.request-context-attribute=request
spring.freemarker.suffix=.ftl
```

首先在resources/templates下面编写hello.ftl页面

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
    <#list users as user>
        user <br />
        <#if user=="Tom">
            hello: Tom! <br />
        </#if>
    </#list>
</body>
</html>
```

然后写一个controller（两种写法）

```
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String hello(Model model) {
        String[] users = {"zhang", "Tom", "wang"};
        model.addAttribute("users", users);
        return "hello";
    }
    @RequestMapping("/hello2")
    public ModelAndView hello2(ModelAndView mv) {
        String[] users = {"zhang", "Tom", "wang"};
        mv.addObject("users", users);
        mv.setViewName("hello");
        return mv;
    }
}
```

这样就可以访问了。

# 语法

## 常用语法

FreeMarker里面的标签也叫指令（个人理解，看的资料不全）

FreeMarker标签前面都有一个#号；

FreeMarker注释也类似：`<#--  注释内容  -->`

变量取值：`${变量名}`

if指令： （这里变量名写不写大括号都可以）

```
<#if 变量名=="xxx">
 hello: Tom
<#elseif 变量名=="yyy">
 hello: Cat
<#else>
</#if>
```

list循环指令：

```
<#list vars as var> 
 ${var}...
</#list>
```

assign指令：定义一个变量并给其赋值

```
<#assign username="Tom"> 
```

include指令：模板嵌套

```
<#include "/footer.html">
```



## 数据类型

布尔类型，布尔类型不能直接输出，需要转换成字符串，有以下三种方式

```
<h1>${flag?c}</h1>
<h1>${flag?string}</h1>
<h1>${flag?string("对的","错的")}</h1>
```

日期，也需要转成字符串（注意：controller需要传送一个Date对象）

```
${time?date} 年月日
${time?time} 时分秒
${time?datetime} 年月日时分秒
${time?string("yyyy-MM-dd HH:mm:ss")}
```

数值型，也可以定义格式

```
${salary}
${salary?string["0.##"]} <#-- 保留两位小数 -->
```

字符串

```
${msg}
${msg?substring(1,4)}  <#--截取字符串-->
${msg?trim?length} <#--去除空字符，并计算长度-->
${names?join(",")}  <#--将数组连接成字符串-->
```

Freemarker必须处理null值，两种方式：

```
<#-- 如果为空，则输出默认字符串 -->
${msg!"该值为空"}

<#-- ??判断变量是否存在，如果存在true，否则false -->
${(msg??)?string}
```

map类型

```
<#list map?keys as key>
 ${key}: ${map[key]}
<#list>
```

## 常见指令

list指令：循环

assign指令：创建变量，或修改变量值

```
<#assign msg="hello Tom">
<#assign num=2 names=["tom", "cat"]>
```



macro指令：用来自定义指令。

```
<#-- 自定义指令 -->
<#macro address>
 江苏省南京市
</#macro>
<#-- 使用指令 -->
<@address></@address>

<#-- 示例2：有参数 -->
<#macro findByIdAndName id name>
 ${name},你的id是：${id}
</#macro>
<@findByIdAndName id="12" name="tom"></@findByIdAndName>
```



import导入指令：可以引入一个库，然后使用引入的库中的指令。库里面的指令一般是自定义的

```
<#import "库名.ftl" as 库名>
<@库名.指令> </@库名.指令>
```



include包含指令

```
<#include "test.ftl">
<#include "test.html">
<#include "test.txt">
```

