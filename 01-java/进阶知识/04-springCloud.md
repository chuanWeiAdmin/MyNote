### 一.RestTemplate的使用

1. 创建配置类

   ```java
   @Configuration
   public class ApplicationContextConfig {
       @Bean
       @LoadBalanced
       public RestTemplate getRestTemplate() {
           return new RestTemplate();
       }
   }
   ```
   
2. 通过方法调用分布式方法

   - 通过 getForObject (url, requestMap, ResponseBean.class)方法
   - 这三个参数分别代表 **REST请求地址**、**请求参数**、**HTTP响应转换被转换成的对象类型**

   ```java
   @RestController
   @Slf4j
   public class OrderZKController {
       public static final String INVOKE_URL = "http://cloud-provider-payment";
   
       //注入对象
       @Resource
       private RestTemplate restTemplate;
   
       @GetMapping(value = "/consumer/payment/zk")
       public String paymentInfo() {
           String result = 
               restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
           return result;
       }
   }
   
   ```



## 二.服务注册与发现

### 1.Eureka服务注册与发现

- ##### **服务治理**

  Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务治理，在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。 

- ##### **服务注册与发现**：

  Eureka采用了CS的设计架构，

  Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。在服务注册与发现中，有一个注册中心。**当服务器启动的时候，会把当前自己服务器的信息 比如 服务地址通讯地址等以别名方式注册到注册中心上**。**另一方（消费者|服务提供者），*以该别名的方式*去注册中心上获取到实际的服务通讯地址**，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))

- ##### pom文件

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

- ##### yaml配置

  ```yaml
  server:
    port: 7001
  
  
  eureka:
    instance:
      hostname: eureka7001.com #eureka服务端的实例名称
    client:
      register-with-eureka: false     #false表示不向注册中心注册自己。
      #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      fetch-registry: false     
      
      service-url:
      #集群指向其它eureka
        #defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
        defaultZone: http://eureka7001.com:7001/eureka/
    #server:
      #关闭自我保护机制，保证不可用服务被及时踢除
      #enable-self-preservation: false
      #eviction-interval-timer-in-ms: 2000
  ```

- ##### 将生产者注册到Eureka

  1. 添加pom依赖

     ```xml
     <!--eureka-client-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
     </dependency>
     ```

  2. 更改yaml文件

     ```yaml
     eureka:
       client:
         #表示是否将自己注册进EurekaServer默认为true。
         register-with-eureka: true
         #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
         fetchRegistry: true
         service-url:
           #单机版
           defaultZone: http://localhost:7001/eureka
           # 集群版
           #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
       instance:
           instance-id: payment8001
           #访问路径可以显示IP地址
           prefer-ip-address: true
           #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
           #lease-renewal-interval-in-seconds: 1
           #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
           #lease-expiration-duration-in-seconds: 2
     
     ```

  3. **在主启动类中添加注解**

     ```java
     @SpringBootApplication
     @EnableEurekaClient
     public class PaymentMain8001
     {
         public static void main(String[] args) {
             SpringApplication.run(PaymentMain8001.class, args);
         }
     }
     ```

- ##### 将消费者注册到 Eureka 中

  **同服务者相同**

- ##### **Eureka 搭建服务端集群**

  相互调用相互守望

  ```yaml
  eureka:
    instance:
      hostname: eureka7001.com #eureka服务端的实例名称
    client:
      register-with-eureka: false     #false表示不向注册中心注册自己。
      #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      fetch-registry: false    
      
      service-url:
      #集群指向其它eureka
        #defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
        defaultZone: http://eureka7001.com:7001/eureka/
  ```

