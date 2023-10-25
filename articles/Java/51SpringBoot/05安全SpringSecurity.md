

记住几个类：

* `WebSecurityConfigurerAdapter`自定义Security策略
* `AuthenticationManagerBuilder`自定义认证策略
* `@EnableWebSecurity`开启Security模式

Spring Security有两个目标

* 认证`Authentication`
* 授权`Authorization`

步骤：

1、引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

2、自定义配置类（我这里没成功，可能和登录路径有关系）

```
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // 授权
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll() // 允许所有人
                .antMatchers("/blogManage/**").hasRole("vip"); // 管理页需要有vip权限
        // 没有权限调转登录页
         http.formLogin();
         http.logout().logoutUrl("/logout");
    }

    // 认证
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 这些数据应该从数据库读取
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()) // 密码加密
                .withUser("admin")
                .password(new BCryptPasswordEncoder().encode("admin"))
                .roles("vip", "vip2"); // 加角色
    }
}
```

3、也可以想shiro一样在前端使用权限标签