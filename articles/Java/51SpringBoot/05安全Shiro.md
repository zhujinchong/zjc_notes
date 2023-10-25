# Shiro

## 简介

Apache Shiro 是一个强大灵活的开源安全框架，可以完全处理身份验证、授权、加密和会话管理。

Shiro功能包括：

* Authentication（认证）：用户身份识别，通常被称为用户“登录”

* Authorization（授权）：访问控制。比如某个用户是否具有某个操作的使用权限。

* Session Management（会话管理）：特定于用户的会话管理，甚至在非web 或 EJB 应用程序。

* Cryptography（加密）：在对数据源使用加密算法加密的同时，保证易于使用。

Shiro还增加其他的功能的支持：

* Web支持：Shiro 提供的 Web 支持 api ，可以很轻松的保护 Web 应用程序的安全。

* 缓存：缓存是 Apache Shiro 保证安全操作快速、高效的重要手段。

* 并发：Apache Shiro 支持多线程应用程序的并发特性。
* 等等

注意：Shiro 不会去维护用户、维护权限，这些需要我们自己去设计/提供，然后通过相应的接口注入给 Shiro。

Shiro架构三个主要概念：

* Subject主体：当前用户，可以是一个人，也可以是第三方服务、守护进程帐户、时钟守护任务或者其它–当前和软件交互的任何事件。
* Realms：由我们自己实现认证和授权的过程，然后交给SecurityManager。
* SecurityManager：管理所有Subject，Realms。它是 Shiro 架构的核心，内部自动配置授权、认证等。

Shiro架构中的其他概念：

* Principal身份：是subject进行身份认证的标识，必须唯一，如ID！一个Subject可以有多个身份，但必须有一个主身份。
* credential：是只有Subject自己知道的安全信息，如密码等

## 快速入门

导入依赖

```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

自定义一个Realm类（在这里自定义认证、授权过程，然后交给SecurityManager）

```
class MyRealm extends AuthorizingRealm {

    // 授权过程
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();
        // 一般是从数据库中获取角色和权限数据
        Set<String> roles = Set.of("admin", "user");
        Set<String> permissions = Set.of("user:delete", "user:add");
        //
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setStringPermissions(permissions);
        simpleAuthorizationInfo.setRoles(roles);
        return simpleAuthorizationInfo;
    }

    // 认证过程
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        // 1.从主体传过来的认证信息中，获得用户名
        String userName = (String) authenticationToken.getPrincipal();
        // 2.通过用户名到数据库中获取凭证
        String password = "tom";
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(userName, password, this.getName());
        return authenticationInfo;
    }
}

```

模拟用户登录认证、是否具有角色和权限。

```
public class TestAuthenticator {
    public static void main(String[] args) {
        // 实现自己的 Realm 实例
        MyRealm myRealm = new MyRealm();

        // 构建SecurityManager环境（这段代码一般是在Config类中）
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(myRealm);
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        
        // 获取当前Subject （这段代码一般实在Controller中）
        Subject subject = SecurityUtils.getSubject();
        // 省略从前端获取用户名和密码步骤，直接进行token认证
        UsernamePasswordToken token = new UsernamePasswordToken("tom", "tom");
        // 登录
        subject.login(token);
        // 判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        if (subject.isAuthenticated()) {
            // 判断是否有角色权限
            System.out.println(subject.hasRole("admin"));
            // 判断是否有user.add权限
            System.out.println(subject.isPermitted("user:add"));
        }
        // 登出
        subject.logout();
    }
}
```



## 授权方式

**Shiro中的授权方式：**

* 基于角色的访问控制Role-Based Access Control

    ```
    if (subject.hasRole("admin")){ ... }
    ```

* 基于资源的访问控制Resource-Based Access Control

    ```
    if (subject.isPermission("user:update:*")) { ... }
    ```

第一个`角色`很好理解，和用户是Many vs Many关系；

第二个里面有一个权限字符串，定义规则是：`资源标识符:操作:资源实例标识符`，意思是对哪个资源的哪个实例有什么操作。（可以使用*通配符）用户和权限也是多对多关系。

例如：

* 用户创建权限：`user:create`或者`user:create:*`
* 用户修改实例01的权限：`user:update:01`
* 用户对实例01的所有权限：`user:*:01`

**实现方式**

编程式

```
if (Subject.hasRole("admin")){ ... }
```

注解式

```
@RequireRolse("admin")
public void hello() { ... }
```

标签式（如JSP页面）

```
<shiro:hasRole name="admin"> ... </shiro:hasRole>
```

## SpringBoot2整合

引入依赖

```
<!-- shiro -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

或者

```
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring-boot-starter</artifactId>
	<version>1.5.3</version>
</dependency>
```

自定义一个MyRealm类，和上面一样。

创建一个ShiroConfig类，该类有三个操作

* 拿到MyRealm，交给容器
* 配置一个SecutiryManage，装配好myRealm
* 配置一个ShiroFilter，装配好securityManage，定义过滤规则。Apache Shiro 的核心通过 Filter 来实现，就好像 SpringMvc 通过 DispachServlet 来主控制一样。 既然是使用 Filter 一般也就能猜到，是通过URL规则来进行过滤和权限校验，所以我们需要定义一系列关于URL的规则和访问权限。