- ##### 使用生产者集群的时候出现的问题

  使用 @Bean 标签的时候RestTemplate不具备负载均衡的能力，所以会报错

  要在*配置类*中使用**@LoadBalanced注解赋予RestTemplate负载均衡的能力**

  ```java
  @Configuration
  public class ApplicationContextConfig {
      //@Bean
      @LoadBalanced //使用注解使 RestTemplate 具有负载均衡的能力
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

- ##### actuator 微服务信息完善

  1. 主机名称:服务名称的修改，让Eureka上不显示主机名称

     ```yaml
     eureka:
       client:
         register-with-eureka: true
         fetchRegistry: true
         service-url:
           #单机版
           defaultZone: http://localhost:7001/eureka
       instance: #让Eureka现在服务名
           instance-id: payment800
     ```

  2. 访问路径有ip提示

     ```yaml
     eureka:
       client:
         register-with-eureka: true
         fetchRegistry: true
         service-url:
           #单机版
           defaultZone: http://localhost:7001/eureka
       instance:
           instance-id: payment8001
           #访问路径可以显示IP地址
           prefer-ip-address: true
     ```

- ##### 服务发现 Discovery 

  1. 修改controller

     ```java
     @RestController
     @Slf4j
     public class PaymentController {
         @Resource
         private DiscoveryClient discoveryClient;
     
         @GetMapping(value = "/payment/discovery")
         public Object discovery() {
             //获得 全部的服务
             List<String> services = discoveryClient.getServices();
             for (String element : services) {
                 log.info("*****element: " + element);
             }
     
             //获取 其中一个服务的全部实例
             List<ServiceInstance> instances = 
                 discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
             for (ServiceInstance instance : instances) {
                 log.info(instance.getServiceId()
                         + "\t" + instance.getHost()
                         + "\t" + instance.getPort()
                         + "\t" + instance.getUri());
             }
     
             return this.discoveryClient;
         }
     }
     ```

  2. 修改启动类

     ```java
     @SpringBootApplication
     @EnableEurekaClient		//这个注解之后可能不常用了
     @EnableDiscoveryClient	//这个注解之后还会用到***************
     public class PaymentMain8001
     {
         public static void main(String[] args) {
             SpringApplication.run(PaymentMain8001.class, args);
         }
     }
     ```

### 2.Zookeeper服务注册与发现

- 服务提供者添加进注册中心

  1. 修改pom文件

     **注意：如果zookeeper的版本和服务器上的版本不同会报错，并且springboot会自带一个zookeeper，要先排除这个自带的**

     ```xml
     <!-- SpringBoot整合zookeeper客户端 -->
     <dependency>
     	<!--springBoot中会带一个zookeeper如果同注册中心的版本不符会报错-->
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
         <!--所以先排除自带的zookeeper3.5.3-->
         <exclusions>
             <exclusion>
                 <groupId>org.apache.zookeeper</groupId>
                 <artifactId>zookeeper</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
     <!--添加同服务器版本相同的zookeeper3.4.9版本-->
     <dependency>
         <groupId>org.apache.zookeeper</groupId>
         <artifactId>zookeeper</artifactId>
         <version>3.4.9</version>
     </dependency>
     ```

  2. 修改yaml文件

     ```yaml
     #8004表示注册到zookeeper服务器的支付服务提供者端口号
     server:
       port: 8004
     #服务别名----注册zookeeper到注册中心名称
     spring:
       application:
         name: cloud-provider-payment  #会在zookeeper中显示的名字
       cloud:
         zookeeper:
           connect-string: 192.168.111.144:2181  #2181 zookeeper的默认端口
     ```

  3. 设置主启动类

     ```java
     @SpringBootApplication
     //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
     @EnableDiscoveryClient
     public class PaymentMain8004
     {
         public static void main(String[] args) {
                 SpringApplication.run(PaymentMain8004.class, args);
         }
     }
     ```

- 服务消费者注册近zookeeper

  1. 修改 yaml 文件

     **同上**

  2. 修改pom文件

     **同上**

  3. 设置主启动类

     **同上**

  4. 设置RestTemplate的配置文件

     ```java
     @Configuration
     public class ApplicationContextConfig{
         @Bean
         @LoadBalanced
         public RestTemplate getRestTemplate(){
             return new RestTemplate();
         }
     }
     ```

### 3.consul 的注册和发现

- 微服务注册进consul

  1.  加pom

     ```xml
     <!--SpringCloud consul-server -->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-consul-discovery</artifactId>
     </dependency>
     ```

  2. 改yaml

     ```yaml
     ###微服务的端口号
     server:
       port: 8006
     
     spring:
       application:
         name: consul-provider-payment
     ####consul注册中心地址
       cloud:
         consul:
           host: localhost
           port: 8500
           discovery:
             #hostname: 127.0.0.1
             service-name: ${spring.application.name}
     ```

  3. 主启动类

     **同上**

- 将服务消费者注册进 consul

  **同上**

## 三.负载均衡

### 1.Ribbon 实现负载均衡

- **Ribbon 是在客户端的一种负载均衡，软负载均衡**
- *Ribbon 就是（负载均衡 + RestTemplate 调用）*
- 

### 2.Ribbon 核心组件 IRule

- 可以根据特定的算法从服务列表中选取一个要访问的服务（7种模式）

  |      | 模式                                    | 简介                                                         |
  | ---- | --------------------------------------- | :----------------------------------------------------------- |
  | 1    | com.netflix.loadbalancer.RoundRobinRule | 轮询                                                         |
  | 2    | com.netflix.loadbalancer.RandomRule     | 随机                                                         |
  | 3    | com.netflix.loadbalancer.RetryRule      | 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务 |
  | 4    | WeightedResponseTimeRule                | 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择 |
  | 5    | BestAvailableRule                       | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务 |
  | 6    | AvailabilityFilteringRule               | 先过滤掉故障实例，再选择并发较小的实例                       |
  | 7    | ZoneAvoidanceRule                       | 默认规则,复合判断server所在区域的性能和server的可用性选择服务器 |

- 实现方法

  1. 配置类不能同 @ComponentScan 即 @SpringBootApplication 同包获子包下

  2. 在同包上一层新建一个包存放 IRule 的配置类

     ```java
     @Configuration
     public class MySelfRule {
         @Bean
         public IRule myRule() {
             //定义为随机，一共有 7 种类型
             return new RandomRule();
         }
     }
     ```

  3. RestTemplate 的配置类

     同上 第一 节 的RestTemplate 配置类

     ```java
     @Configuration
     public class ApplicationContextConfig {
         @Bean
         //@LoadBalanced
         public RestTemplate getRestTemplate() {
             return new RestTemplate();
         }
     }
     ```

  4. 主启动类添加注解

     @RibbonClient(name = "要访问的服务名",configuration=自定义的配置类.class)

     -  name  要访问的服务名
     -  configuration 自定义的配置类

     ```java
     @SpringBootApplication
     @EnableEurekaClient
     @RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)
     public class OrderMain80
     {
         public static void main(String[] args) {
                 SpringApplication.run(OrderMain80.class, args);
         }
     }
     ```


### 3.OpenFeign服务接口调用

**前言：Feign（*用着消费端*）是一个声明式的Web服务客户端，让编写Web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可**

解释前面的话:服务提供者有哪些接口，就在Feign中定义哪些接口

*注：Feign已经停止更新*

#### 1.pom文件

```xml
<dependency>
    <!-- 1.添加Feign -->
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 2.主启动类

