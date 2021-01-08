# Spring Boot

## 任务

### 异步任务

* 在 Spring 3.x 之后，内置了 `@Async` 来代替多线程
* 基于 `@Async` 标注的方法，称之为异步方法；这些方法在执行的时候，将会在独立的线程中被执行

* 步骤
  1. 在需要异步执行的方法上添加注解 `@Async`
  2. 在启动类上添加注解 `@EnableAsync`，开启异步支持



### 邮件发送

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-mail</artifactId>
   </dependency>
   ```

2. application.yaml

   ```yaml
   spring:
     mail:
       username: 398128446@qq.com
       password: fhslpoeuayukbhai
       host: smtp.qq.com
   ```

3. 测试使用

   ```java
   package com.example.task02;
   
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.mail.SimpleMailMessage;
   import org.springframework.mail.javamail.JavaMailSenderImpl;
   import org.springframework.mail.javamail.MimeMessageHelper;
   
   import javax.mail.MessagingException;
   import javax.mail.internet.MimeMessage;
   import java.io.File;
   
   @SpringBootTest
   class Task02ApplicationTests {
   
   	@Autowired
   	private JavaMailSenderImpl javaMailSender;
   
   	// 简单邮件
   	@Test
   	void contextLoads() {
   		SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
   		simpleMailMessage.setSubject("傻子你好啊");
   		simpleMailMessage.setText("不要放弃啊");
   		simpleMailMessage.setTo("398128446@qq.com");
   		simpleMailMessage.setFrom("398128446@qq.com");
   		javaMailSender.send(simpleMailMessage);
   	}
   
   	// 复杂邮件
   	@Test
   	void testmail() throws MessagingException {
   		MimeMessage mimeMessage = javaMailSender.createMimeMessage();
   		// 组装
   		MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
   		// 正文
   		mimeMessageHelper.setSubject("废物你好呀");
   		mimeMessageHelper.setText("<p style='color:red'>咬住牙</p>", true);
   		// 附件
   		mimeMessageHelper.addAttachment("1.jpg", new File("C:\\Users\\lenovo\\Desktop\\1.jpg"));
   		mimeMessageHelper.setTo("398128446@qq.com");
   		mimeMessageHelper.setFrom("398128446@qq.com");
   		javaMailSender.send(mimeMessage);
   	}
   }
   
   ```

   

### 定时任务

1. 开启定时功能

   ```java
   package com.example.task03;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.scheduling.annotation.EnableScheduling;
   
   @SpringBootApplication
   @EnableScheduling // 开启定时功能
   public class Task03Application {
   
   	public static void main(String[] args) {
   		SpringApplication.run(Task03Application.class, args);
   	}
   
   }
   ```

2. 测试使用

   ```java
   package com.example.task03.service;
   
   import org.springframework.scheduling.annotation.Scheduled;
   import org.springframework.stereotype.Service;
   
   @Service
   public class ScheduledService {
       @Scheduled(cron = "* * * * * ?")
       public void hello() {
           System.out.println("hello，你被执行了");
       }
   }
   ```

   

# Redis

说明：在 Spring Boot 2.x 之后，原来使用的 jedis 被替换为 lettuce

jedis：采用的直连，多个线程操作不安全。避免问题发生，使用 jedis pool 连接池，类似 BIO 模式

lettuce：采用 netty，实例可以在多个线程中共享，不存在线程不安全的情况，可以减少线程数，类似 NIO 模式

源码分析：

```java
@Bean
@ConditionalOnMissingBean(name = "redisTemplate")
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
@ConditionalOnMissingBean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```



> 整合测试

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. application.yaml 配置连接

   ```yaml
   spring:
     redis:
       host: 192.168.147.129
       port: 6379
   ```

3. 测试

   ```java
   package com.example.demo01;
   
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.data.redis.connection.RedisConnection;
   import org.springframework.data.redis.core.RedisTemplate;
   
   @SpringBootTest
   class Demo01ApplicationTests {
   
   	@Autowired
   	private RedisTemplate redisTemplate;
   
   	@Test
   	void contextLoads() {
   		// opsForValue() 操作字符串
   		// opsForList() 操作 List
   		// opsForHash() 操作 Hash
   		// opsForSet() 操作 Set
   		// opsForZSet() 操作 Sorted Set
   		// opsForGeo() 操作 地理位置信息
   
   		// 获取 Redis 的连接对象
   //		RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
   //		connection.flushDb();
   //		connection.flushAll();
   
   		redisTemplate.opsForValue().set("mykey", "我太难了");
   		System.out.println(redisTemplate.opsForValue().get("mykey"));
   	}
   
   }
   ```



> 自定义 RedisTemplate

```java
package com.example.demo01.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class RedisConfig {
    // 自定义 redisTemplate
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        // Json 序列化配置
        Jackson2JsonRedisSerializer<Object> objectJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectJackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // String 序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        // 配置具体的序列化方式
        // key 采用 String 的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // value 采用 jackson 的序列化方式
        template.setValueSerializer(objectJackson2JsonRedisSerializer);
        // hash 的 key 采用 String 的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // hash 的 value 采用 jackson 的序列化方式
        template.setHashValueSerializer(objectJackson2JsonRedisSerializer);

        return  template;
    }
}
```

> 封装工具类

redisdemo/demo01/utils/RedisUtil



## 分布式 Dubbo + ZooKeeper + Spring Boot

> Provider

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>2.7.8</version>
   </dependency>
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-recipes</artifactId>
       <version>5.1.0</version>
   </dependency>
   ```

2. application.yaml

   ```yaml
   dubbo:
     application:
       name: provider
     registry:
       address: 192.168.147.129
       port: 2181
       protocol: zookeeper
     protocol:
       name: dubbo
       port: 20880
   server:
     port: 8080
   ```

3. 启动类

   ```java
   // @EnableDubbo  //会扫描所有的包，从中找出 dubbo 的 @DubboService 标注的类
   // @DubboComponentScan(basePackages = "com.chy.user-service.service")  //只扫描指定的包
   ```

4. service 实现类

   ```java
   @DubboService
   ```

> Consumer

前两步同上

3. RPC 调用，注入接口需与 Provider 同级

   ```java
   @DubboReference
   private TicketService ticketService;
   ```

   