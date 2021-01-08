# MyBatis-Plus

### 1 简介

MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生



### 2 特性

* **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响
* **损耗小**：启动即会自动注入基本 CRUD，性能基本无损耗，直接面向对象操作
* **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
* **支持 Lambda 形式调用**：通过 Lambda 表达式，方便地编写各类查询条件，无需再担心字段写错
* **支持主键自动生成**：支持多达4种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
* **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
* **支持自定义全局通用操作**：支持全局通用方法注入（Write once, use anywhere）
* **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper、Model、Service、Controller 层代码，支持模板引擎，更有超多自定义配置
* **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
* **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
* **内置全局拦截插件**：提供全表 delete、update操作智能分析阻断，也可自定义拦截规则，预防误操作



### 3 快速入门

#### 3.1 地址

* https://baomidou.com/guide/quick-start.html



#### 3.2 使用第三方插件

1. 导入对应依赖
2. 研究依赖如何配置
3. 代码如何编写
4. 提高扩展技术能力



#### 3.3 步骤

1. 创建数据库

2. 创建 user表

   ```mysql
   use db1;
   
   DROP TABLE IF EXISTS user;
   
   CREATE TABLE user
   (
   	id BIGINT(20) NOT NULL COMMENT '主键ID',
   	name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
   	age INT(11) NULL DEFAULT NULL COMMENT '年龄',
   	email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
   	PRIMARY KEY (id)
   );
   -- 真实开发中，还需要如下字段：version（乐观锁）、is_deleted（逻辑删除）、gmt_create、gmt_modified
   
   DELETE FROM user;
   
   INSERT INTO user (id, name, age, email) VALUES
   (1, 'Jone', 18, 'test1@baomidou.com'),
   (2, 'Jack', 20, 'test2@baomidou.com'),
   (3, 'Tom', 28, 'test3@baomidou.com'),
   (4, 'Sandy', 21, 'test4@baomidou.com'),
   (5, 'Billie', 24, 'test5@baomidou.com');
   ```

3. 编写项目，使用 SpringBoot 初始化项目

4. 导入依赖

   ```xml
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
   </dependency>
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <optional>true</optional>
   </dependency>
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.4.0</version>
   </dependency>
   ```

   说明：使用 MyBatis-Plus 可以节省大量的代码，尽量不要同时导入 MyBatis 和 MyBatis-Plus

5. 连接数据库

   ```yml
   spring:
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql:///db1?useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
       data-username: root
       data-password: greenday
   ```

6. 使用了 MyBatis-Plus 之后

   * pojo

     ```java
     package com.example.demo01.pojo;
     
     import lombok.AllArgsConstructor;
     import lombok.Data;
     import lombok.NoArgsConstructor;
     
     @Data
     @AllArgsConstructor
     @NoArgsConstructor
     public class User {
     
         private Long id;
         private String name;
         private Integer age;
         private String email;
     }
     ```

   * mapper  接口

     ```java
     package com.example.demo01.mapper;
     
     import com.baomidou.mybatisplus.core.mapper.BaseMapper;
     import com.example.demo01.pojo.User;
     
     // 在对应的 Mapper 上继承基本的接口 BaseMapper
     public interface UserMapper extends BaseMapper<User> {
         // 所有的 CRUD 操作都已经编写完成
     }
     ```

   * 在启动类上添加注解`@MapperScan("com.example.demo01.mapper")`

   * 测试

     ```java
     package com.example.demo01;
     
     import com.example.demo01.mapper.UserMapper;
     import org.junit.jupiter.api.Test;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.boot.test.context.SpringBootTest;
     import org.springframework.util.ObjectUtils;
     
     @SpringBootTest
     class Demo01ApplicationTests {
     
         private UserMapper userMapper;
     
         @Autowired
         private void Demo01ApplicationTests(UserMapper userMapper) {
             this.userMapper = userMapper;
         }
     
         @Test
         void contextLoads() {
             // 参数是一个 Wrapper，条件构造器
             List<User> userList = userMapper.selectList(null);
             Optional.ofNullable(userList)
                     .ifPresent(list -> list.forEach(System.out::println));
         }
     
     }
     ```
     


### 4 配置日志

* application.yml

  ```yaml
  mybatis-plus:
    configuration:
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  ```



### 5 CRUD扩展

#### 5.1 插入操作

```java
// 测试插入
@Test
public void testInsert() {
    User user = new User();
    user.setName("Hreate");
    user.setAge(3);
    user.setEmail("398128446@qq.com");
    int num = userMapper.insert(user); // 自动生成ID
    System.out.println(num); // 受影响的行数
    System.out.println(user);
}
```



##### 5.1.1 主键生成策略

* 包括：uuid、自增id、雪花算法、redis、zookeeper

* 雪花算法：

  snowflake 是 Twitter 开源的分布式ID生成算法，结果是一个long型的ID。其核心思想是：使用41bit作为毫秒数，10bit作为机器的ID（5个bit是数据中心，5个bit的机器ID），12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生4096个ID），最后还有一个符号位，永远是0

  ![](C:\Users\lenovo\Pictures\Screenshots\13382703-b64e38457ddd13e2.jpg)