```java
@SpringBootApplication
@EnableFeignClients //******2.请注意 这里是激活 feign
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class, args);
    }
}
```

#### 3.Feign 接口类

```java
@Component  // 将类交给容器，************* 一定不要忘了 ************
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")  // *******3.使用
public interface PaymentFeignService {
    @GetMapping(value = "/payment/get/{id}")  //好像是要写
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
    
	@GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout();
}
```

#### 4.Controller层代码

```java
@RestController
@Slf4j
public class OrderFeignController {
    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
        return paymentFeignService.getPaymentById(id);
    }
    
    @GetMapping(value = "/consumer/payment/feign/timeout")
    public String paymentFeignTimeout() {
        // OpenFeign客户端一般默认等待1秒钟
        return paymentFeignService.paymentFeignTimeout();
    }
}
```





## 四.服务降级，熔断，限流

### 1.Hystrix（豪猪哥）

**前言**： Hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，***Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性***。

#### 1.1.三个重要概念

- 服务**降级**：服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback--**相当于if else中的else**
- 服务**熔断**：类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示
- 服务**限流**：秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

**注：什么情况下回出现服务降级的情况**

1. 程序运行异常
2. 超时
3. 服务熔断触发服务降级
4. 线程池/信号量打满也会导致服务降级

**注2：服务熔断会触发服务降级**

#### 1.2.构建

##### 1.2.1 pom文件

```xml
<!--hystrix-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

#### 1.3 服务降级

注解 : @HystrixCommand

##### 1.3.1 业务类

- 在业务方法上添加注解 HystrixCommand
- 创建一个用来报错执行的方法(兜底的方法)

```java
@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
                    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000")
})
public String paymentInfo_TimeOut(Integer id) {
    try {
        TimeUnit.MILLISECONDS.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "线程池:  " + Thread.currentThread().getName() + " id:  " + id +
        "\t" + "O(∩_∩)O哈哈~" + "  耗时(秒): ";
}
// 服务器超时或者报错 ，用来显示的方法
public String paymentInfo_TimeOutHandler(Integer id) {
    return "线程池:  " + Thread.currentThread().getName() + "  8001系统繁忙或者运行报错，请稍后再试,id:  " + id +
        "\t" + "o(╥﹏╥)o";
}	
```

注：HystrixCommand 内容介绍

- fallbackMethod：放方法报错或超时执行的方法
- commandProperties



##### 1.3.2 主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker		//表示启用 Hystrix 
public class PaymentHystrixMain8001{
    public static void main(String[] args) {
            SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}

```

##### 1.3.3 全局跳转

- 使用注解 @DefaultProperties(defaultFallback = "方法名")

  ```java
  @RestController
  //这个标签设置全局的异常处理方法
  @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
  public class OrderHystirxController {
      //这个方法就不会走全局异常处理，正常了就是正常，异常了就是异常
      @GetMapping("/consumer/payment/hystrix/ok/{id}")
      public String paymentInfo_OK(@PathVariable("id") Integer id) {
          return "id"+id;
      }
  
      @GetMapping("/consumer/payment/hystrix/timeout/{id}")
      @HystrixCommand		//留着这个的作用就是在服务方法异常的时候走默认的处理方法
      public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
          int age = 10 / 0;
          return "id"+id;
      }
  
      // 下面是全局fallback方法
      public String payment_Global_FallbackMethod() {
          return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
      }
  }
  ```

  ##### 1.3.4 通过Feign实现更优雅的解决方式

  ###### 1.yml文件

  ```yaml
  # 开启Feign对Hystrix 的支持还是什么的
  feign:
    hystrix:
      enabled: true
  ```

  ###### 2.Feign接口

  - 通过 @FeignClient 注解中的 fallback属性，
  - fallback属性中指定的是Feign接口的实现类

  ```java
  @Component
  @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT", fallback = PaymentFallbackService.class)
  //fallback中的值是接口的实现类
  public interface PaymentHystrixService {
      @GetMapping("/payment/hystrix/ok/{id}")
      public String paymentInfo_OK(@PathVariable("id") Integer id);
  }
  ```

  ###### 3.Feign接口的实现类

  ```java
  @Component  //这个东西一定不能忘了，要将类交给容器创建
  public class PaymentFallbackService implements PaymentHystrixService {
      // 当 Feign 接口中的 paymentInfo_OK 方法出现异常就会转到当前方法中，实现服务熔断 
      @Override
      public String paymentInfo_OK(Integer id) {
          return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
      }
  }
  ```



#### 1.4 服务熔断

##### 1.4.1 何为熔断

 **熔断机制概述**：熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。**当检测到该节点微服务调用响应正常后，恢复调用链路**。

 在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，**缺省是5秒内20次调用失败**，就会启动熔断机制。熔断机制的注解是**@HystrixCommand。**   



##### 1.4.2 配置服务熔断

```java

@Service
public class PaymentService {
	//配置服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),// 是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),// 请求次数
			// 时间窗口期
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), 
			// 失败率达到多少后跳闸
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"),
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if (id < 0) {
            throw new RuntimeException("******id 不能负数");
        }
        String serialNumber = IdUtil.simpleUUID();
        return Thread.currentThread().getName() + "\t" + "调用成功，流水号: " + serialNumber;
    }

    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " + id;
    }

}
```

##### 1.4.3 总结

![](.\..\..\99-资源\服务熔断.png)

- 熔断类型
  - 熔断打开:请求不再进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间)，当打开时长达到所设时钟则进入半熔断状态
  - 熔断关闭:熔断关闭不会对服务进行熔断
  - 熔断半开:部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

