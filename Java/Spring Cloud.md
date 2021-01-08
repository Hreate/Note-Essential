### Ribbon

1. 基于 HTTP 的客户端均衡负载器。

2. @LoadBalanced 注解可以让 RestTemplate 在请求时开启负载均衡：

   ```java
   @Bean
   @LoadBalanced //开启负载均衡的功能
   RestTemplate restTemplate() {
       return new RestTemplate();
   }
   ```

3. LoadBalancerClient 接口对象可以调用 choose 方法，通过负载均衡选择获取服务的实例 ServiceInstance。

4. DiscoveryClient 接口对象可以调用 getInstances 方法，获取服务地址的 list。



### Feign

1. 轻量级的 rest 客户端，集成了 Ribbon。

2. 示例：

   ```java
   // 第一种写法
   @FeignClient("user") // 调用的服务名
   @RequestMapping("/user")
   public interface UserFeign {
       @GetMapping("/{id}")
       Result findById(@PathVariable String id);
   }
   
   // 第二种写法
   @FeignClient(name = "user", path = "/user")
   public interface UserFeign {
       @GetMapping("/{id}")
       Result findById(@PathVariable String id);
   }
   ```

3. 消费者的启动类上加 @EnableFeignClients 注解

   ```java
   @EnableFeignClients(basePackages = {"com.easybuy.user.feign"}) // 注明包路径
   ```

   

4. 注意点：

   * FeignClient 接口有参数时，@PathVariable("xxx") 和 @RequestParam("xxx") 注解括号内必须把参数名注明。
   * FeignClient 返回值为复杂对象时其值类型必须有无参构造函数。