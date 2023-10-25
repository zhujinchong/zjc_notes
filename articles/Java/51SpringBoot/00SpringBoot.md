# 1  快速入门

1、新建一个maven项目（有MySql数据库）

2、在pom.xml中导入依赖

```
	<dependencies>
        <!-- javaWeb mybatis druid mysql Junit5 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.1</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.28</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- freemarker模板 -->
        <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
        <!-- 热部署：Ctrl+F9 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

以及

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.0.RELEASE</version>
</parent>
```



3、在resources下面修改/新建配置文件application.properties

```
# 端口（默认）
server.port=8080

# mysql
spring.datasource.url=jdbc:mysql:///test?useUnicode=true&characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

# mybatis 这里的classpath就是resources目录
mybatis.mapper-locations=classpath:mapper/*.xml

# freemarker（默认）
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.template-loader-path=classpath:/templates/
spring.freemarker.suffix=.ftl
```

4、新建com/MainApp.java启动类

```
@SpringBootApplication
public class MainApp {
    public static void main(String[] args) {
        SpringApplication.run(MainApp.class, args);
    }
}
```

5、新建com/demo/DemoController.java业务类

```
@Controller
@RequestMapping("/demo")
public class DemoController {
    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        return "hello world";
    }
}
```

6、启动启动类，在浏览器访问`localhost:8080/demo/hello`

# 2  配置文件

## 配置文件位置

配置文件位置（按优先级排序）：

```
1. ./config/	项目根目录下的config目录下
2. ./			项目根目录下
3. classpath:/config/	即项目的resources目录下的config下
4. classpath:/			即项目的resources下
```



## properties赋值

这里有一个配置文件`person.properties`，现在要赋值给属性。

这样看起来有些麻烦

```
@Component
@PropertySource(value = "person.properties")
public class Person {
	// Value里面其实是SpEL,即Spring的表达式语言
	@Value("${name}")
    private String name;
    @Value("${age}")
    private Integer age;
    Getter/Setter
}
```



## yaml语法

基本语法：

* 大小写敏感
* 缩进表示层级（缩进只允许用tab，不许用空格）
* 缩进空格数不重要，只要层级对其即可
* #表示注释

支持的数据类型：

* 数值、字符串
* 数组
* 对象：键值对的集合

```
# 层级
key1:
	child-key1: value1
# 数组
myList: [value1, value2, value3]
# 复杂的对象
myObj:
	-
		id: 1
		name: xx
    -
    	id: 2
    	name: yy
# 布尔大写 小写 都可以
booleanKey: true
# 字符串单引号 双引号 没有引号 都可以
stringkey: 'hello world'
# 日期必须使用ISO 8601格式 'yyyy-MM-dd'
dateKey: 2018-02-17
# 时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
dateTime: 2018-02-17T15:02:31+08:00
```

## yaml赋值

配置文件有：

```
person:
  name: zhangsan
  age: 18
```

之前给属性赋值是通过注解`@Value`

现在可以：

```
// 只有组件在这个容器中，才可以使用ConfigurationProperties注解
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    Getter/Setter
}
```

然后验证：

```
@Autowired
private Person person;

@GetMapping("/person")
@ResponseBody
public Person person() {
    return person;
}
```

## yaml扩展

yaml支持复杂对象、SpEL、松散绑定、JSR303数据校验

SpEL：spring表达式语言，如`random`可以赋值随机数。

松散绑定:

```
如果配置里是first-name:xxx
属性名是firstName也可以识别
```

## JSR303数据校验

如果配置是person.email: xx@mail.com
此时，可以校验配置文件的数据的格式, 如：

```
@Component
@ConfigurationProperties(perefix = "person")
@Validated // 数据校验
public class Person {
	@Email
	private String email;
}
```

这里有许多注解及参数，不讲。

## 多环境配置

一般需要开发环境、测试环境，现有三个配置文件

```
appllication.properties
application-dev.properties
application-test.properties
```

我们可以在application.properties中配置使用哪个配置

```
# 只需要配置文件-后面的字符即可激活
spring.profiles.active=dev
```

如果是yaml文件，各个环境配置可以写在一个文件里面，用-分割

```
server:
	port: 8081
spring:
	profiles:
		active: dev

---
spring:
	profiles: dev
server:
	port: 8082
    
---
spring:
	profiles: test
server:
	port: 8083
```