##### 1.4.4 全部的配置

```java
@HystrixCommand(fallbackMethod = "str_fallbackMethod", groupKey = "strGroupCommand", commandKey = "strCommand", threadPoolKey = "strThreadPool", commandProperties = {
        // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
        @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
        // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
        @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
        // 配置命令执行的超时时间
        @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
        // 是否启用超时时间
        @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
        // 执行超时的时候是否中断
        @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
        // 执行被取消的时候是否中断
        @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
        // 允许回调方法执行的最大并发数
        @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
        // 服务降级是否启用，是否执行回调函数
        @HystrixProperty(name = "fallback.enabled", value = "true"),
        // 是否启用断路器
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
        // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
        // 如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
        // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过
        // circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50,
        // 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
        // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，
        // 会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，
        // 如果成功就设置为 "关闭" 状态。
        @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
        // 断路器强制打开
        @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
        // 断路器强制关闭
        @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
        // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
        @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
        // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据
        // 设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
        // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
        @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
        // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
        @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
        // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
        @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
        // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
        @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
        // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
        // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
        // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
        @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
        // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
        @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
        // 是否开启请求缓存
        @HystrixProperty(name = "requestCache.enabled", value = "true"),
        // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
        @HystrixProperty(name = "requestLog.enabled", value = "true"),
}, threadPoolProperties = {
        // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
        @HystrixProperty(name = "coreSize", value = "10"),
        // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，
        // 否则将使用 LinkedBlockingQueue 实现的队列。
        @HystrixProperty(name = "maxQueueSize", value = "-1"),
        // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
        // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue
        // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
        @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
})
public String strConsumer() {
    return "hello 2020";
}

public String str_fallbackMethod() {
    return "*****fall back str_fallbackMethod";
}
```





## 五:网关

### 1.新一代网关 getway

***注意：请检查一下你的依赖中是否含有spring-boot-starter-web，如果有，请干掉它。因为我们的SpringCloud Gateway是一个netty+webflux实现的web服务器，和Springboot Web本身就是冲突的。***

#### 1.1 三大核心概念

- 路由(Route)：路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由
- 断言(Predicate)：参考的是Java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，如果请求与断言相匹配则进行路由
- 过滤(Filter)：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

#### 1.2 构建项目

##### 1.2.1 pom文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

##### 1.2.2 yml文件

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
       routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

##### 1.2.3 主启动类

```java
@SpringBootApplication
@EnableEurekaClient  //同样适用了Eureka ,getway就是个微服务同样注册进Eureka
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class, args);
    }
}
```



#### 1.3 getway配置网关的两种方法

##### 1.3.1 yml文件配

见前文 构建项目中的yml文件

##### 1.3.2 编码配置

```java
@Configuration  //只要是配置就要加上这个注解  不要忘了
public class GateWayConfig {
    @Bean  //同上
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        // 相当于访问 localhost:9527/guone 会转发到 http://news.baidu.com/guonei
        routes.route("path_route_atguigu",
                     r -> r.path("/guonei")
                     .uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }
}
```

#### 1.4 动态路由配置

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:	#开启从注册中心创建路由的功能
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

```

#####  1.4.x gateway中uri配置方式

在gateway中配置uri配置有三种方式，包括

- 第一种：ws(websocket)方式: uri: ws://localhost:9000
- 第二种：http方式: uri: http://localhost:8130/
- 第三种：lb(注册中心中服务名字)方式: uri: lb://brilliance-consumer

#### 1.5 Predicate的使用

一共有十多种，百度用到哪个查哪个

#### 1.6 Filter的使用

##### 1.6.1 常用的GetWayFilte

看官网 或 百度

##### 1.6.2 自定义过滤器

- 主要是实现两个接口 GlobalFilter, Ordered

```java
@Component //一定要加这个注解 将类交给容器
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***********come in MyLogGateWayFilter:  " + new Date());

        String uname = exchange.getRequest().getQueryParams().getFirst("uname");

        if (uname == null) {
            log.info("*******用户名为null，非法用户，o(╥﹏╥)o");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //加载过滤器的顺序 ， 数字越小越先加载
        return 0;
    }
}
```



## 六.服务配置 服务总线

### 1 Spring Config 配置中心搭建

#### 1.1 服务端

注：要将

##### 1.1.1 pom 文件 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

##### 1.1.2 yml 文件

注：要使用 文件名为  application.yml

- application.yml  系统层面的配置文件 （**优先加载**）
- bootstrap.yml  用户层面的配置文件

```yaml
spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          #自己git仓库上地址
          uri: git@github.com:zzyybs/springcloud-config.git #GitHub上面的git仓库名字
		  ##搜索目录--仓库下面的项目名
          search-paths:
            - springcloud-config
      ####读取分支
      label: master
```

##### 1.1.3 启动类

```java
@SpringBootApplication
@EnableConfigServer    //SpringConfig 的开启注解
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

##### 1.1.4 访问规则

http://config-3344.com:3344/master/config-dev.yml

- config-3344.com  ： 相当于 localhost---如果没有配置host文件的话
- master ：分支
- config-dev.yml ： 文件名

#### 1.2 客户端

##### 1.2.1 pom文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <!-- 使用spring-cloud-starter-config -->
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

##### 1.2.2 yml文件

```yaml
spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   
      #上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      
      uri: http://localhost:3344 #配置中心地址k
```

##### 1.2.3 控制层类

