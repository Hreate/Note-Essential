# Spring Security

### 1 简介

Spring Security 是针对 Spring 项目的安全框架，也是 Spring Boot 底层安全模块默认的技术选型，它可以实现强大的 Web 安全控制，对于安全控制，我们仅需要引入 `spring-boot-starter-security` 模块，进行少量配置，即可实现强大的安全管理。

原理：AOP

记住几个类：

* `WebSecurityConfigurerAdapter`：自定义 Security 策略
* `AuthenticationManagerBuilder`：自定义认证策略
* `@EnableWebSecurity`：开启 WebSecurity 模式



Spring Security 的两个主要目标是“认证”和“授权”（访问控制）

“认证”（Authentication）

“授权”（Authorization）

这个概念是通用的，而不是只在 Spring Security 中存在



### 2 使用

#### 2.1 模板

```java
package com.example.demo01.config;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }
}
```

#### 2.2 JDBC Authenication

```java
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
    // ensure the passwords are encoded properly
    UserBuilder users = User.withDefaultPasswordEncoder();
    auth
        .jdbcAuthentication()
            .dataSource(dataSource)
            .withDefaultSchema()
            .withUser(users.username("user").password("password").roles("USER"))
            .withUser(users.username("admin").password("password").roles("USER","ADMIN"));
}
```