* 默认（雪花算法）

  ```java
  @TableId(type = IdType.ASSIGN_ID)
  ```

* 配置主键自增：

  1. 实体类属性上 `@TableId(type = IdType.AUTO)`
  2. 数据库字段一定要设置成自增

* 源码解释

  https://baomidou.com/guide/annotation.html#tableid

  

#### 5.2 更新操作

```java
// 测试更新
@Test
public void testUpdate() {
    User user = new User();
    // 通过条件自动拼接动态sql
    user.setId(1322362313323417603L);
    user.setName("acho");
    user.setAge(18);
    // 参数是一个对象
    int num = userMapper.updateById(user);
    System.out.println(num);
}
```



##### 5.2.1 自动填充

创建时间、修改时间，这些操作一般是自动化完成的，不希望手动更新

阿里巴巴开发手册：所有的数据库表：gmt_create（创建时间）、gmt_modified（修改时间）

  >方式一：数据库级别（工作中没有权限修改数据库）

1. 在表中新增字段

   ```mysql
   alter table user add `create_time` datetime default current_timestamp comment '创建时间';
   alter table user add `update_time` datetime on update current_timestamp default current_timestamp comment '更新时间';
   ```

2. 再次测试插入方法，测试前需要先把实体类同步

   ```java
   private Date createTime;
   private Date updateTime;
   ```

   

>方式二：代码级别

1. 删除数据库的默认值和自动更新

   ```mysql
   alter table user modify create_time datetime comment '创建时间';
   alter table user modify update_time datetime comment '更新时间';
   ```

2. 实体类相应的属性上增加注解

   ```java
   // 字段添加填充内容
   @TableField(fill = FieldFill.INSERT)
private Date createTime;
   @TableField(fill = FieldFill.INSERT_UPDATE)
   private Date updateTime;
   ```
   
3. 编写处理器来处理这个注解即可

   ```java
   package com.example.demo01.handler;
   
   import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
   import lombok.extern.slf4j.Slf4j;
   import org.apache.commons.logging.Log;
   import org.apache.ibatis.reflection.MetaObject;
   import org.springframework.stereotype.Component;
   
   import java.util.Date;
   
   @Slf4j
   @Component
   public class MyMetaObjectHandler implements MetaObjectHandler {
       // 插入时的填充策略
       @Override
       public void insertFill(MetaObject metaObject) {
           log.info("start insert fill...");
           this.setFieldValByName("createTime", new Date(), metaObject);
           this.setFieldValByName("updateTime", new Date(), metaObject);
       }
   
       //更新时的填充策略
       @Override
       public void updateFill(MetaObject metaObject) {
           log.info("start update fill...");
           this.setFieldValByName("updateTime", new Date(), metaObject);
       }
   }
   ```



#### 5.3 乐观锁

##### 5.3.1 乐观锁实现方式

* 取出记录时，获取当前 version
* 更新时，带上这个 version
* 执行更新时，`set version = newVersion where version = oldVersion`
* 如果 version 不对，就更新失败



##### 5.3.2 乐观锁插件使用

1. 给数据库表增加 version 字段

   ```mysql
   alter table user add version int(10) not null default 1 comment '乐观锁' after email;
   ```

2. 实体类增加对应属性

   ```java
   @Version // 乐观锁 version 注解
   private Integer version;
   ```

3. 注册组件

   ```java
   package com.example.demo01.config;
   
   import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
   import com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor;
   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.transaction.annotation.EnableTransactionManagement;
   
   @MapperScan("com.example.demo01.mapper")
   @EnableTransactionManagement
   @Configuration
   public class MyBatisPlusConfig {
   
       // 注册乐观锁插件
       @Bean
       public MybatisPlusInterceptor mybatisPlusInterceptor() {
           MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
           interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
           return interceptor;
       }
   }
   ```

4. 测试

   ```java
   // 测试乐观锁成功
   @Test
   public void testOptimisticLockerSuccess() {
       // 1、查询用户信息
       User user = userMapper.selectById(1L);
       // 2、修改用户信息
       user.setName("Neo");
       user.setEmail("398128446@qq.com");
       // 3、执行更新操作
       int num = userMapper.updateById(user);
       System.out.println(num);
   }
   
   // 测试乐观锁失败，多线程下
   @Test
   public void testOptimisticLockerFailed() {
       // 线程1
       User user = userMapper.selectById(1L);
       user.setName("Neo111");
       user.setEmail("398128446@qq.com");
   
       // 模拟另外一个线程执行了插队操作
       User user2 = userMapper.selectById(1L);
       user2.setName("Neo222");
       user2.setEmail("398128446@qq.com");
       int num1 = userMapper.updateById(user2);
       System.out.println(num1);
   
       // 自旋锁来多次尝试提交
       int num = userMapper.updateById(user);
       System.out.println(num);
   }
   ```