```java
@RestController
public class ConfigClientController {
    // 访问到的文件里面的配置内容
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

#### 1.3 存在问题

当在github中更改配置的时候，**服务端**可以访问到更改后的数据但是**客户端**不能访问到已经更改的数据

#### 1.4 解决方案-手动版的自动刷新

##### 1.4.1 pom模块引入actuator监控

注：可以所有模块都加上  **网关不需要**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

##### 1.4.2 修改YML，暴露监控端口

**注：有这种暴露端口什么东西的一定要加actuator**

```yaml
# 暴露监控端点 有这种暴露端口什么东西的一定要加 actuator
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

##### 1.4.3 用@RefreshScope修饰业务类Controller修改

```java
@RestController
@RefreshScope  //一定要加这个注解 ，没有这个注解不能刷新
public class ConfigClientController {
    // 访问到的文件里面的配置内容
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

##### 1.4.4 手动使用post访问客户端

可以使用工具发送 post 请求  url 为   http://localhost:3355/actuator/refresh  

也可以使用doc命令发送请求

curl -X POST "http://localhost:3355/actuator/refresh"

**注：如果想实现动态刷新请看下一节**

### 2.SpringCloud Bus 消息总线

#### 2.1 设计思想

- 由消息总线通知客户端A，再由客户端A通知其他客户端
- 由消息总线通知服务端，由服务端通知其他客户端

#### 2.2 构建项目

注 ： 一定要有一个可以正常运行的RebbitMQ 的运行环境

另外 SpringCloud Bus 还支持 kafka

##### 2.2.1 服务端 更改配置

- 改pom文件

  ```xml
  <!--添加消息总线RabbitMQ支持-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  ```

- 改服务端的 application.yml 

  ```yaml
  #注意 服务端的 rabbitmq 要顶头写
  #rabbitmq相关配置
  rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
  
  #注意暴露端点一定要加 actuator
  ##rabbitmq相关配置,暴露bus刷新配置的端点
  management:
    endpoints: #暴露bus刷新配置的端点
      web:
        exposure:
          include: 'bus-refresh'  
          #这里为什么是 bus-refresh , 社会上的事少打听 知道太多对你没好处
  ```

  - **注意1：服务端的 rabbitmq 要顶头写**
  - **注意2：暴露端点一定要加 actuator**

##### 2.2.2 客户端 更改配置

- pom 文件修改

  ```xml
  <!--添加消息总线RabbitMQ支持--> <!-- 同上 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  ```

- 服务端的 bootstrap.yml 修改

  ```yaml
  spring:
    application:
      name: config-client
    cloud:
      #Config客户端配置
      config:
      ...
    # 特别注意 这个地方的 rabbitmq 是在spring下面的 一定要记住
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
      
  # 暴露监控端点
  management:
    endpoints:
      web:
        exposure:
          include: "*"    
  ```

  - 特别注意：服务端的 rabbitmq 一定要是在spring下面的，不要写错了

##### 2.2.3 修改配置文件并发送post请求

curl -X POST "http://localhost:3344/actuator/bus-refresh"

**一次发送处处生效**



#### 2.3 动态刷新的定点通知

curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"

- 同上有区别  这个只通知了 3355  

- 通知的公式  http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}

- {destination} 目的地是 微服务+端口号==config-client:3355

  ```yaml
  server:
    port: 3355
  
  spring:
    application:
      name: config-client
  ```

  

## 七.消息驱动 

**SpringCloud Stream**

### 1.消息驱动概述

#### 1.1 是什么

- 为了解决同一个项目下存在多个MQ的情况
- 屏蔽底层消息中间件的差异,降低切换成本，统一消息的编程模型
- 目前只支持  Rabbit MQ 和 Kafka

1.2 编码 API 和常用的注解

| 组成            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前只支持Rabbit MQ 和 Kafka                         |
| Binder          | Binder是应用与消息中间件之间的封装，目前事项了Rabbit MQ 和 Kafka，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应Kafka的topic，Rabbit MQ的exchange)，这些都是通过配置文件来实现的 |
| @Input          | 注解标识输入通道，通过该输入通道接收到的消息进入应用程序     |
| @Output         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
| @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
| @EnableBinding  | 指定信道channel和exchange绑定在一起                          |



### 2.生产者的构建

#### 2.1 pom 文件

```xml
<!-- stream  整合 rabbit 如果是Kafka 那就将下面的东西换成Kafka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

#### 2.2 yml文件

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          # 消息的生产者用这个 output 
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

#### 2.3主启动

和普通的启动类都一样，没有区别

#### 2.4 控制类

也没有区别

#### 2.5 业务类

**注：重点在这里**

- IMessageProvider：自己定义的一个接口，不用太过在意，**(懒的写了，知道这个就行)**
- 发送消息的方法 output.send(MessageBuilder.withPayload("消息").build());
- Source.class 生产者的话就再EnableBinding 添加 Source.class 表示建立一个通道

```java
// 重点 这里不再使用 @Service 而是使用 Stream 的注解
@EnableBinding(Source.class) //定义消息的推送管道，注意Source别引错包了
public class MessageProviderImpl implements IMessageProvider {
    @Resource
    private MessageChannel output; // 消息发送管道

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("*****serial: " + serial);
        return null;
    }
}
```



### 3.消息驱动 消费者

#### 3.1 pom 文件

同上没有变化，都是一个pom文件

#### 3.2 yml 文件

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          # 消费者就使用 input 这个
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

#### 3.3 主启动类

同之前一样，没有区别

#### 3.4 控制类