```
@Configuration
public class ShiroConfig {
    // 创建自定义Realm
    @Bean
    public Realm realm() {
        return new MyRealm();
    }

    // 创建安全管理器
    @Bean()
    public DefaultWebSecurityManager getDefaultWebSecurityManager() {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(realm());
        return defaultWebSecurityManager;
    }

    // 创建ShiroFilter
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean() {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(getDefaultWebSecurityManager());

        // 默认认证界面
        shiroFilterFactoryBean.setLoginUrl("/login");
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        // 拦截器
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        // 配置不会被拦截的链接 顺序判断
        filterChainDefinitionMap.put("/static/**", "anon");
        filterChainDefinitionMap.put("/index", "anon");
        filterChainDefinitionMap.put("/logout", "anon");
        filterChainDefinitionMap.put("/doLogin", "anon");
        filterChainDefinitionMap.put("/register", "anon");
        filterChainDefinitionMap.put("/doRegister", "anon");

		// 都需要身份认证，否则去登录
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }
}

```

**拦截器分类**

anon表示可以匿名访问

authc表示需要登录才能访问资源

perms表示需要验证用户是否拥有资源权限

roles表示需要验证用户是否用于资源角色

等等

## MD5加密

在自定义Realm的doGetAuthenticationInfo方法中，取到数据库中的密码，传给SecurityManager

```
// 1.从主体传过来的认证信息中，获得用户名
String username = (String) authenticationToken.getPrincipal();
// 2.通过用户名到数据库中获取凭证
User user = userMapper.findByUsername(username);
if (ObjectUtils.isEmpty(user)) {
    return null;
}
SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
        username, // 用户名
        user.getPassword(),  // 密码
        ByteSource.Util.bytes(username),  // 加密的盐，这里使用的是用户名
        this.getName());        // 自定义realm的类名
return authenticationInfo;
```

然后在自定义的ShiroConfig中添加一个Bean

```
@Bean("hashedCredentialsMatcher")
public HashedCredentialsMatcher hashedCredentialsMatcher() {
    HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
    //指定加密方式为MD5
    credentialsMatcher.setHashAlgorithmName("MD5");
    //加密次数
    credentialsMatcher.setHashIterations(10);
    credentialsMatcher.setStoredCredentialsHexEncoded(true);
    return credentialsMatcher;
}
```



## Redis缓存

引入依赖

```
<!-- redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

本地启动redis服务器

项目的配置文件中添加：

```
# redis默认端口
spring.redis.port=6379
spring.redis.host=localhost
# redis中有0-15号库，用0就可以
spring.redis.database=0
```

自定义一个RedisCacheManager类

```
```

在自定义Shiro配置类ShiroConfig中的MyRealm方法中设置RedisCacheManager

## FreeMarker使用shiro

freemaker默认情况下是不能使用shiro标签进行权限控制的。还好已经由大神James Gregory将此问题解决，并将源码发布到了GitHub上面了。GitHub上项目地址：https://github.com/jagregory/shiro-freemarker-tags

这个项目实质上就是实现了一套freemaker的自定义标签，所我们需要在freemarker的配置文件中添加自定义shiro标签。

1、导包

```
<!-- freemarker支持shiro标签的第三方类库 -->
<dependency>
  <groupId>net.mingsoft</groupId>
  <artifactId>shiro-freemarker-tags</artifactId>
  <version>0.1</version>
  <exclusions>
    <exclusion>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
    </exclusion>
    <exclusion>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-all</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

2、在freemaker配置文件中添加自定义shiro标签

```
@Configuration
public class FreemarkerConfig {

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer(FreeMarkerProperties freeMarkerProperties) {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPaths(freeMarkerProperties.getTemplateLoaderPath()); //模板加载路径默认 "classpath:/templates/"
        configurer.setDefaultEncoding("utf-8");//设置页面默认编码（不设置页面中文乱码）
        Map<String,Object> variables=new HashMap<String,Object>();
        variables.put("shiro", new ShiroTags());
        configurer.setFreemarkerVariables(variables);//添加shiro自定义标签
        return configurer;
    }
}
```

3、页面使用shiro相关标签

```
<@shiro.guest>  
 游客访问 <a href = "login.jsp"></ a>
</@shiro.guest> 

user 标签：用户已经通过认证\记住我 登录后显示响应的内容
<@shiro.user>  
 欢迎[<@shiro.principal/>]登录，< a href="/logout.html">退出</ a>  
</@shiro.user>    

authenticated标签：用户身份验证通过，即 Subjec.login 登录成功 不是记住我登录的
<@shiro.authenticated>  
    用户[<@shiro.principal/>]已身份验证通过  
</@shiro.authenticated>   

notAuthenticated标签：用户未进行身份验证，即没有调用Subject.login进行登录,包括"记住我"也属于未进行身份验证
<@shiro.notAuthenticated>
    当前身份未认证（包括记住我登录的）
</@shiro.notAuthenticated> 

principal 标签：显示用户身份信息，默认调用Subjec.getPrincipal()获取，即Primary Principal
<@shiro.principal property="username"/>

hasRole标签：如果当前Subject有角色将显示body体内的内容
<@shiro.hasRole name="admin">  
    用户[<@shiro.principal/>]拥有角色admin<br/>  
</@shiro.hasRole> 

hasAnyRoles标签：如果Subject有任意一个角色(或的关系)将显示body体里的内容
<@shiro.hasAnyRoles name="admin,user,member">  
 用户[<@shiro.principal/>]拥有角色admin或user或member<br/>  
</@shiro.hasAnyRoles>   

lacksRole:如果当前 Subjec没有角色将显示body体内的内容
<@shiro.lacksRole name="admin">  
 用户[<@shiro.principal/>]不拥有admin角色
</@shiro.lacksRole>   

hashPermission:如果当前Subject有权限将显示body体内容
<@shiro.hasPermission name="user:add">  
    用户[<@shiro.principal/>]拥有user:add权限
</@shiro.hasPermission>   

lacksPermission:如果当前Subject没有权限将显示body体内容
<@shiro.lacksPermission name="user:add">  
    用户[<@shiro.principal/>]不拥有user:add权限
</@shiro.lacksPermission> 
```