#### 5.4 查询操作

```java
// 测试查询
@Test
public void testSelectById() {
    User user = userMapper.selectById(1L);
    System.out.println(user);
}

// 测试批量查询
@Test
public void testSelectBatchIds() {
    List<User> userList = userMapper.selectBatchIds(Arrays.asList(1, 2, 3));
    Optional.ofNullable(userList)
        .ifPresent(list -> list.forEach(System.out::println));
}

// 条件查询之一使用 map 进行条件封装
@Test
public void testSelectByMap() {
    // 自定义查询条件
    Map<String, Object> map = new HashMap<>();
    map.put("name", "Hreate");
    map.put("age", 3);
    List<User> userList = userMapper.selectByMap(map);
    Optional.ofNullable(userList)
        .ifPresent(list -> list.forEach(System.out::println));
}
```



##### 5.4.1 分页查询

1. `limit` 关键字进行分页

2. PageHelper 第三方插件

3. MyBatis-Plus 内置分页插件 PaginationInnerInterceptor

   > 使用

   1. 配置拦截器组件

      ```java
      /**
       * 新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免
       * 缓存出现问题
       */
      @Bean
      public MybatisPlusInterceptor mybatisPlusInterceptor() {
          MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
          PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
          // 设置请求的页面大于最大页后操作，true 调回到首页，false 继续请求，默认false
          // paginationInnerInterceptor.setOverflow(false);
          // 设置最大单页限制数量，默认500条，-1 不受限制
          // paginationInnerInterceptor.setMaxLimit(500L);
          interceptor.addInnerInterceptor(paginationInnerInterceptor);
          return interceptor;
      }
      
      @Bean
      public ConfigurationCustomizer configurationCustomizer() {
          return configuration -> configuration.setUseDeprecatedExecutor(false);
      }
      ```

   2. 直接使用 Page 对象即可

      ```java
      // 测试分页查询、
      @Test
      public void testPagination() {
          // 参数一：当前页；参数二：页面大小
          Page<User> page = new Page<>(2, 5);
          userMapper.selectPage(page, null);
          List<User> userList = page.getRecords();
          Optional.ofNullable(userList)
              .ifPresent(list -> list.forEach(System.out::println));
          System.out.println(page.getTotal());
      }
      ```



#### 5.5 删除操作

> ##### 物理删除

```java
// 测试删除
@Test
public void testDeleteById() {
    int num = userMapper.deleteById(1322362313323417604L);
    System.out.println(num);
}

// 测试批量删除
@Test
public void testDeleteBatchIds() {
    int num = userMapper.deleteBatchIds(Arrays.asList(1322362313323417602L, 1322362313323417603L));
    System.out.println(num);
}

//测试使用 map 封装条件删除
@Test
public void testDeleteByMap() {
    Map<String, Object> map = new HashMap<>();
    map.put("name", "Hreate");
    map.put("age", 3);
    int num = userMapper.deleteByMap(map);
    System.out.println(num);
}
```



> ##### 逻辑删除

防止数据丢失，类似于回收站

1. 在数据表中增加一个 deleted 字段

   ```mysql
   alter table user add deleted tinyint(1) not null default 0 comment '逻辑删除';
   ```

2. 实体类中增加属性

   ```java
   @TableLogic // 逻辑删除
   private Boolean deleted;
   ```

3. 配置

   ```yaml
   # 此为默认值，如果你的默认值和 MyBatis-Plus 默认的一样,该配置可省略
   mybatis-plus:
     global-config:
       db-config:
         logic-delete-value: true
         logic-not-delete-value: false
   ```

   

### 6 条件构造器

十分重要：Wrapper

写一些复杂的 sql 时就可以用 Wrapper 来替代

1. 测试一

   ```java
   @Test
   void contextLoads() {
       // 查询 name 不为空并且 email 不为空并且 age > 12 的用户
       QueryWrapper<User> wrapper = new QueryWrapper<>();
       wrapper.isNotNull("name")
           .isNotNull("email")
           .ge("age", 12);
       List<User> userList = userMapper.selectList(wrapper);
       Optional.ofNullable(userList).ifPresent(list -> list.forEach(System.out::println));
   }
   ```

2. 测试二

   ```java
   @Test
   void test2() {
       // 查询名字为 Hreate
       QueryWrapper<User> wrapper = new QueryWrapper<>();
       wrapper.eq("name", "Hreate");
       User user = userMapper.selectOne(wrapper);
       Optional.ofNullable(user).ifPresent(System.out::println);
   }
   ```

3. 测试三

   ```java
   @Test
   void test3() {
       // 查询年龄在20~30岁之间的用户数量
       QueryWrapper<User> wrapper = new QueryWrapper<>();
       wrapper.between("age", 20, 30);
       Integer count = userMapper.selectCount(wrapper);// 查询结果数
       System.out.println(count);
   }
   ```

4. 测试4

   ```java
   
   ```

   