```java
@Component
@EnableBinding(Sink.class) // 消费者就要用 Sink
public class ReceiveMessageListenerController {
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("消费者1号,----->接受到的消息: " + message.getPayload() + "\t  "+                   							"port: " + serverPort);
    }
}
```

**当请求输出端的时候，在输入端就会展示出相对应的数据**



### 4.分组消费与持久化

#### 4.1 出现的问题

- 存在重复消费的问题
- 消息持久化的问题

#### 4.2 分组解决重复消费的问题

*注意在Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。*
**不同组是可以全面消费的(重复消费)，**
**同一组内会发生竞争关系，只有其中一个可以消费。**

- 使用配置文件分组

  ```yaml
  spring:
    application:
      name: cloud-stream-consumer
    cloud:
        stream:
          binders: # 在此处配置要绑定的rabbitmq的服务信息；
            defaultRabbit: # 表示定义的名称，用于于binding整合
             #..... 同上的配置，这里就不写了
          bindings: # 服务的整合处理
            input: # 这个名字是一个通道的名称
              destination: studyExchange # 表示要使用的Exchange名称定义
              content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
              binder: defaultRabbit # 设置要绑定的消息服务的具体设置
              # 通过 group 分组 后面的是分组的名字
              group: atguiguA
  ```

#### 4.3 分组解决持久化问题

**已经分组的消费端就支持持久化**



## 八.分布式请求链路跟踪

### 1 概述

 在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

**解决方案 Sleuth+zipkin 实现请求链路的跟踪 **



### 2 搭建

**前情提要 ：要先下载搭建zipkin环境**

#### 2.1 pom文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

#### 2.1 yaml 文件

```yaml
spring:
  application:
    name: cloud-payment-service
  zipkin:
      base-url: http://localhost:9411
  sleuth:
    sampler:
    #采样率值介于 0 到 1 之间，1 则表示全部采集
    probability: 1
```



### 3 查询链路过程

打开浏览器访问：http://localhost:9411  查看链路过程



## 九.Nacos 服务注册和配置中心

注：要想使用Nacos一定要先下载运行 

**使用docker搭建Nacos环境 https://www.cnblogs.com/wqp001/p/15329529.html**

docker run -d --name nacos -p 8848:8848 -e PREFER_HOST_MODE=hostname -e MODE=standalone nacos/nacos-server



### 1.Nacos作为服务注册中心

#### 1.1 服务提供者

1.1.1 pom文件

- 父pom

  ```xml
  <!-- 引入spring cloud alibaba -->
  <dependency>
  	<groupId>com.alibaba.cloud</groupId>
  	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
  	<version>2.2.6.RELEASE</version>
  	<type>pom</type>
  	<scope>import</scope>
  </dependency>
  ```

- 本模块的 pom

  ```xml
  <!--SpringCloud ailibaba nacos -->
  <dependency>
  	<groupId>com.alibaba.cloud</groupId>
  	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```
  
- 注：nacos 配置的版本一定要和 springboot的版本对应

  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.2.6.RELEASE</version>  <!-- 这个地方一定要和上面对应 -->
  </parent>
  <dependencyManagement>
      <dependencies>
          <!-- 声明 springCloud Alibaba 版本 -->
          <dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-alibaba-dependencies</artifactId>
              <version>2.2.6.RELEASE</version>   <!-- 这个地方一定要和上面对应 -->
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```

1.1.2 yml

```yaml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider   #这个就是在nacos上的地址
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址
# 将端口暴露出去
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

1.1.3 主启动类

```java
@EnableDiscoveryClient  //使用这个注解开启将服务添加到注册中心
@SpringBootApplication
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class, args);
    }
}
```

1.1.4 业务类 

没有什么特殊的，就是普普通通的业务类

```java
@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: " + serverPort + "\t id" + id;
    }
}
```



#### 1.2 服务消费者

注：使用远程调用一定是 RestTemplate 结合 Ribbon 就一定会有一个配置类，配置类中一定会有一个@LoadBalanced注解

1.2.1 pom 文件

**同提供者 没有任何区别**

1.2.2 yaml 文件

```yaml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url: #目的是在业务类中使用@Value将下面这个值注入，调用地址写在配置文件中
  nacos-user-service: http://nacos-payment-provider
```

1.2.3 主启动类

没有区别

1.2.4 配置类

注：

- **直接写一个配置文件可能会遇到不能注入的情况，可以将方法放到启动类中**
- 未完待续 .....

```java
@Configuration  //是配置类一定要加这个注解
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced  //RestTemplate 结合 Ribbon一定要加这个注解
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

1.2.5 业务类

```java
@RestController
public class OrderNacosController {
    @Resource
    private RestTemplate restTemplate;

    //通过配置文件注入的数据
    @Value("${service-url.nacos-user-service}")
    private String serverURL; 
    //该数据为 服务提供者在注册中心的名字，即spring.application.name: 服务名称

    @GetMapping(value = "/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject(serverURL + "/payment/nacos/" + id, String.class);
    }

}
```

### 2.Nacos 作为配置中心

**注意 ：nacos作为配置中心 主要的配置一定要写在 bootstrap.yml 中**

#### 2.1 基础配置

2.1.1 pom 文件

**只要是跟Nacos集中配置相关的就将下面两个加上**

```xml
<!--nacos-config-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--nacos-discovery-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2.1.2 yml 文件

- application.yml

  ```yaml
  spring:
    profiles:
      active: dev # 表示开发环境
      #active: test # 表示测试环境
      #active: info
  ```

