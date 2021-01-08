# Swagger

学习目标

* 了解 Swagger 的作用和概念
* 了解前后端分离
* 在 Spring Boot 中集成 Swagger



### 1 简介

#### 1.1 前后端分离

* Vue + Spring Boot
* 后端：后端控制层、服务层、数据访问层
* 前端：前端控制层、视图层
* 前后端相对独立，松耦合
* 前端通过调用后端 API 接口，获取数据
* 前后端可以部署在不同的服务器上



#### 1.2 问题

* 前后端集成联调，前端人员和后端人员无法做到“及时协商，尽早解决”，最终导致问题集中爆发



#### 1.3 解决方案

* 首先制订 schema，实时更新最新 API，降低集成的风险
* 后端提供接口，需要实时更新最新的改动及消息
* 前端测试后端接口



#### 1.4 Swagger 简介

* 号称世界上最流行的 API 框架
* RESTful API 文档在线自动生成工具 => API 文档与 API 定义同步更新
* 直接运行，可以在线测试 API 接口
* 支持多种语言



### 2 Spring Boot 集成 Swagger

#### 2.1 步骤

* 示例：https://www.cnblogs.com/architectforest/p/13470170.html

* 导入依赖

  ```xml
  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-boot-starter</artifactId>
      <version>3.0.0</version>
  </dependency>
  ```
  
* 编写配置文件

  * `application.yml`，默认为 true

    ```yaml
    springfox:
      documentation:
        swagger-ui:
          enabled: true
    ```
    
    生产环境中要设置未 false
    
  * Swagger3Config.java

    ```java
    package com.example.demo01.config;
    
    import org.springframework.context.annotation.Configuration;
    import springfox.documentation.oas.annotations.EnableOpenApi;
    
    @Configuration
    @EnableOpenApi
    public class SwaggerConfig {
    
    }
    ```

* 测试，访问：http://127.0.0.1:8080/swagger-ui/index.html



#### 2.2 配置 Swagger

* Swagger 的 Bean 实例 `Docket`

  ```java
  package com.example.demo01.config;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import springfox.documentation.builders.ApiInfoBuilder;
  import springfox.documentation.oas.annotations.EnableOpenApi;
  import springfox.documentation.service.ApiInfo;
  import springfox.documentation.service.Contact;
  import springfox.documentation.spi.DocumentationType;
  import springfox.documentation.spring.web.plugins.Docket;
  
  @Configuration
  @EnableOpenApi
  public class SwaggerConfig {
  
      // 配置了 Swagger 的 Docket 的 Bean 实例
      @Bean
      public Docket createRestApi() {
          return new Docket(DocumentationType.OAS_30)
                  .apiInfo(apiInfo());
      }
  
      // 配置 Swagger 信息
      private ApiInfo apiInfo() {
          return new ApiInfoBuilder()
                  .title("Hreate 的 API 文档")
                  .description("不要放弃，学不死，往死里学")
                  .version("v1.0")
                  .termsOfServiceUrl("http://blog.hreate.com")
                  .contact(new Contact("Hreate", "http://blog.hreate.com", "398128446@qq.com"))
                  .license("Apache 2.0")
                  .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0")
                  .build();
      }
  }
  ```

* 配置扫描接口

  使用方法：`Docket().select().build();`

  ```java
  // 配置了 Swagger 的 Docket 的 Bean 实例
  @Bean
  public Docket createRestApi() {
      return new Docket(DocumentationType.OAS_30)
          .apiInfo(apiInfo())
          .select()
          // 配置要扫描接口的方式
          // basePackage()：扫描指定的包
          // any()：扫描全部
          // none()：不扫描
          // withClassAnnotation()：扫描类上的注解
          // withMethodAnnotation()：扫描方法上的注解
          .apis(RequestHandlerSelectors.basePackage("com.example.demo01.controller"))
          // 扫描指定路径
          .paths(PathSelectors.any())
          .build();
  }
  ```

* 补充：根据 profiles，启动 Swagger

  ```java
  // 配置了 Swagger 的 Docket 的 Bean 实例
  @Bean
  public Docket createRestApi(Environment environment) {
  
      // 设置要启用 Swagger 的使用环境
      Profiles profiles = Profiles.of("dev", "test");
      // 通过 environment.acceptsProfiles 判断是否处在需要启用的环境中
      boolean flag = environment.acceptsProfiles(profiles);
  
      return new Docket(DocumentationType.OAS_30)
          .apiInfo(apiInfo())
          .enable(flag) // 是否启动 Swagger，默认为 true
          .select()
          // 配置要扫描接口的方式
          // basePackage()：扫描指定的包
          // any()：扫描全部
          // none()：不扫描
          // withClassAnnotation()：扫描类上的注解
          // withMethodAnnotation()：扫描方法上的注解
          .apis(RequestHandlerSelectors.basePackage("com.example.demo01.controller"))
          // 扫描指定路径
          .paths(PathSelectors.any())
          .build();
  }
  ```

* 配置 API 文档的分组

  ```java
  .groupName("group1")
  ```

* 配置多个分组：多个 Docket 实例

* 实体类

  ```java
  package com.example.demo01.pojo;
  
  import io.swagger.annotations.ApiModel;
  import io.swagger.annotations.ApiModelProperty;
  import io.swagger.v3.oas.annotations.media.Schema;
  import io.swagger.v3.oas.annotations.tags.Tag;
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  @ApiModel(description = "用户实体类")
  public class User {
      @ApiModelProperty("用户名")
      private String username;
      @ApiModelProperty("密码")
      private String password;
  }
  ```

* Controller

  ```java
  package com.example.demo01.controller;
  
  import com.example.demo01.pojo.User;
  import io.swagger.v3.oas.annotations.Operation;
  import io.swagger.v3.oas.annotations.Parameter;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class HelloController {
  
      @GetMapping("/hello")
      @Operation(summary = "测试示例")
      public String hello() {
          return "hello";
      }
  
      // 只要接口的返回值中存在实体类，该实体类就会被 Swagger 扫描到
      @PostMapping("/user")
      public User user() {
          return new User();
      }
  
      @GetMapping("/sayhello")
      public String sayHello(@Parameter(description = "用户名") String username) {
          return "Hello " + username;
      }
  
      @PostMapping("/showuser")
      public User showUser(User user) {
          return user;
      }
  }
  ```



#### 3 总结

* 可以通过 Swagger 给一些比较难理解的属性或者接口增加注释信息
* 接口文档实时更新
* 可以在线测试
* 注意：在上线时，关闭 Swagger