# 3  自动配置原理

```
1. SpringBoot启动会加载大量的自动配置类
2. 自动配置类，会自动配置用到的组件
3. 自动配置类，会从properties类中获取某些属性（默认属性、或读配置文件等）

所以，我们一般先引入依赖（自动配置类生效），在配置文件中添加配置，或者自定义某个配置类。
```

自动配置类一般为：`xxxAutoConfiguration`

其获取的属性一般在：`xxxProperties`中，所以可以根据此类，在配置文件修改配置。



# 4  web开发

## 静态资源位置

web的自动配置类是`WebMvcAutoConfiguration`

该类的方法`addResourceHandlers`，该方法在加载静态资源顺序：（按优先级）

```
/webjars/**						(这里需要通过localhost:8080/webjars/访问)
classpath:/META-INF/resources/  (这里的classpath就是项目的resouces目录)
classpath:/resources/
classpath:/static/
classpath:/public/
```

当然，还可以在配置文件中添加静态资源位置（一般不同）：

```
spring.mvc.static-path-pattern=classpath:/static/
```

## 首页和图标

首页：在上述静态资源位置放入index.html即可，或者通过Controller返回。

图标（网页Title处）：首先关闭默认图标，然后将`favicon.ico`和index放一块即可。

```
spring.mvc.favicon.enabled=false
```

## 模板引擎

jsp是一个模板引擎，但是不方便。此外还有freemarker和Thymeleaf，想使用模板，先引入依赖。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

首先在`classpath:/templates/`目录下创建一个`index.ftl`

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
<h1>${name}</h1>
</body>
</html>
```

然后创建接口

```
@GetMapping("/index")
public String index(Model model) {
    model.addAttribute("name", "zhangsan");
    return "index";
}
```

就可以用了

# 5  拦截器

拦截所有页面、不包括静态资源、实现使用登录拦截。

1、定义登录，并设置session

```
@PostMapping("/doLogin")
public String userLogin(String username, String password, Model model, HttpSession session) {
    User user = userService.getUserByUserName(username);
    if (user != null && user.getPassWord().equals(password)) {
        session.setAttribute("loginUser", user);
        return "/index";
    } else {
        model.addAttribute("msg", "用户名或密码错误");
        return "/login";
    }
}
```



2、定义拦截器，并判断session。

```
public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        User loginUser = (User) request.getSession().getAttribute("loginUser");
        if (loginUser == null) {
            request.setAttribute("unLogin", true);
            request.getRequestDispatcher("/login").forward(request, response);
            return false;
        } else {
            return true;
        }
    }
}
```



3、定义WebMvcConfig，并将拦截器注册到Mvc中

```
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginHandlerInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/index", "/login", "/register", "/doLogin", "/bootstrap/**", "/img/**", "/common/**");
    }
}
```



4、前端页面判断freemarker

```
<#if (unLogin??)>
	<a href="xx">
<#else>
	<a href="yy">
<#/if>
```



# 6  整合mybatis

整合数据库、连接池、mybatis，使用`快速入门`的依赖。

1、重写dao，改成xxMapper接口

```
@Mapper
@Repository
public interface UserMapper {
    Integer addUser(User user);
    Integer updateUser(User user);
    Integer deleteUserById(@Param("id") Integer id);
    User getUserByUserName(@Param("username") String userName);
    List<User> getUserList();
}
```

2、在配置文件中添加mapper.xml，地址一般写为：

```
mybatis.mapper-locations=classpath:mapper/*.xml
```

3、在mapper中新建xx.xml文件，写sql语句

```\
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yly.manage.mapper.CategoryMapper">

    <select id="getCategoryList" resultType="com.yly.manage.pojo.Category">
        select * from category
    </select>

    <insert id="addCategory" parameterType="com.yly.manage.pojo.Category" useGeneratedKeys="true" keyProperty="id">
        insert into category (name,ref_id)
        values (#{name},#{ref_id})
    </insert>

    <delete id="deleteCategoryById">
        delete from category where id = #{id};
    </delete>
</mapper>
```



# 7  测试

引入测试依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

测试文件

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = MainApp.class)
public class DemoTest {

    @Autowired
    private TestService testService;

    @Test
    public void test01() {
        //
    }
}
```