- bootstrap.yml

  ```yaml
  # nacos配置
  server:
    port: 3377
  
  spring:
    application:
      name: nacos-config-client
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848 #Nacos服务注册中心地址
        config:
          server-addr: localhost:8848 #Nacos作为配置中心地址
          file-extension: yaml #指定yaml格式的配置,注意 这里写的是什么在Nacos中一定要写的是什么不能写错了
          
  ```

- 为什么会有两个配置文件(其实只有一个也可以)

  - Nacos同springcloud-config一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。
  - springboot中配置文件的加载是存在优先级顺序的，**bootstrap**优先级高于**application**

2.1.3 主启动类

同其他的没有区别

2.1.4  业务类

同spring config 没有区别，需要注意的是要加一个  @RefreshScope  让代码支持Nacos的动态刷新

```java
@RestController
@RefreshScope //支持Nacos的动态刷新功能。
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

2.1.5 Nacos中的匹配规则

 **最后公式**：${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}

- 对应上面的DataId nacos-config-client-dev.yaml
- 其中：spring.profiles.active 可以没有，最好加上，要不然会有莫名其妙的问题（不是必须有的的东西）

![](.\..\..\99-资源\Nacos 配置中心配置规则.bmp)



#### 2.2 分类配置

2.2.1 DataID方案

2.2.2 Group方案

2.2.3 Namespace方案

```yaml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 121.4.184.80:8848 #Nacos服务注册中心地址
      config:
        server-addr: 121.4.184.80:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP  #指定组别
        namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4 #指定命名空间的id
```



### 3 Nacos的集群和持久化配置

别多哔哔，直接百度





## 十.Sentinel实现熔断与限流

### 1 初始化演示工程

#### 1.1 pom文件

```xml
<dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    <groupId>com.atguigu.springcloud</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>${project.version}</version>
</dependency>
<!--SpringCloud ailibaba nacos -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

#### 1.2 yaml 文件

```yaml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 121.4.184.80:2222 #Nacos服务注册中心地址
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: 121.4.184.80:8080 #配置Sentinel dashboard地址
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719
```

#### 1.3 主启动类

```java
@EnableDiscoveryClient
@SpringBootApplication
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class, args);
    }
}
```

#### 1.4 注

- sentinel 使用的是懒加载，要访问一次才能在页面上显示出来



### 2 流控规则

#### 2.1 基本介绍

- 资源名：唯一名称，请求路径（uri）
- 针对来源：sentinel可以针对调用者进行限流，默认default（不区分来源）
- 阈值类型/单机阈值
  - QPS（每秒的请求数量）：当 api 的QPS达到阈值的时候，进行限流
  - 线程数：当调用该 api 的线程数达到阈值的时候，进行限流
- 是否集群：不需要集群
- 流控模式：
  - 直接：api 达到限流条件时，直接限流
  - 关联：当关联的资源达到阈值的时，就限流自己
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果到达阈值，就进行限流）【api级别只针对来源】
- 流控效果：
  - 快速失败：直接失败，抛异常
  - WarmUp ：根据codeFactor（冷加载因子，默认是3）的值，从阈值/codeFactor ，经过预热时长，才打到设置的QPS阈值
  - 排队等待：匀速排队，请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效

### 3 降级规则

注：sentinel 没有半开状态

#### 3.1 基本介绍

![](.\..\..\99-资源\降级示例.bmp)

- RT （平均响应时间，秒级）
  - 平均响应时间 **超出阈值** 且 **在时间窗口内通过的请求>=5** ,两个条件同时满足后触发降级
  - 窗口期过后关闭断路器
  - RT最大 4900（更大的需要通过 -Dcsp.sentinel.statistic.max.rt=XXXX才能生效）
- 异常比例（秒级）
  - QPS >= 5 且异常比例（秒级统计）超过阈值时，触发降级；时间窗口结束后，关闭降级
- 异常数（分钟级）
  - 异常数（分钟统计）超过阈值时，触发降级，时间窗口结束后，关闭降级



### 4 热点 key 限流

