# MyBatis

1. #### 导入依赖，数据库连接驱动依赖和 mybatis 依赖。

2. #### 编写 mybatis 的核心配置文件。

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"></transactionManager>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/db1?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
                   <property name="username" value="root"/>
                   <property name="password" value="greenday"/>
               </dataSource>
           </environment>
       </environments>
       <!--每一个 Mapper.xml 都需要在 MyBatis 核心配置文件中注册-->
       <mappers>
           <!--<mapper resource="com/example/demo/dao/UserMapper.xml"></mapper>-->
           <!-- xml 文件名必须和接口文件名相同-->
           <package name="com.example.demo.dao"/>
       </mappers>
   </configuration>
   ```

3. #### 加载核心配置文件，获取 SqlSession 对象，可封装成工具类：

   ```java
   public class MyBatisUtils {
       static {
           try {
               // 使用MyBatis的第一步，获取 SqlSessionFactory 对象
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
       
       // 获得 SqlSession 的实例
       // SqlSession 完全包含了面向数据库执行 SQl 命令所需的所有方法
       public static SqlSession getSqlSession() {
           return sqlSessionFactory.openSession();
       }
   }
   ```

4. #### 编写代码

   * 实体类

     ```java
     package com.example.demo.pojo;
     
     /**
      * user 实体类
      *
      * @author Acho
      * @since 2020-10-15 00:09
      **/
     public class User {
         private int id;
         private String name;
         private int age;
     
         public User() {
         }
     
         public User(int id, String name, int age) {
             this.id = id;
             this.name = name;
             this.age = age;
         }
     
         public int getId() {
             return id;
         }
     
         public void setId(int id) {
             this.id = id;
         }
     
         public String getName() {
             return name;
         }
     
         public void setName(String name) {
             this.name = name;
         }
     
         public int getAge() {
             return age;
         }
     
         public void setAge(int age) {
             this.age = age;
         }
         
         @Override
         public String toString() {
             return "User{" +
                     "id=" + id +
                     ", name='" + name + '\'' +
                     ", age=" + age +
                     '}';
         }
     }
     
     ```

   * Dao 接口

     ```java
     public interface UserDao {
         // 查询全部用户
         List<User> findUserList();
         // 根据ID查询用户
      User findUserByID(int ID);
     }
     ```
     
   * 接口实现类由原来的 UserDaoImpl 转变为一个 Mapper 配置文件 Mapper.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE mapper
             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <!-- namespace 绑定一个对应的 Dao/Mapper 接口的全限定类名-->
     <mapper namespace="com.example.demo.dao.UserMapper">
         <select id="findUserList" resultType="com.example.demo.pojo.User">
             select * from user
         </select>
         <select id="findUserById" parameterType="int" resultType="com.example.demo.pojo.User">
             select * from user where id = #{id}
         </select>
     </mapper>
     ```

   * 测试

     ```java
     // 获得 SqlSession 对象
     SqlSession sqlSession = MyBatisUtils.getSqlSession();
     
     // 方式一（推荐）：getMapper
     UserDao mapper = sqlSession.getMapper(UserDao.class);
     List<User> userList = mapper.getUserList();
     
     // 方式二（不推荐）：
       // List<User> userList = sqlSession.selectList("com.example.demo.dao.UserMapper.getUserList");
     System.out.println(userList);
     
     // 关闭 SqlSession
     sqlSession.close();
     ```

     ```java
     // 另一种写法，JDK 7 中加入的自动关闭资源
     try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
         UserMapper mapper = sqlSession.getMapper(UserMapper.class);
         List<User> userList = mapper.getUserList();
         System.out.println(userList);
     }
     ```

     ```java
     try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
         UserMapper mapper = sqlSession.getMapper(UserMapper.class);
         User user = mapper.findUserByID(2);
         System.out.println(user);
     }     
     ```

     

5. #### CUD 增删改需要提交事务

   * Create（增加）：

     1. Dao 接口

        ```java
        // 增加用户
            int addUser(User user);
        ```

     2. Mapper.xml

        ```xml
        <!--对象中的属性可以直接取用-->
        <insert id="addUser" parameterType="com.example.demo.pojo.User">
        	insert into user (id, name, age) values (#{id}, #{name}, #{age})
        </insert>
        ```

     3. 测试

        ```java
        try (final SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            int num = mapper.addUser(new User(4, "Neo", 80));
            // 提交事务
            System.out.println(num);
            if (num > 0) {
                sqlSession.commit();
            }
        }
        ```

   * Update（修改）

     1. Dao 接口

        ```java
        // 修改用户
        int updateUser(User user);
        ```

     2. Mapper.xml

        ```xml
        <update id="updateUser" parameterType="com.example.demo.pojo.User">
            update user set name=#{name}, age=#{age} where id=#{id}
        </update>
        ```

     3. 测试

        ```java
        try (final SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            int num = mapper.updateUser(new User(4, "吓破胆", 26));
            System.out.println(num);
            if (num > 0) {
                sqlSession.commit();
            }
        }
        ```

   * Delete （删除）

     1. Dao 接口

        ```java
        // 删除用户
        int deleteUser(int id);
        ```

     2. Mapper.xml

        ```xml
        <delete id="deleteUser">
            delete from user where id=#{id}
        </delete>
        ```

     3. 测试

        ```java
        try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            int num = mapper.deleteUser(4);
            System.out.println(num);
            if (num > 0) {
                sqlSession.commit();
            }
        }
        ```
   
6. #### 使用 Map 传参

   如果实体类或者数据库中的表，字段或者参数过多，此时应当考虑使用 Map ！

   * Dao 接口

     ```java
     int addUserByMap(Map<String, Object> map);
     ```

   * Mapper.xml

     ```xml
     <insert id="addUserByMap" parameterType="map">
         insert into user (id, name, age) values (#{userId}, #{username}, #{userAge})
     </insert>
     ```

   * 测试

     ```java
     try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
         UserMapper mapper = sqlSession.getMapper(UserMapper.class);
         Map<String, Object> map = new HashMap<>();
         map.put("userId", 5);
         map.put("username", "加油");
         map.put("userAge", 27);
         int num = mapper.addUserByMap(map);
         System.out.println(num);
         if (num > 0) {
             sqlSession.commit();
         }
     }
     ```

7. #### 模糊查询

   * Dao 接口

     ```java
     // 模糊查询
     List<User> getUserLike();
     ```

   * Mapper.xml

     ```xml
     <select id="findUserLike" resultType="com.example.demo.pojo.User">
         select * from user where name like #{value}
     </select>
     ```

   * 测试

     ```java
     try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
         UserMapper mapper = sqlSession.getMapper(UserMapper.class);
         List<User> userList = mapper.findUserLike("%通%");
         System.out.println(userList);
     }
     ```

     

#### 注意：

MyBatis 默认关闭自动提交

```java
// 通过这种方式打开的SqlSession，autoCommit默认为false，需要手动提交事务
SqlSession sqlSession = sqlSessionFactory.openSession();

// 通过这种方式打开的SqlSession，autoCommit为false，需要手动提交事务
SqlSession sqlSession = sqlSessionFactory.openSession(false);

// 通过这种方式打开的SqlSession，autoCommit为true，会自动提交事务
SqlSession sqlSession = sqlSessionFactory.openSession(true);
```



Map 传递参数，直接在 sql 中取出 key 即可！

对象传递参数，直接在 sql 中取出对象的属性即可！

只有一个基本类型参数的情况下，可以直接在 sql 中取到！

多个参数用 Map，或者注解！



使用SQL 通配符可以替代一个或多个字符，即模糊查询。
SQL 通配符必须与 LIKE 运算符一起使用。在 SQL 中，可使用以下通配符如下：
1、`%`  替代一个或多个字符
2、`_  `仅替代一个字符
3、`[charlist]`  字符列中的任何单一字符
4、`[^charlist]` 或者 `[!charlist]`  不在字符列中的任何单一字符



模糊查询怎么写？

1. Java 代码执行的时候，传递通配符

   ```java
   List<User> userList = mapper.findUserLike("%通%");
   ```

2. 在 sql 中拼接通配符

   ```sql
   select * from user where name like "%"#{value}"%"
   ```




8. #### 配置解析

   1. ##### 核心配置文件

      * mybatis-config.xml

      * MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

        ```
        configuration (配置)
        	* properties (属性)
        	* settings (设置)
        	* typeAliases (类型别名)
        	typeHandlers (类型处理器)
        	objectFactory (对象工厂)
        	plugins (插件)
        	* environments (环境配置)
        		* environment (环境变量)
        			* transactionManager (事务管理器)
        			* dataSource (数据源)
        	databaseIdProvider (数据库厂商标识)
        	* mappers (映射器)
        ```

   2. ##### 环境配置（environments）

      MyBatis 可以配置成适用多种环境

      不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。

      

      ##### 事务管理器（transactionManager）：

      在 MyBatis 中有两种类型的事务管理器（也就是 `type="[JDBC|MANAGED]"`）

      * JDBC - 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。

      * MANAGED - 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如：

        ```xml
        <transactionManager type="MANAGED">
        	<property name="closeConnection" value="false"/>
        </transactionManager>
        ```

      如果使用 Spring + MyBatis ，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

      

      ##### 数据源（dataSource）

      有三种内建的数据源类型（也就是 `type=[UNPOOLED|POOLED|JNDI]`）

      UNPOOLED - 每次被请求时打开和关闭连接。

      POOLED - 连接池。

      JNDI - 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。

      

      自定义数据源类型：

      * 继承 UnpooledDataSourceFactory

        首先，编写一个 Java 类，这个类需要继承 MyBatis 的 `org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory` 这个类。

        实际上，继承 `org.apache.ibatis.datasource.unpooled.PooledDataSourceFactory` 也是可以的，这两种方式没什么区别，因为 `PooledDataSourceFactory` 就是继承自 `UnpooledDataSourceFactory` 的。

        ```java
        public class DruidDataSourceFactory extends UnpooledDataSourceFactory {
        	public DruidDataSourceFactory() {
        		this.dataSource = new DruidDataSource();
        	}
        }
        ```

        实际上很简单，就是在构造方法中将父类中的 dataSource 属性赋值为 DruidDataSource 的一个实例。

      * 修改配置元数据 mybatis-config.xml

        ```xml
        <dataSource type="com.example.DruidDataSourceFactory">
        	<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/db1?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
            <property name="username" value="root"/>
            <property name="password" value="greenday"/>
        </dataSource>
        ```

      

      ##### 属性（properties）

      可以通过 properties 属性来实现引用配置文件

      1. 编写一个配置文件，db.properties

         ```properties
         driver=com.mysql.cj.jdbc.Driver
         url=jdbc:mysql://localhost:3306/db1?useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
         username=root
         password=greenday
         ```

      2. 在核心配置文件中引入

         ```xml
         <properties resource="db.properties">
             <property name="username" value="root"/>
             <property name="password" value="greenday"/>
         </properties>
         ```

         * 可以直接引入外部配置文件
         * 可以在其中增加一些属性配置
         * 如果两个文件有相同属性，外部配置文件的属性优先生效

      

      ##### 类型别名（typeAliases）

      * 为 Java 类型设置一个短的名字
      * 用来减少类完全限定名的冗余

      1. 为每个 Java 类型单独起别名

         ```xml
         <typeAliases>
             <typeAlias type="com.example.demo02.pojo.User" alias="User"/>
         </typeAliases>
         ```

      2. 指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean。在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名

         ```xml
         <typeAliases>
             <package name="com.example.demo02.pojo"/>
         </typeAliases>
         ```

         若有注解，则别名为其注解值

         ```java
         @Alias("user")
         public class User {
             ...
         }
         ```

      ##### 设置（settings）

      这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

      | 设置名            | 描述                                                         | 有效值                                                       | 默认值 |
      | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ |
      | cacheEnable       | 全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。   | true \| false                                                | true   |
      | lazyLoadingEnable | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false  |
      | logImpl           | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

      

   3. ##### 其他配置

      * typeHandlers（类型处理器）
      * objectFactory（对象工厂）
      * plugins（插件）
        * mybatis-generator-core
        * mybatis-plus
        * 通用 mapper

      

   4. ##### 映射器（mappers）

      MapperRegistry：注册绑定 Mapper 文件。

      方式一：

      ```xml
      <!-- 使用相对于类路径的资源引用 -->
      <mappers>
      	<mapper resource="com/example/demo/dao/UserMapper.xml"/>
      </mappers>
      ```

      

      方式二：

      ```xml
      <!-- 使用映射器接口实现类的完全限定类名 -->
      <mappers>
      	<mapper class="com.example.demo.dao.UserMapper"/>
      </mappers>
      ```

      注意点：

      * 接口和他的 Mapper 配置文件必须同名
      * 接口和他的 Mapper 配置文件必须在同一个包下，可以把 Mapper.xml 放在 resources 目录下跟 Dao 接口同级包中（注意：resources 中新建多级包的格式为 `xxx/xxx/xxx`）

      

      方式三，注意点同上：

      ```xml
      <!-- 将包内的映射器接口实现全部注册为映射器 -->
      <mappers>
      	<package name="com.example.demo.dao"/>
      </mappers>
      ```

      

   5. ##### 作用域（scope）和生命周期

      不同的作用域和生命周期类是至关重要的，因为错误的使用会导致非常严重的并发问题。

      * SqlSessionFactoryBuilder

        一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一致存在，以保证所有的 XML 解析资源可以被释放给更重要的事情。

      * SqlSessionFactory

        一旦被创建就应该在应用的运行期间一直存在。使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，因此 SqlSessionFactory 的最佳作用域是应用作用域。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

      * SqlSession

        每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳作用域是请求或方法作用域。

        

9. #### 解决属性名和字段名不一致的问题

   解决方法：

   * 方法一：sql 语句中给不一样的字段起别名

     ```sql
     select id,name as username,age from user where id = #{id}
     ```

   * 方法二：resultMap 结果集映射

     * resultMap 元素是 MyBatis 中最重要最强大的元素。
     * resultMap 的设计思想是，对于简单的语句根本不需要配置显示的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。

     Mapper.xml

     ```xml
     <!-- 结果集映射 -->
     <resultMap id="UserMap" type="userInfo">
         <!-- column数据库中的字段，properties实体类中的属性,只需要手动映射不一样的字段 -->
         <result column="name" property="username"/>
     </resultMap>
     <select id="findUserInfoById" resultMap="UserMap">
         select * from user where id = #{id}
     </select>
     ```

     

10. #### 日志

    1. ##### 日志工厂

       * SLF4J

       * LOG4J

       * LOG4J2 【掌握】

       * JDK_LOGGING

       * COMMONS_LOGGING

       * STDOUT_LOGGING 【掌握】

       * NO_LOGGING

         

       STDOUT_LOGGING标准日志输出

       ```xml
       <settings>
           <setting name="logImpl" value="STDOUT_LOGGING"/>
       </settings>
       ```

       

    2. ##### Log4j

       什么是Log4j?

       * Log4j是Apache的一个开源项目，通过使用Log4j，可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等

       * 可以控制每一条日志的输出格式

       * 通过定义每一条日志信息的级别，能够更加细致地控制日志的生成过程

       * 通过一个配置文件来灵活地进行配置，而不需要修改应用的代码

         

       1. 导入依赖

          ```xml
          <dependency>
              <groupId>log4j</groupId>
              <artifactId>log4j</artifactId>
              <version>1.2.17</version>
          </dependency>
          ```

          

       2. log4j.properties

          ```properties
          # 将等级为 DEBUG 的日志输出到 console 和 file 这两个目的地，console 和 file 的定义在下面的代码
          log4j.rootLogger=DEBUG,console,file
          
          # 控制台输出的相关设置
          log4j.appender.console=org.apache.log4j.ConsoleAppender
          log4j.appender.console.Target=System.out
          log4j.appender.console.Threshold=DEBUG
          log4j.appender.console.layout=org.apache.log4j.PatternLayout
          log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
          
          # 文件输出的相关设置
          log4j.appender.file=org.apache.log4j.RollingFileAppender
          log4j.appender.file.File=./log/record.log
          log4j.appender.file.MaxFileSize=10mb
          log4j.appender.file.Threshold=DEBUG
          log4j.appender.file.layout=org.apache.log4j.PatternLayout
          log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
          
          # 日志输出级别
          log4j.logger.org.mybatis=DEBUG
          log4j.logger.java.sql=DEBUG
          log4j.logger.java.sql.Statement=DEBUG
          log4j.logger.java.sql.ResultSet=DEBUG
          log4j.logger.java.sql.PreparedStatement=DEBUG
          ```

       3. 配置 log4j 为日志的实现

          ```xml
          <settings>
              <setting name="logImpl" value="LOG4J"/>
          </settings>
          ```

       4. log4j 的使用

          1. 在要使用 log4j 的类中，导入包 `import org.apache.log4j.Logger;`

          2. 日志对象，参数为当前类的 class

             ```java
             private Logger logger = Logger.getLogger(Demo02ApplicationTests.class);
             ```

          3. 日志级别

             ```java
             logger.info("info: 进入了 testLog4j 方法");
             logger.debug("debug: 进入了 testLog4j 方法");
             logger.error("error: 进入了 testLog4j 方法");
             ```

             

11. #### 分页

    * ##### 思考：为什么要分页？

    减少数据的处理量

    

    * ##### 使用 Limit 分页

      语法

      ```mysql
      select * from user limit startIndex,pageSize; # startIndex 从0开始
      select * from user limit 3; # 从索引0开始，查询3条
      ```

      使用 MyBatis 实现分页

      1. Dao 接口

         ```java
         //分页
         List<User> findUserByLimit(Map<String, Integer> map);
         ```

      2. Mapper.xml

         ```xml
         <select id="findUserByLimit" parameterType="map" resultType="user">
             select * from user limit #{startIndex},#{pageSize}
         </select>
         ```

      3. 测试

         ```java
         try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
             UserMapper mapper = sqlSession.getMapper(UserMapper.class);
             Map<String, Integer> map = new HashMap<>();
             map.put("startIndex", 1);
             map.put("pageSize", 2);
             List<User> userList = mapper.findUserByLimit(map);
             userList.forEach(System.out::println);
         }
         ```

    * 分页插件 PageHelper

      

12. #### 使用注解开发


    1. 面向接口编程
    
       * 优点：解耦，可拓展，提高复用，分层开发——中、上层不用管具体的实现，大家都遵守共同的标准，使得开发变得容易，规范性更好。
    
       * 关于接口的理解
    
         接口从更深层次的理解，应是定义（规范、约束）与实现（名实分离的原则）的分离。
    
         接口的本身反映了系统设计人员对系统的抽象理解。
    
         接口应有两类：
    
         	*	第一类是对一个个体的抽象，它可对应为一个抽象体（abstract class）;
         	*	第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）；
    
         一个个体可能有多个抽象面。抽象体与抽象面是由区别的。
    
       * 三个面向区别
    
         面向对象是指，以对象为单位，考虑它的属性及方法。
    
         面向过程是指，以一个具体的流程（事务过程）为单位，考虑它的实现。
    
         接口设计与非接口设计是针对复用技术而言的，与面向对象（过程）不是一个问题，更多的体现就是对系统整体的架构。


​         

    2. 开发流程


       1. 注解在接口上实现
    
          ```java
          @Select("select * from user")
          List<User> findUsers();
          ```
    
       2. 需要在核心配置文件中绑定接口
    
          ```xml
          <!-- 绑定接口 -->
          <mappers>
              <!--<package name="com.example.demo03.dao"/>-->
              <mapper class="com.example.demo03.dao.UserMapper"/>
          </mappers>
          ```
    
       本质：反射机制实现
    
       底层：动态代理
    
       3. CRUD 注解
    
          编写接口，增加注解
    
          ```java
          // 方法存在多个参数，所有的参数前面必须加上 @Param 注解
          @Select("select * from user where id = #{id}")
          User findUserById(@Param("id") int id);
          
          @Insert("insert into user (id,name,age) values (#{id},#{name},#{age})")
          int addUser(User user);
          
          @Update("update user set name=#{name},age=#{age} where id=#{id}")
          int updateUser(User user);
          
          @Delete("delete from user where id = #{id}")
          int deleteUser(@Param("id") int id);
          ```
    
          关于 @Param 注解
    
          * 基本类型或者 String 类型的参数，需要加上
    
          * 引用类型不需要加
    
          * 如果只有一个参数，且是基本类型或者 String 类型，可以省略
    
          * 在 sql 中引用的就是 @Param("[属性名]") 中设定的属性名


​            

13. #### Lombok

    使用步骤：

    1. 在 IDEA 中安装 Lombok 插件。

    2. 在项目中导入 Lombok 依赖

       ```xml
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <version>1.18.16</version>
           <scope>provided</scope>
       </dependency>
       ```

    3. 在实体类上加注解

       ```java
       @Getter and @Setter
       @FieldNameConstants
       @ToString
       @EqualsAndHashCode
       @AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
       @Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
       @Data
       @Builder
       @SuperBuilder
       @Singular
       @Delegate
       @Value
       @Accessors
       @Wither
       @With
       @SneakyThrows
       @val
       @var
       experimental @var
       @UtilityClass
       ```

       说明

       ```
       @Data：无参构造、getter、setter、toString、hashCode、equals
       @AllArgsConstructor
       @NoArgsConstructor
       ```

       

14. #### 多对一处理

    SQL：
    
    ```mysql
    create table `teacher` (
        `id` int(10) not null,
        `name` varchar(30) default null,
        primary key (`id`)
    ) engine=INNODB default charset=utf8;
    
    insert into teacher(`id`,`name`) values (1,'人造人');
    
    create table `student` (
        `id` int(10) not null,
        `name` varchar(30) default null,
        `tid` int(10) default null,
        primary key (`id`),
        key `fktid` (`tid`),
        constraint `fktid` foreign key (`tid`) references `teacher` (`id`)
    ) engine=INNODB default charset=utf8;
    
    insert into `student`(`id`, `name`, `tid`) VALUES (1,'小明',1);
    insert into `student`(`id`, `name`, `tid`) VALUES (1,'小红',1);
    insert into `student`(`id`, `name`, `tid`) VALUES (3,'小张',1);
    insert into `student`(`id`, `name`, `tid`) VALUES (4,'小李',1);
    insert into `student`(`id`, `name`, `tid`) VALUES (5,'小王',1);
    ```
    
    
    
    1. ##### 方式一：按照查询嵌套处理
    
       ```xml
       <resultMap id="StudentTeacher" type="student">
           <id property="id" column="id"/>
           <!-- 复杂的属性，我们需要单独处理，对象：association，集合：collection -->
           <association property="teacher" column="tid" javaType="teacher" select="findTeacher"/>
       </resultMap>
       <select id="findStudent" resultMap="StudentTeacher">
           select * from student
       </select>
       <select id="findTeacher" resultType="teacher">
           select * from teacher where id=#{id}
       </select>
       ```
    
    2. ##### 方式二：按照结果嵌套处理
    
       ```xml
       <resultMap id="StudentTeacher" type="student">
           <id column="sid" property="id"/>
           <result column="sname" property="name"/>
           <association property="teacher" javaType="teacher">
               <id column="t_id" property="id"/>
               <result column="tname" property="name"/>
           </association>
       </resultMap>
       <select id="findStudent" resultMap="StudentTeacher">
           select s.id sid,s.name sname,t.id t_id,t.name tname
           from student s,teacher t
           where s.tid=t.id
       </select>
       ```
    
       
    
       回顾 Mysql 多对一查询方式：
    
       * 子查询
       * 联表查询
    
    
    
    15. #### 一对多处理
    
        1. ##### 方式一：按照查询嵌套处理
    
           ```xml
           <!-- 按照查询嵌套处理 -->
           <resultMap id="TeacherStudent" type="teacher">
               <id property="id" column="id"/>
               <collection property="students" column="id" ofType="student" select="findStudentByTeacherId"/>
           </resultMap>
           <select id="findOneTeacher" resultMap="TeacherStudent">
               select * from teacher where id=#{id}
           </select>
           <select id="findStudentByTeacherId" resultType="student">
               select * from student where tid=#{id}
           </select>
           ```
    
           
    
        2. ##### 方式二：按照结果嵌套处理
    
           ```xml
           <!-- 按结果嵌套查询 -->
           <resultMap id="TeacherStudent" type="teacher">
               <id column="t_id" property="id"/>
               <result column="tname" property="name"/>
               <collection property="students" ofType="student">
                   <id column="sid" property="id"/>
                   <result column="sname" property="name"/>
               </collection>
           </resultMap>
           <select id="findOneTeacher" resultMap="TeacherStudent">
               select t.id t_id,t.name tname,s.id sid,s.name sname
               from teacher t,student s
               where t.id=#{id} and s.tid = t.id
           </select>
           ```
    
           
    
        3. ##### 小结
    
           * resultMap 中一定要将主键 `id` 显式注明
    
           * 关联 - association 用 `javaType`
    
           * 集合 - collection 用 `ofType`
    
             
    
15. ##### 动态 SQL

    什么是动态 SQL：就是指根据不同的条件生成不同的 SQL 语句

    

    MyBatis 采用功能强大的基于 OGNL 的表达式：

    * if
    * choose (when, otherwise)
    * trim (where, set)
    * foreach

    

    SQL：

    ```mysql
    create table `blog` (
        `id` varchar(50) not null comment '博客ID',
        `title` varchar(100) not null comment '博客标题',
        `author` varchar(30) not null comment '博客作者',
        `create_time` datetime not null comment '创建时间',
        `views` int(30) not null comment '浏览量'
    ) engine=InnoDB default charset=utf8;
    ```

    1. ##### if

       ```xml
       <select id="queryBlogIf" parameterType="map" resultType="blog">
           select * from blog
           <where>
               <if test="title != null">
                   title = #{title}
               </if>
               <if test="author != null">
                   and author = #{author}
               </if>
           </where>
       </select>
       ```

    2. ##### choose, when, otherwise

       ```xml
       <select id="queryBlogChoose" parameterType="map" resultType="blog">
           select * from blog
           <where>
               <choose>
                   <when test="title != null">
                       title = #{title}
                   </when>
                   <when test="author != null">
                       and author = #{author}
                   </when>
                   <otherwise>
                       and views > #{views}
                   </otherwise>
               </choose>
           </where>
       </select>
       ```

       

    3. ##### trim (where, set)

       ```xml
       <update id="updateBlog" parameterType="map">
           update blog
           <set>
               <if test="title != null">
                   title = #{title},
               </if>
               <if test="author != null">
                   author = #{author}
               </if>
           </set>
           <where>
               id = #{id}
           </where>
       </update>
       ```

       ```xml
       <trim prefix="" prifixOverrides="" suffix="" suffixOverrides="">
       	...
       </trim>
       
       <!-- 例如 -->
       <trim prefix="WHERE" prifixOverrides="AND|OR">
       	...
       </trim>
       <trim prefix="SET" suffixOverrides=",">
       	...
       </trim>
       ```

       

    4. ##### foreach

       详细解析：https://www.cnblogs.com/-blog/p/5178106.html

       动态 SQL 的另一个常用的操作需求是对一个集合进行遍历，通常是在构建 IN 条件语句的时候

       ```xml
       <select id="selectPostIn" resultType="domain.blog.Post">
       	select *
           from post p
           where id in
           <foreach item="item" index="index" collection="list" open="(" separator="," close=")">
           	#{item}
           </foreach>
       </select>
       ```

       * 当使用可迭代对象或数组时，index 是当前迭代的次数，item 的值是本次迭代获取的元素。
       * 当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

       

    5. ##### SQL 片段

       有的时候，可能会将一些公共的部分抽取出来，方便复用。

       1. 使用 sql 标签抽取公共的部分

          ```xml
          <sql id="if-title-author">
              <if test="title != null">
                  title = #{title}
              </if>
              <if test="author != null">
                  and author = #{author}
              </if>
          </sql>
          ```

       2. 在需要使用的地方，使用 include 标签引用即可

          ```xml
          <select id="queryBlogIf" parameterType="map" resultType="blog">
              select * from blog
              <where>
                  <include refid="if-title-author"></include>
              </where>
          </select>
          ```

       注意事项：

       * 最好基于单表来定义 SQL 片段
       * 抽取的公共部分不要存在 where 标签

       

       建议：先写出完整的 SQL 语句，测试通过后，再去对应地修改成为动态 SQL 实现通用即可。

    
    
16. #### 缓存

    ##### 16.1 简介

    1. 什么是缓存（Cache）？
       * 存在内存中的临时数据。
       * 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库数据文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。
    2. 为什么使用缓存？
       * 减少和数据库的交互次数，减少系统开销，提高系统效率。
    3. 什么样的数据能使用缓存？
       * 经常查询并且不经常改变的数据。

    

    ##### 16.2 MyBatis 缓存

    * MyBatis 包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大地提升查询效率。
    * MyBatis 系统中默认定义了两级缓存：一级缓存和二级缓存
      * 默认情况下，只有一级缓存开启。（SqlSession 级别的缓存，也成为本地缓存）
      * 二级缓存需要手动开启和配置，它是基于 namespace 级别的缓存。
      * 为了提高扩展性，MyBatis 定义了缓存接口 Cache。我们可以通过实现 Cache 接口来自定义二级缓存。
    * 具体情况：
      * 映射语句文件中的所有 select 语句的结果将会被缓存。
      * 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
      * 缓存会使用最近最少使用算法（LRU Least Recently Used）算法来清楚不需要的缓存。
      * 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
      * 缓存会保存列表或对象（无论查询方法返回哪种）的1024个引用。
      * 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。
    * 提示：
      * 缓存只作用域 cache 标签所在地映射文件中的语句。如果混合使用 Java API 和 XML 映射文件，在公用接口中的语句将不会被默认缓存。需要使用 @CacheNamespaceRef 注解指定缓存作用域。

    

    ##### 16.3 一级缓存

    * 一级缓存也叫本地缓存：

      * 与数据库同一次会话期间查询到的数据会放在本地缓存中。
      * 以后如果需要获取相同的数据，直接从缓存中拿，没必要再去查询数据库。

    * 缓存失效的情况：

      1. 查询不同的东西
      2. 增删改操作，可能会改变原来的数据，所以必定会刷新缓存
      3. 查询不同的 Mapper.xml
      4. 手动清理缓存

      ```java
      try (SqlSession sqlSession = MyBatisUtils.getSqlSession()) {
          UserMapper mapper = sqlSession.getMapper(UserMapper.class);
          User user1 = mapper.findUserById(1);
          System.out.println(user1);
      
      //			int num = mapper.updateUser(new User(2, "胆小鬼", 10));
      //			System.out.println(num);
      
          //手动清理缓存
          sqlSession.clearCache();
      
          User user2 = mapper.findUserById(1);
          System.out.println(user2);
          System.out.println(user1 == user2);
      }
      ```

      

    * 小结：

      * 一级缓存默认是开启的，只在一次 SqlSession 中有效，也就是拿到连接到关闭连接这个区间段。
      * 一级缓存就是一个 Map。

    

    ##### 16.4 二级缓存

    * 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存

    * 基于 namespace 级别的缓存，一个名称空间，对应一个二级缓存

    * 工作机制

      * 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
      * 如果当前会话关闭了，这个会话对应的一级缓存就没了。会话关闭了，一级缓存中的数据被保存到二级缓存中
      * 新的会话查询信息，就可以从二级缓存中获取数据
      * 不同的 mapper 查出的数据会放在自己对应的缓存（Map）中

    * 默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：

      ```xml
      <cache/>
      ```

      

    步骤：

    1. 核心配置文件中，全局缓存是默认开启地

       ```xml
       <!-- 显式地开启全局缓存 -->
       <setting name="cacheEnabled" value="true"/>
       ```

    2. 在要使用二级缓存的 Mapper.xml 中开启

       ```xml
       <cache/>
       ```

       也可以自定义参数

       ```xml
       <cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
       ```

    3. 测试

       1. 问题：需要将实体类序列化，否则就会报错

          ```java
          Caused by: java.io.NotSerializableException: com.example.demo06.pojo.User
          ```

       2. 代码

          ```java
          User user1 = null;
          try (SqlSession sqlSession1 = MyBatisUtils.getSqlSession()) {
              UserMapper mapper1 = sqlSession1.getMapper(UserMapper.class);
              user1 = mapper1.findUserById(1);
              System.out.println(user1);
          }
          try (SqlSession sqlSession2 = MyBatisUtils.getSqlSession()) {
              UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
              User user2 = mapper2.findUserById(1);
              System.out.println(user2);
              System.out.println(user1 == user2);
          }
          ```

       

    小结：
    
    * 只要开启了二级缓存，在同一个 Mapper 下就有效
    * 所有数据都会先放在一级缓存中
    * 只有当会话提交或者关闭的时候，才会提交到二级缓存中