#### 4.1代码

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey", blockHandler = "deal_testHotKey")
// 上面 value 中的值就是要放在页面上资源名的值
// blockHandler 表达异常的时候要走的错误方法
public String testHotKey(@RequestParam(value = "p1", required = false) String p1,
                         @RequestParam(value = "p2", required = false) String p2) {
    //int age = 10/0;
    return "------testHotKey";
}
// 前边参数同方法一样 ， 后面加一个 BlockException
public String deal_testHotKey(String p1, String p2, BlockException exception) {

    return  "------deal_testHotKey,o(╥﹏╥)o";  //sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
}
```



- **注 ：当使用了SentinelResource 注解的时候一定要配置兜底的异常方法**
- @SentinelResource 只会识别配置的异常，系统内的异常要另外添加注解

### 5 系统规则

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load 、cpu 使用率、平局RT 、入口QPS和并发线程等几个纬度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统的整体稳定性

系统保护规则是应用整体纬度的，而不是资源纬度的，并且对**入口流量生效**。入口流量指的是进入应用的流量（EntryType.IN）,比如Web 服务或Dubbo服务端接收请求，都属于入口流量



系统规则支持以下的模式

- Load自适应（仅对Linux/Unix-like 机器生效）：系统的load作为启发指标，进行自适应系统保护。当系统load超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR阶段）系统容量有系统的 maxQPS*minRT 估算得出。设定的查考值一般是 CPU cores * 2.5
- CPU usage(1.5.0+版本)：当系统CPU使用率超过阈值触发系统保护（取值范围0.0-1.0），比较灵敏
- 平局 RT ：当单台机器上的所有入口流量的平均 RT 达到阈值即触发系统保护，单位为毫秒
- 并发线程数：当单台机器上所有入口流量的并发线程达到阈值即触发系统保护
- 入口 QPS：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护



### 6 @SentinelResource

#### 6.1 客户自定义限流处理逻辑

- 自定义的异常处理类

  ```java
  public class CustomerBlockHandler {
      // 返回值跟业务类一样就行，问题 如果不一样会怎么样
      //事实证明这里的返回值必须跟业务类相同，不然会报错
      public static CommonResult handlerException(BlockException exception) {
          return new CommonResult(4444, "按客戶自定义,global handlerException----1");
      }
  }
  ```

- 业务类

  ```java
  @GetMapping("/rateLimit/customerBlockHandler")
  @SentinelResource(value = "customerBlockHandler",
          blockHandlerClass = CustomerBlockHandler.class,  // 异常处理的类
          blockHandler = "handlerException2")  //要使用异常处理类，中的方法
  public CommonResult customerBlockHandler() {
      return new CommonResult(200, "按客戶自定义", new Payment(2020L, "serial003"));
  }
  ```

- 总结

  ![](.\..\..\99-资源\自定义异常处理.bmp)

#### 6.2 更多注解属性说明

知道就行 ，但是我不知道



### 7 服务熔断

#### 7.1 Ribbon系列

- 配置类---(同其他的一样)

  ```java
  @Configuration
  public class ApplicationContextConfig {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

- 业务类

  ```java
  
  @RestController
  @Slf4j
  public class CircleBreakerController {
      public static final String SERVICE_URL = "http://nacos-payment-provider";
  
      @Resource
      private RestTemplate restTemplate;
  
      @RequestMapping("/consumer/fallback/{id}")
      @SentinelResource(value = "fallback", fallback = "handlerFallback", blockHandler = "blockHandler",
              exceptionsToIgnore = {IllegalArgumentException.class})
      public CommonResult<Payment> fallback(@PathVariable Long id) {
          CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
  
          if (id == 4) {
              throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
          } else if (result.getData() == null) {
              throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
          }
  
          return result;
      }
  
      //本例是fallback
      public CommonResult handlerFallback(@PathVariable Long id, Throwable e) {
          Payment payment = new Payment(id, "null");
          return new CommonResult<>(444, "兜底异常handlerFallback,exception内容  " 
                                    + e.getMessage(), payment);
      }
  
      //本例是blockHandler
      public CommonResult blockHandler(@PathVariable Long id, BlockException blockException) {
          Payment payment = new Payment(id, "null");
          return new CommonResult<>(445, "blockHandler-sentinel限流,无此流水: blockException  " 
                                    + blockException.getMessage(), payment);
      }
  }
  
  ```



#### 7.2 Feign系列

- yaml 文件

  ```yml
  # 激活Sentinel对Feign的支持
  feign:
    sentinel:
      enabled: true
  ```

- 主启动类

  ```java
  @EnableDiscoveryClient
  @SpringBootApplication
  @EnableFeignClients    //使用Feign要添加这个注解
  public class OrderNacosMain84 {
      public static void main(String[] args) {
          SpringApplication.run(OrderNacosMain84.class, args);
      }
  }
  ```

- Feign接口

  ```java
  @FeignClient(value = "nacos-payment-provider",     // 生产者注册进nacos中的名字
               fallback = PaymentFallbackService.class)  //实现类
  public interface PaymentService {
      @GetMapping(value = "/paymentSQL/{id}")
      public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
  }
  ```

- 实现类

  ```java
  @Component  //一定要记得将类加入到容器中
  public class PaymentFallbackService implements PaymentService {
      @Override
      public CommonResult<Payment> paymentSQL(Long id) {
          return new CommonResult<>(44444, "服务降级返回,---PaymentFallbackService", 
                                    new Payment(id, "errorSerial"));
      }
  }
  ```

- 业务类

  ```java
  @Resource
  private PaymentService paymentService;
  
  @GetMapping(value = "/consumer/paymentSQL/{id}")
  public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
      return paymentService.paymentSQL(id);
  }
  ```

  

### 8 规则持久化

#### 8.1 pom文件

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

#### 8.2 yml配置

```yaml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 121.4.184.80:2222 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719
      datasource:
        ds1:
          nacos:
            server-addr: 121.4.184.80:2222
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

#### 8.3 nacos配置

在nacos中配置一个名为 ${spring.application.name} 的配置文件

```json
[
    {
        "resource": "/rateLimit/byUrl",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

- resource：资源名称；
- limitApp：来源应用；
- grade：阈值类型，0表示线程数，1表示QPS；
- count：单机阈值；
- strategy：流控模式，0表示直接，1表示关联，2表示链路；
- controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；
- clusterMode：是否集群。



#### 8.4  可能还有其他方式的持久化



## 十一 分布式事务处理 seata

### 1 专业术语

**1+3**

- 1：全局唯一ID
- 3：三组件模型

### 2 三组件模型

#### 2.1 介绍

- **TC (Transaction Coordinator) - 事务协调者**：维护全局和分支事务的状态，驱动全局事务提交或回滚。
- **TM (Transaction Manager) - 事务管理器**：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
- **RM (Resource Manager) - 资源管理器**：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。



#### 2.2 官网流程图

![](.\..\..\99-资源\seata处理过程.bmp)

1. TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID；
2. XID 在微服务调用链路的上下文中传播；
3. RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖；
4. TM 向 TC 发起针对 XID 的全局提交或回滚决议；
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。

